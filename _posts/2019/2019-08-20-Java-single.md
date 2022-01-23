---
layout: post  
title: java设计模式之单例模式
tagline: by 小九
categories: 设计模式
tags: 
    - 小九

---

java设计模式之单例模式
<!--more-->

### 单例模式的特点

1. 单例类只能有一个构造函数,并且是私有的
2. 单例类必须自己创建自己的唯一实例
3. 单例类必须给所有其他对象提供这一实例。

### 单例模式的优缺点

1. 优点:在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例
2. 缺点:没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。

### 饿汉式单列

1. 优点：没有加任何的锁、执行效率比较高，在用户体验上来说，比懒汉式更好。
2. 缺点：类加载的时候就初始化，不管用与不用都占着空间，浪费了内存，有可能占着茅

下面我们来看下饿汉式单列的代码

```java   
public class HungrySingleton {
	//私有的构造函数
    private HungrySingleton() { }
    //自己创建自己的实例
    private static final HungrySingleton hungrySingleton = new HungrySingleton();
    //向其他对象提供这一实例
    public static HungrySingleton getInstance() {
        return hungrySingleton;
    }
}
```

### 懒汉式单列 

   懒汉式单列是线程不安全的

下面我们来看下懒汉式单列的代码

```java
public class LazySimpleSingleton {
    //私有的构造函数
    private LazySimpleSingleton() {}

    private static LazySimpleSingleton lazy = null;
    //实例化对象并提供这一实例
    public static LazySimpleSingleton getInstance() {
        if (lazy == null) {
            lazy = new LazySimpleSingleton();
        }
        return lazy;
    }
}
```

### 双重检查锁单例

​	这种单例模式主要解决懒汉式单列的线程不安全的问题,用了synchronized 加锁,所以性能会有所消耗

下面我们来看下双重检查锁单例的代码

```java
public class LazyDoubleCheckSingleton {

    private volatile static LazyDoubleCheckSingleton lazy = null;

    private LazyDoubleCheckSingleton() { }

    public static LazyDoubleCheckSingleton getInstance() {
        if (lazy == null) {
            synchronized (LazyDoubleCheckSingleton.class) {
                if (lazy == null) {
                    lazy = new LazyDoubleCheckSingleton();
                }
            }
        }
        return lazy;
    }
}
```

### 静态内部类单例

​	这种单例模式可以说是完美的单例模式了

1. 延迟加载(使用的时候才会实例化),避免项目启动内存的消耗
2. 内部类一定是在方法调用之前初始化，巧妙地避免了线程安全问题

下面我们来看下双重检查锁单例的代码

```java
public class LazyInnerClassSingleton implements Serializable {
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

### spring里面单例模式的应用场景

spring里面bean的作用域默认是单例的,比如最常见的controller,service,dao这些.但是spring里面的单例bean对象是线程不安全的,因为如果这些对象里面有成员变量的话,在并发访问的时候这些成员变量将会是并发线程中的共享对象，也是影响线程安全的重要因素

这里给出几个方案

1. 将bean的作用域改为prototype
2. 自己保证线程安全
3. 不要使用成员变量,尽量使用纯方法的类,如:controller,service,dao

### 总结

本文主要讲了单例模式的特点,优缺点和4种单例模式的创建,但是这4中单例模式真的是完美的单例模式吗?有没有什么方式可以破坏单例?有没有什么方式可以防止单例被破坏?敬请期待单例模式下一期