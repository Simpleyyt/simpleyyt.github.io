---

layout: post
categories: java
title: 单例模式
tagline: by 炸鸡可乐
tags: 
  - 炸鸡可乐
---

作为对象的创建模式，单例模式确保其某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类。

<!--more-->

### 一、介绍
> 单例模式是一种创建型模式

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一，属于**创建型模式**的一种，它提供了一种创建对象的最佳方式。

这种模式的意义在于保证一个类仅有一个实例，并提供一个访问它的全局访问点，避免重复的创建对象，节省系统资源。
### 二、实现思路
创建一个类，**将其默认构造方法私有化**，使外界不能通过`new Object`来获取对象实例，同时提供一个对外获取对象唯一实例的方法。

例如，创建一个`SingleObject`，如下：
```
public class SingleObject {
 
   //创建 SingleObject 的一个对象
   private static SingleObject instance = new SingleObject();
 
   //让构造函数为 private，这样该类就不会被实例化
   private SingleObject(){}
 
   //获取唯一可用的对象
   public static SingleObject getInstance(){
      return instance;
   }
 
   public void showMessage(){
      System.out.println("Hello World!");
   }
}
```
从 singleton 类获取唯一的对象

```
public class SingletonPatternDemo {
   public static void main(String[] args) {
 
      //不合法的构造函数
      //编译时错误：构造函数 SingleObject() 是不可见的
      //SingleObject object = new SingleObject();
 
      //获取唯一可用的对象
      SingleObject object = SingleObject.getInstance();
 
      //显示消息
      object.showMessage();
   }
}
```
执行程序，输出结果:

```
Hello World!
```
### 三、单例模式的几种实现方式
##### 3.1、懒汉式，线程不安全(不推荐)
> 这种方式是最基本的实现方式，这种实现最大的问题就是不支持多线程。因为没有加锁 synchronized，所以严格意义上它并不算单例模式。
这种方式 lazy loading 很明显，不要求线程安全，在多线程不能正常工作。

```
public class Singleton {  

    private static Singleton instance;  
    
    private Singleton (){}  
  
    public static Singleton getInstance() {
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
    }  
}
```
##### 3.2、懒汉式，线程安全(不推荐)
> 这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。
优点：第一次调用才初始化，避免内存浪费。
缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。
getInstance() 的性能对应用程序不是很关键（该方法使用不太频繁）。

```
public class Singleton {

    private static Singleton instance;
    
    private Singleton (){}
    
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
##### 3.3、饿汉式(推荐使用)
> 这种方式比较常用，但容易产生垃圾对象。
优点：没有加锁，执行效率会提高。
缺点：类加载时就初始化，浪费内存。
它基于 classloader 机制避免了多线程的同步问题，不过，instance 在类装载时就实例化，虽然导致类装载的原因有很多种，在单例模式中大多数都是调用 getInstance 方法， 但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化 instance 显然没有达到 lazy loading 的效果。

```
public class Singleton {

    private static Singleton instance = new Singleton();
    
    private Singleton (){}
    
    public static Singleton getInstance() {
        return instance;
    }
}
```
##### 3.4、双检锁/双重校验锁(推荐使用)
> 这种方式采用双锁机制，安全且在多线程情况下能保持高性能。
getInstance() 的性能对应用程序很关键。

```
public class Singleton {  

    private volatile static Singleton singleton;  
    
    private Singleton (){}  
    
    public static Singleton getSingleton() {  
        if (singleton == null) {  
            synchronized (Singleton.class) {  
                if (singleton == null) {  
                    singleton = new Singleton();  
                }  
            }  
        }  
        return singleton;  
    }  
}
```
##### 3.5、静态内部类(推荐使用)
> 这种方式能达到双检锁方式一样的功效，对静态域使用延迟初始化，但实现更简单。这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。

```
public class Singleton {

    private static class SingletonHolder {
    
        private static final Singleton INSTANCE = new Singleton();
    }
    
    private Singleton (){}
    
    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
##### 3.6、枚举(推荐使用)
> 这种实现方式还没有被广泛采用，但这是实现单例模式的最佳方法。它更简洁，自动支持序列化机制，绝对防止多次实例化。
这种方式是 Effective Java 作者 Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还自动支持序列化机制，防止反序列化重新创建新的对象，绝对防止多次实例化。不过，由于 JDK1.5 之后才加入 enum 特性，用这种方式写不免让人感觉生疏，在实际工作中，也很少用。

```
public enum Singleton { 

    INSTANCE;  
    
    public void doMethod() {
    
    }  
}
```
### 四、应用
单例模式在Java中的应用也很多，例如`Runtime`就是一个典型的例子，源码如下：
```java
public class Runtime {
    
    private static Runtime currentRuntime = new Runtime();

    /**
     * Returns the runtime object associated with the current Java application.
     * Most of the methods of class <code>Runtime</code> are instance 
     * methods and must be invoked with respect to the current runtime object. 
     * 
     * @return  the <code>Runtime</code> object associated with the current
     *          Java application.
     */
    public static Runtime getRuntime() { 
    return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {}

    ...
}
```
很清晰的看到，使用了**饿汉式**方式创建单例对象！
### 五、总结
一般情况下，不建议使用第 1 种和第 2 种懒汉方式，建议使用第 3 种饿汉方式。只有在要明确实现 lazy loading 效果时，才会使用第 5 种登记方式。如果涉及到反序列化创建对象时，可以尝试使用第 6 种枚举方式。如果有其他特殊的需求，可以考虑使用第 4 种双检锁方式。
