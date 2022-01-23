---
layout: post
title:  线程安全之synchronized关键字
categories: 并发编程系列
tagline: by 小九
tags:
  - 小九
---

之前我讲了关于 线程基础方面的相关知识，本篇文章将会带着大家来学习下线程安全相关的知识。
<!--more-->

## 1.多线程下为什么会存在线程安全问题

线程的合理使用能够提升程序的处理性能，一是能够利用多核 `CPU` 来实现线程的并行执行，二是线程的异步化执行能够提高系统的吞吐量。

虽然线程有这些优点，但同时也带来了很多问题。比如说：

### 共享变量带来的安全性问题

先来看个图：

![](http://www.justdojava.com/assets/images/2019/java/image-xiaojiu/20191216/1.jpg)

一个变量 i ，如果线程 A 或者线程 B 单独访问并且修改变量 i 的值没有任何问题，那如果并行的修改变量 i ，那就会有安全性问题。

然后用代码来模拟一下这种场景，为了更好的看到效果，我用100个线程：

```java
public class ThreadDemo1 {

    private static int i = 0;

    public static void inc() {
        try {
            Thread.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        i++;
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> ThreadDemo1.inc()).start();
        }
        Thread.sleep(1000);
        System.out.println("运行结果" + i);
    }
}
```

输出结果：

```java
88
```

这个输出结果是不固定的，第一次可能是 88 ，第二次可能是 87 ，这个结果就和我们预期的结果不一致（预期结果是100），所以一个对象是否是线程安全的，取决于它是否会被多个线程访问，以及程序中是如何去使用这个对象的。如果 多个线程访问同一个共享对象，在不需额外的同步以及调用端代码不用做其他协调的情况下，这个共享对象的状态 依然是正确的（正确性意味着这个对象的结果与我们预期 规定的结果保持一致），那说明这个对象是线程安全的。

对于线程安全性，本质上是管理对于数据状态的访问，而且这个这个状态通常是共享的、可变的。共享：是指这个 数据变量可以被多个线程访问；可变：指这个变量的值在 它的生命周期内是可以改变的。

## 2.如何保证线程并行的数据安全性-Synchroinzed 

针对上面那种情况，我们该如何解决这种问题呢？首先想到的就是加锁，并且这种锁必须是互斥的。比如上面的图片的例子，如果线程A在修改 i 的值时，线程 B 就不能去修改 i 的值。也就是说并行去修改共享变量的值会有线程安全性问题，那么我们不让你并行，不就解决了这个问题嘛。所以java提供了 Synchroinzed 关键字。

### 2.1 Synchroinzed 的基本认识

Synchroinzed  很早就有了，只是之前是重量级锁，所以很好有人使用。在 javaSE 1.6 对Synchroinzed进行了优化引入了偏向锁和轻量级锁。所以在并发量不高的情况还是推荐使用 Synchroinzed  来加锁。为什么是并发量不高的情况推荐使用，因为并发量高的情况 Synchroinzed  会升级为重量级锁。

### 2.2 Synchroinzed 的三种加锁方式

1. 修饰实例方法，锁是当前实例对象 ，进入同步代码前要获得当前实例的锁
2. 修饰静态方法，锁是当前类的class对象 ，进入同步代码前要获得当前类对象的锁
3. 修饰代码块，锁是括号里面的对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

看下简单的代码

```java
public class SynchroinzedDemo {

    /**
     * 对静态方法加锁
     */
    public static synchronized void test(){}
    /**
     * 对实例方法加锁
     */
    public synchronized void test1(){}
    /**
     * 对代码块加锁
     */
    public void test2(){
        synchronized(this){}
    }
}
```

然后我们将上面的例子实现 synchronized 加锁：

```java
public class ThreadDemo1 {

    private static int i = 0;

    public static void inc() {
        synchronized (ThreadDemo1.class){
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            i++;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> ThreadDemo1.inc()).start();
        }
        Thread.sleep(1000);
        System.out.println("运行结果" + i);
    }
}
```

运行结果：

```java
运行结果100
```

完美的解决共享变量并行修改带来的线程安全问题。

# 总结

本文带着大家了解了一下线程的安全性问题和解决线程安全性问题的 synchronized 关键字的用法。后面的并发编程系列会讲解更多的解决线程安全性的方法。敬请期待！

