---
layout: post
category: java线程
title: 面试官:都说阻塞 I/O 模型将会使线程休眠，为什么 Java 线程状态却是 RUNNABLE？
tagline: by 小黑
tags: 
  - 小黑
---

使用 Java 阻塞 I/O 模型读取数据，将会导致线程阻塞，线程将会进入休眠，从而让出 CPU 的执行权，直到数据读取完成。这个期间如果使用 jstack 查看线程状态，却可以发现Java 线程状态是处于 RUNNABLE，这就和上面说的存在矛盾，为什么会这样？

<!--more-->

上面的矛盾其实是混淆了操作系统线程状态与 Java 线程状态。这里说的线程阻塞进入休眠状态，其实是操作系统层面线程实际状态。而我们使用 jstack 查看的线程状态却是 JVM 中的线程状态。

线程是操作系统中一种概念，Java 对其进行了封装，Java 线程本质上就是操作系统的中线程，其状态与操作系统的状态大致相同，但还是存在一些区别。

下面首先来看我们熟悉的 Java 线程状态。

## Java 线程状态

Java 线程状态定义在 `Thread.State` 枚举中，使用  `thread#getState` 方法可以获取当前线程的状态。

*`Thread.State` 状态如下图*：

![State.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190901/State-bdd4cf03.png)

可以看到 Java 线程总共存在 6 中状态，分别为：

- NEW（初始状态）
- RUNNABLE（运行状态）
- BLOCKED（阻塞状态）
- WATTING（等待状态）
- TIMED_WAITING（限时等待状态）
- TERMINATED（终止状态）

**NEW（初始状态）与 RUNNABLE（运行状态）**

每个使用  `new Thread()` 刚创建出线程实例状态处于 **NEW** 状态，一旦调用 `thread.start()`，线程状态将会变成 **RUNNABLE**。

**RUNNABLE（运行状态） 与 BLOCKED（阻塞状态）**

**RUNNABLE** 状态的线程在进入由 `synchronized`修饰的方法或代码块前将会尝试获取一把隐式的排他锁，一旦获取不到，线程状态将会变成  **BLOCKED**，等待获取锁。一旦有其他线程释放这把锁，线程成功抢到该锁，线程状态就将会从 **BLOCKED** 转变为 **RUNNABLE** 状态。

**RUNNABLE（运行状态） 与 WATTING（等待状态）**

处于 **WATTING** 状态的线程将会一直处于无限期的等待状态，需要等待其他线程唤醒。总共存在三种方法将会使线程从 **RUNNABLE** 变成 **WATTING**。

1. `Object#wait`

线程在获取到 `synchronized` 隐式锁后，显示的调用 `Object#wait()`方法。这种情况下该线程将会让出隐式锁，一旦其他线程获取到该锁，且调用了  `Object.notify()` 或`object.notifyAll()`，线程将会唤醒，然后变成 **RUNNABLE**。

2. `Thread#join`

`join `方法是一种线程同步方法。假设我们在 main 方法中执行 Thread A.join() 方法，main 线程状态就会变成 **WATTING**。直到 A 线程执行完毕，main 线程才会再变成 **RUNNABLE**。

3. `LockSupport#park()`

LockSupport 是 JDK 并发包里重要对象，很多锁的实现都依靠该对象。一旦调用 `LockSupport#park()`，线程就将会变为 **WATTING** 状态。如果需要唤醒线程就需要调用 LockSupport#unpark,然后线程状态重新变为 **RUNNABLE**。


**RUNNABLE（运行状态） 与 TIMED_WAITING（限时等待状态）**

**TIMED_WAITING** 与 **WATTING** 功能一样，只不过前者增加限时等待的功能，一旦等待时间超时，线程状态自动变为 **RUNNABLE**。以下几种情况将会触发这种状态：

1. `Thread#sleep(long millis)`
2. 占有 synchronized 隐式锁的线程调用 `Object.wait (long timeout)` 方法
3. `Thread#join (long millis)`
4. `LockSupport#parkNanos (Object blocker, long deadline) `
5. `LockSupport#parkUntil (long deadline)`

**RUNNABLE（运行状态）与 TERMINATED（终止状态）**

线程一旦执行结束或者线程执行过程发生异常且未正常捕获处理，状态都将会自动变成 **TERMINATED**。

Java 线程 6 种状态看起来挺复杂的，但其实上面 **BLOCKED**，**WATTING**，**TIMED_WAITING**，都会使线程处于休眠状态，所以我们将这三类都归类为休眠状态。这么分类的话，Java 线程生命周期就可以简化为下图：

![java线程状态2.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190901/java线程状态2-06769441.png)

## 通用操作系统线程状态

上面讲完 Java 系统的线程状态，我们来看下通用操作系统的线程状态。操作系统线程状态可以分为初始状态，可运行状态，运行状态，休眠状态以及终止状态，如下图：

![操作系统线程状态1.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190901/操作系统线程状态1-61c4dbe1.png)

这 5 中状态详细情况如下：

1. 初始状态，这时候线程刚被创建，还不能分配 CPU 。
2. 可运行状态，线程等待系统分配 CPU ，从而执行任务。
3. 运行状态，操作系统将 CPU 分配给线程，线程执行任务。
4. 休眠状态，运行状态下的线程如果调用阻塞 API，如阻塞方式读取文件， 线程状态就将变成休眠状态。这种情况下，线程将会让出 CPU 使用权。休眠结束，线程状态将会先变成可运行状态。
5. 线程执行结束或者执行过程发生异常将会使线程进入终止状态，这个状态下线程使命已经结束。

## 对比两者线程状态

比较 Java 线程与操作系统线程，可以发现 Java 线程状态没有**可运行状态**。也就是说 Java 线程 **RUNNABLE** 状态包括了操作系统的可运行状态与运行状态。一个处于  **RUNNABLE** 状态 Java 线程，在操作系统层面状态可能为可运行状态，正在等待系统分配 CPU 使用权。

另外 Java 线程细分了操作系统休眠状态，分成了 **BLOCKED**，**WATTING**，**TIMED_WAITING** 三种。

当线程调用阻塞式 API，线程进入休眠状态，这里指的是操作系统层面的。从 JVM 层面，Java 线程状态依然处于 RUNNABLE 状态。JVM 并不关心操作系统线程实际状态。从 JVM 看来等待 CPU 使用权（操作系统线程状态为可运行状态）与等待 I/O （操作系统线程状态处于休眠状态）没有区别，都是在等待某种资源，所以都归入 RUNNABLE 状态。

其他 Java 线程状态与操作线程状态类似。



