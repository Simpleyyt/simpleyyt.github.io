---
layout: post  
title: 怎么破坏单例模式和怎么防止单例模式被破坏
tagline: by xiaojiu
categories: Java  
tag: 
    - Java,single

---

怎么破坏单例模式和怎么防止单例模式被破坏
<!--more-->

### 01、简介

单例模式（Singleton Pattern）是指确保一个类在任何情况下都绝对只有一个实例，并提供一个全局访问点，破坏单例模式的话,就是说可以弄出两个或两个以上的实例嘛。既然可以破坏，那么我们怎么防止单例模式被破坏呢？这篇文章我会带着大家来了解一下

### 02、破坏单列模式的方式

1. 反射
2. 序列和反序列化

这里我以静态内部类单例来举例，先看下静态内部类单例的代码

```java   
public class LazyInnerClassSingleton {
    // 私有的构造方法
    private LazyInnerClassSingleton(){}
    // 公有的获取实例方法
    public static final LazyInnerClassSingleton getInstance(){
        return LazyHolder.LAZY;
    }
    // 静态内部类
    private static class LazyHolder{
        private static final LazyInnerClassSingleton LAZY = new LazyInnerClassSingleton();
    }
}
```

### 03、反射破坏单例模式

我们来看代码

```java
public static void main(String[] args) {
        try {
            //很无聊的情况下，进行破坏
            Class<?> clazz = LazyInnerClassSingleton.class;
            //通过反射拿到私有的构造方法
            Constructor c = clazz.getDeclaredConstructor(null);
            //因为要访问私有的构造方法，这里要设为true，相当于让你有权限去操作
            c.setAccessible(true);
            //暴力初始化
            Object o1 = c.newInstance();
            //调用了两次构造方法，相当于 new 了两次
            Object o2 = c.newInstance();
            //这里输出结果为false
            System.out.println(o1 == o2);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

输出为false，说明内存地址不同，就是实例化了多次，破坏了单例模式的特性。

### 04、防止反射破坏单例模式

通过上面反射破坏单例模式的代码，我们可以知道，反射也是通过调用构造方法来实例化对象，那么我们可以在构造函数里面做点事情来防止反射，我们把静态内部类单例的代码改造一下，看代码

```java
public class LazyInnerClassSingleton {
    // 私有的构造方法
    private LazyInnerClassSingleton(){
        // 防止反射创建多个对象
        if(LazyHolder.LAZY != null){
			throw new RuntimeException("不允许创建多个实例");
		}
    }
    // 公有的获取实例方法
    public static final LazyInnerClassSingleton getInstance(){
        return LazyHolder.LAZY;
    }
    // 静态内部类
    private static class LazyHolder{
        private static final LazyInnerClassSingleton LAZY = new LazyInnerClassSingleton();
    }
}
```

这样我们在通过反射创建单例对象的时候，多次创建就会抛出异常

```java
java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.cl.singleton.LazySingletonTest.main(LazySingletonTest.java:68)
Caused by: java.lang.RuntimeException: 只能实例化1个对象
	at com.cl.singleton.LazyInnerClassSingleton.<init>(LazyInnerClassSingleton.java:18)
	... 5 more
```

### 05、序列化破坏单例模式

用序列化的方式,需要在静态内部类(LazyInnerClassSingleton) 实现 Serializable 接口，代码在下面的防止序列化破坏单例模式里面

这里我们先来看下序列和反序列的代码

```java
	//序列化创建单例类
    public static void main(String[] args) {
        LazyInnerClassSingleton s1 = null;
        //通过类本身获得实例对象
        LazyInnerClassSingleton s2 = LazyInnerClassSingleton.getInstance();
        FileOutputStream fos = null;
        try {
            //序列化到文件中
            fos = new FileOutputStream("SeriableSingleton.obj");
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(s2);
            oos.flush();
            oos.close();
			//从文件中反序列化为对象
            FileInputStream fis = new FileInputStream("SeriableSingleton.obj");
            ObjectInputStream ois = new ObjectInputStream(fis);
            s1 = (LazyInnerClassSingleton) ois.readObject();
            ois.close();
            //对比结果,这里输出的结果为false
            System.out.println(s1 == s2);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

结果为false，说明也破坏了单例模式

### 06、防止序列化破坏单例模式

这里我们先来看下改造后的代码，然后分析原理

```java
public class LazyInnerClassSingleton implements Serializable {

    private static final long serialVersionUID = -4264591697494981165L;

    // 私有的构造方法
    private LazyInnerClassSingleton(){
        // 防止反射创建多个对象
        if(LazyHolder.LAZY != null){
            throw new RuntimeException("只能实例化1个对象");
        }
    }
    // 公有的获取实例方法
    public static final LazyInnerClassSingleton getInstance(){
        return LazyHolder.LAZY;
    }
    // 静态内部类
    private static class LazyHolder{
        private static final LazyInnerClassSingleton LAZY = new LazyInnerClassSingleton();
    }
    // 防止序列化创建多个对象,这个方法是关键
    private  Object readResolve(){
        return  LazyHolder.LAZY;
    }

}
```

在执行上面序列和反序列化代码，输出true，是不是一脸懵逼，为什么加了一个readResolve方法，就能防止序列化破坏单例模式，下面就带着大家来看下序列化的源码：

```java
public final Object readObject()throws IOException, ClassNotFoundException{
	if (enableOverride) {
		return readObjectOverride();
	}
	// if nested read, passHandle contains handle of enclosing object
	int outerHandle = passHandle;
	try {
        //看这里,看这里,就是我readObject0
		Object obj = readObject0(false);
		handles.markDependency(outerHandle, passHandle);
		ClassNotFoundException ex = handles.lookupException(passHandle);
		if (ex != null) {
			throw ex;
		}
		if (depth == 0) {
			vlist.doCallbacks();
		}
		return obj;
	} finally {
		passHandle = outerHandle;
		if (closed && depth == 0) {
			clear();
		}
	}
}
```

然后我们看下 `readObject0` 这个方法

```java
private Object readObject0(boolean unshared) throws IOException {
	...
    //主要是这个判断
	case TC_OBJECT:
    	//然后进入readOrdinaryObject这个方法
		return checkResolve(readOrdinaryObject(unshared));
	...
}
```

然后我们看下`readOrdinaryObject` 这个方法

```java
	private Object readOrdinaryObject(boolean unshared)throws IOException{
        ...
        Object obj;
        try {
            //这里判断是否有无参的构造函数,有的话就调用newInstance()实例化对象
            obj = desc.isInstantiable() ? desc.newInstance() : null; 
		...
        if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            Object rep = desc.invokeReadResolve(obj);
          ...
    }
```

这里的关键是`desc.hasReadResolveMethod()` ，这段代码的意思是查看你的单例类里面有没有`readResolve`方法，有的话就利用反射的方式执行这个方法，具体是`desc.invokeReadResolve(obj)`这段代码，返回单例对象。这里其实是实例化了两次，只不过新创建的对象没有被返回而已。如果创建对象的动作发生频率增大，就意味着内存分配开销也就随之增大，这也算是一个缺点吧。

完整的流程就是这样，是不是很神奇。

### 07、总结

破坏和防止单例模式被破坏的知识点 get 到了吗，大家平时多看些源码，能了解很多有趣的东西。