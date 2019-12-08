---
layout: post
title: 设计模式之代理模式
tagline: by 小九
categories: 设计模式
tags: 
  - 小九
---

看了这篇文章，你会对静态代理模式，`JDK` 动态代理模式和 `CGLIB` 动态代理模式有个很清晰的认识

<!--more-->

### 01、简介

1. 什么是代理模式

   代理模式也称为委托模式，属于结构型模式之一。在某些情况下，一个对象不适合或者不能直接引用另一个对象，而代理对象可以在客户端和目标对象之间起到中介的作用，比如我们生活中的邮局，快递公司，婚介所等等。

2. 代理模式分类

    代理模式分为静态代理模式和动态代理模式。

   `静态代理`是由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行之前，代理类.class文件就已经被创建了

   `动态代理`是在程序运行时通过java反射机制动态创建的。

3. 代理模式的目的

   代理模式主要有两个目的：一保护目标对象，二增强目标对象。

### 02、静态代理模式

静态代理模式的话我模拟一个古代结婚的场景。场景是这样的：在古代，某家的公子看上了别家的姑娘，一般都是家里的大人去姑娘的家里提亲，双方父母同意了，然后就拜堂成婚，后面要宴请亲朋好友。这里这个公子只需要拜堂成婚就行了，至于提亲和宴请亲友都是父母操办的。我们用代码来模拟一下这个场景。

首先我们来建个 `Person` 接口：

```java
public interface Person {
    /**
     * 人有很对行为,这里我们用到的是结婚
     */
    void marry();
}
```

然后这家公子要成亲，我们建个 `Son` 类实现 `Person` 接口：

```java
public class Son implements Person {
    @Override
    public void marry() {
        System.out.println("我终于结婚了");
    }
}
```

父亲帮儿子提亲,建个 `Father` 类:

```java
public class Father {
    private Son son;
    public Father(Son son){
        this.son = son;
    }
    public void marry(){
        System.out.println("父亲上门提亲");
        this.son.marry();
        System.out.println("父亲宴请亲友");
    }
}
```

最后是测试代码：

```java
public class Test {
    public static void main(String[] args) {
        Father father = new Father(new Son());
        father.marry();
    }
}
```

输出：

```java
父亲上门提亲
我终于结婚了
父亲宴请亲友
```

代码写完了，大家有没有发现静态代理模式的一个缺点。那就是单一，一个类只能代理一个目标对象。比如上面的场景，父亲只能为自己的儿子提亲，不能为别人家的孩子提亲。

下面我们来看看动态代理是怎么解决这个问题的。

### 03、动态代理模式

动态代理模式分为 `JDK` 动态代理和 `cglib` 动态代理两种。这里先用 `JDK` 动态代理的方式来模拟一个通过婚介所找朋友的场景。

先将 `Person` 接口改动下：

```java
public interface Person {
    /**
     * 找朋友
     */
    void findFriend();
}
```

然后是婚介所 `JDKMatrimonialAgency` 类：

```java
public class JDKMatrimonialAgency implements InvocationHandler {
    //被代理的对象，把引用给保存下来
    private Object target;
    public Object getInstance(Object target) throws Exception{
        this.target = target;
        Class<?> clazz = target.getClass();
        return Proxy.newProxyInstance(clazz.getClassLoader(),clazz.getInterfaces(),this);
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object obj = method.invoke(this.target,args);
        after();
        return obj;
    }
    private void before(){
        System.out.println("这里是婚介所,请提供你的需求");
    }
    private void after(){
        System.out.println("已经找到合适的,尽快安排你相亲");
    }
}
```

`JDK` 动态代理主要是实现 `InvocationHandler` 接口，并实现 `invoke` 方法

然后创建 `Customer` 类：

```java
public class Customer implements Person {
    @Override
    public void findFriend() {
        System.out.println("我要找一个胸大,腿长又好看的美女");
    }
}
```

最后测试类：

```java
public class Test {
    public static void main(String[] args) {
        try {
            Person obj = (Person)new JDKMatrimonialAgency().getInstance(new Customer());
            obj.findFriend();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

看下结果：

```java
这里是婚介所,请提供你的需求
我要找一个胸大,腿长又好看的美女
已经找到合适的,尽快安排你相亲
```

然后我们用 `CGLIB` 来实现，如果不是spring（spring已经集成了 `CGLIB` ）环境需要先引入 `jar` 包：

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

然后加一个 `CglibMatrimonialAgency` 类：

```java
public class CglibMatrimonialAgency implements MethodInterceptor {

    public Object getInstance(Class<?> clazz) throws Exception{
        Enhancer enhancer = new Enhancer();
        //要把哪个设置为即将生成的新类的父类
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return enhancer.create();
    }
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy)
        throws Throwable {
        //业务的增强
        before();
        Object obj = methodProxy.invokeSuper(o,objects);
        after();
        return obj;
    }
    private void before(){
        System.out.println("这里是婚介所,请提供你的需求");
    }
    private void after(){
        System.out.println("已经找到合适的,尽快安排你相亲");
    }
}
```

`CGLIB` 主要是实现 `MethodInterceptor` 并实现 `intercept` 方法。

看下结果：

```java
这里是婚介所,请提供你的需求
我要找一个胸大,腿长又好看的美女
已经找到合适的,尽快安排你相亲
```

### 04、JDK和CGLIB动态代理对比

1. `JDK` 动态代理是实现了被代理对象的接口，`CGLib` 是继承了被代理对象。
2. `JDK` 和 `CGLib` 都是在运行期生成字节码，`JDK` 是直接写 `Class` 字节码，`CGLib` 使用 `ASM`框架写 `Class` 字节码，`Cglib` 代理实现更复杂，生成代理类比 `JDK` 效率低。
3. `JDK` 调用代理方法，是通过反射机制调用，`CGLib` 是通过 `FastClass` 机制直接调用方法，
   `CGLib` 执行效率更高。

### 05、代理模式的优缺点

优点:

1. 降低耦合度,扩展性好
2. 代理对象将代理对象和目标对象分离,起到保护目标对象的作用
3. 可以对目标对象的功能增强

缺点:

1. 增加类的数量
2. 因为会调用增强方法,所以会造成处理速度慢
3. 增加了系统的复杂度（这是好的架构都会有的缺点，比如spring）



