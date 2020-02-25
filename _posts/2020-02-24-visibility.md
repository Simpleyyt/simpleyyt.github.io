---
layout: post
categories: java并发
title: 诡异的并发之可见性
tagline: by 淼淼之森
tags: 
    - 淼淼之森
---

我们都知道，随着祖国越来越繁荣昌盛，随着科技的进步，设备的更新换代，计算机体系结构、操作系统、编译程序都在不断地改革创新，但始终有一点是不变的（我对鸭血粉丝的热爱忠贞不渝）：那就是下面三者的性能耗时：CPU < 内存 < I/O
<!--more-->

但也正因为这些改变，也就在并发程序中出现了一些诡异的问题，而其中最昭著的三大问题就是：可见性、有序性、原子性。

而今天阿粉我就为大家介绍其中的恶霸之一可见性。

## 零、可见性的阐述

**可见性** 的定义是：一个线程对共享变量的修改，另外一个线程能够立刻看到。

在单核时代，所有线程都在一个CPU上执行，所以一个线程的写，一定是对其它线程可见的。就好比，一个总经理下面就一个项目负责人。

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/1.png)


此时，项目经理查看到任务G后，分配给员工A和员工B，那么这个任务的进度就能随时掌握在项目经理手中了；每个员工都能从项目经理处得知最新的项目进度。

而在多核时代后，每个CPU都有自己的缓存，这就出现了可见性问题。

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/2.png)

此时，两个项目经理同时查看到任务G后，各自分配给自己下属员工，那么这个任务的进度就只能掌握在各自项目经理手中了，因为所有员工的工作进度并不是汇报给同一个项目经理；那么，每个员工只能得知自己项目组员工的工作进度，并不能得知其他项目组的工作进度。所以，当多个项目经理在做同一个任务时，就可能出现任务配比不均、任务进度拖延、任务重复进行等多种问题。

下文我们的阐述，若无特殊说明，都是基于多核的。

## 一、导致共享变量在线程之间不可见的原因：
### 1.1、线程交叉执行

线程交叉执行多数情况是由于线程切换导致的，例如下图中的线程A在执行过程中切换到线程B执行完成后，再切换回线程A执行剩下的操作；此时线程B对变量的修改不能对线程A立即可见，这就导致了计算结果和理想结果不一致的情况。

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/4.png)

### 1.2、重排序结合线程交叉执行

例如下面这段代码
```
    int a = 0;    //行1
    int b = 0;    //行2
    a = b + 10;   //行3
    b = a + 9;    //行4
```
如果行1和行2在编译的时候改变顺序，执行结果不会受到影响；

如果将行3和行4在变异的时候交换顺序，执行结果就会受到影响，因为b的值得不到预期的19；

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/5.png)

由图知：由于编译时改变了执行顺序，导致结果不一致；而两个线程的交叉执行又导致线程改变后的结果也不是预期值，简直雪上加霜!

### 1.3、共享变量更新后的值没有在工作内存及主存间及时更新

因为主线程对共享变量的修改没有及时更新，子线程中不能立即得到最新值，导致程序不能按照预期结果执行。

例如下面这段代码：
```
package com.itquan.service.share.resources.controller;

import java.time.LocalDateTime;

/**
 * @author ：mmzsblog
 * @description：共享变量在线程间的可见性测试
 */
public class VisibilityDemo {

    // 状态标识flag
    private static boolean flag = true;

    public static void main(String[] args) throws InterruptedException {
        System.out.println(LocalDateTime.now() + "主线程启动计数子线程");
        new CountThread().start();

        Thread.sleep(1000);
        // 设置flag为false，使上面启动的子线程跳出while循环，结束运行
        VisibilityDemo.flag = false;
        System.out.println(LocalDateTime.now() + "主线程将状态标识flag被置为false了");
    }

    static class CountThread extends Thread {
        @Override
        public void run() {
            System.out.println(LocalDateTime.now() + "计数子线程start计数");
            int i = 0;
            while (VisibilityDemo.flag) {
                i++;
            }
            System.out.println(LocalDateTime.now() + "计数子线程end计数，运行结束：i的值是" + i);
        }
    }

}
```
运行结果是：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/3.png)

从控制台的打印结果可以看出，因为主线程对flag的修改，对计数子线程没有立即可见，所以导致了计数子线程久久不能跳出while循环，结束子线程。

对于这种情况，作为有强迫症的阿粉我当然不能忍，所以就引出了下一个问题：如何解决线程间不可见性

## 二、如何解决线程间不可见性
为了保证线程间可见性我们一般有3种选择:

### 2.1、volatile:只保证可见性

`volatile`关键字能保证可见性，但也只能保证可见性，在此处就能保证flag的修改能立即被计数子线程获取到。

此时纠正上面例子出现的问题，只需在定义全局变量的时候加上`volatile`关键字
```
    // 状态标识flag
    private static volatile boolean flag = true;
```
### 2.2、Atomic相关类:保证可见性和原子性

将标识状态flag在定义的时候使用Atomic相关类来进行定义的话，就能很好的保证flag属性的可见性以及原子性。

此时纠正上面例子出现的问题，只需在定义全局变量的时候将变量定义成Atomic相关类
```
    // 状态标识flag
    private static AtomicBoolean flag = new AtomicBoolean(true);
```
不过值得注意的一点是，此时原子类相关的方法设置新值和得到值的放的是有点变化，如下：
```
    // 设置flag的值
    VisibilityDemo.flag.set(false);
    
    // 获取flag的值
    VisibilityDemo.flag.get()
```

### 2.3、Lock: 保证可见性和原子性

此处我们使用的是Java常见的synchronized关键字。

此时纠正上面例子出现的问题，只需在为计数操作`i++`添加`synchronized`关键字修饰
```
    synchronized (this) {
        i++;
    }
```

通过上面三种方式，阿粉我都得到类似如下的期望结果：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/6.png)

然而，接下来阿粉我要对其中的`volatile`和`synchronized`关键字做一番较为详细的解释。

## 三、可见性-volatile

Java内存模型对volatile关键字定义了一些特殊的访问规则，当一个变量被volatile修饰后，它将具备两种特性，或者说volatile具有下列两层语义：
- 第一、保证了不同线程对这个变量进行读取时的可见性。即一个线程修改了某个变量的值， 这个新值对其他线程来说是立即可见的。 (volatile解决了线程间共享变量的可见性问题)。
- 第二、禁止进行指令重排序， 阻止编译器对代码的优化。

针对第一点，volatile保证了不同线程对这个变量进行读取时的可见性，具体表现为：
- 1： 使用 volatile 关键字会强制将在某个线程中修改的共享变量的值立即写入主内存。
- 2： 使用 volatile 关键字的话， 当线程 2 进行修改时， 会导致线程 1 的工作内存中变量的缓存行无效（反映到硬件层的话， 就是 CPU 的 L1或者 L2 缓存中对应的缓存行无效);

附一张CPU缓存模型图：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/7.png)

- 3： 由于线程 1 的工作内存中变量的缓存行无效，所以线程1再次读取变量的值时会去主存读取。基于这一点，所以我们经常会看到文章中或者书本中会说volatile 能够保证可见性。

**综上所述**：就是用volatile修饰的变量，对这个变量的读写，不能使用 CPU 缓存，必须从内存中读取或者写入。

使用volatile无法保障线程安全，那么volatile的作用是什么呢？

其中之一：（对状态量进行标记，保证其它线程看到的状态量是最新值）

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/8.png)

volatile关键字是Java虚拟机提供的最轻量级的同步机制，很多人由于对它理解不够（其实这里你想理解透的话可以看看happens-before原则），而往往更愿意使用synchronized来做同步。所以接下来阿粉我再说说`synchronized`关键字。

## 四、可见性synchronized

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/02-01/9.png)

### 4.1、作用域
synchronized关键字的作用域有二种：

- 1）是某个对象实例内，`synchronized aMethod(){}`可以防止多个线程同时访问这个对象的synchronized方法。<br>

    如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任何一个synchronized方法。<br>

    这时，不同的对象实例的synchronized方法是不相干扰的。也就是说，其它线程照样可以同时访问相同类的另一个对象实例中的synchronized方法。<br>

    因为当修饰非静态方法的时候，锁定的是当前实例对象。

- 2）是某个类的范围，`synchronized static aStaticMethod{}`防止多个线程同时访问这个类中的synchronized static 方法。它可以对类的所有对象实例起作用。 <br>
 
    因为当修饰静态方法的时候，锁定的是当前类的 Class 对象。

### 4.2、可用于方法中的某个区块中
除了方法前用synchronized关键字，synchronized关键字还可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。

用法是: 
```
synchronized(this){
    /*区块*/
}
```
它的作用域是当前对象； 

### 4.3、不能继承
synchronized关键字是不能继承的，也就是说，基类的方法
```
synchronized f(){
    // 具体操作
} 
```
在继承类中并不自动是
```
synchronized f(){
    // 具体操作  
}
```
而是变成了
```
f(){
    // 具体操作
}
```
继承类需要你显式的指定它的某个方法为synchronized方法； 

综上3点所述：synchronized关键字主要有以下这3种用法：
- **修饰实例方法**：作用于当前实例加锁，进入同步代码前要获得当前实例的锁
- **修饰静态方法**：作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁
- **修饰代码块**：指定加锁对象，对给定对象加锁，进入同步代码块前要获得给定对象的锁

这三种用法就基本保证了共享变量在读取的时候，读取到的是最新的值。

### 4.4、JVM关于synchronized的两条规定：
- 线程解锁前，必须把共享变量的最新值刷新到主内存

- 线程加锁时，将清空工作内存中共享变量的值，从而是使用共享变量时，需要从主内存中重新读取最新的值（注意：加锁与解锁是同一把锁）

从上面的这两条规则也可以看出，这种方式保证了内存中的共享变量一定是最新值。

但我们在使用synchronized保证可见性的时候也要注意以下几点：
- A．无论synchronized关键字加在方法上还是对象上，它取得的锁都是对象；而不是把一段代码或函数当作锁――而且同步方法很可能还会被其他线程的对象访问。
- B．每个对象只有一个锁（lock）与之相关联。Java 编译器会在 synchronized 修饰的方法或代码块前后自动加上加锁 lock() 和解锁 unlock()，这样做的好处就是加锁 lock() 和解锁 unlock() 一定是成对出现的，毕竟忘记解锁 unlock() 可是个致命的 Bug（意味着其他线程只能死等下去了）。
- C．实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

以上内容就是我对并法中的可见性的一点理解与总结了，下期我们接着叙述并发中的有序性。








参考文章：
- 1、极客时间的Java并发编程实战
- 2、https://www.jianshu.com/p/89a8fa8ffe39
- 3、https://www.cnblogs.com/xiaonantianmen/p/9970368.html
- 4、https://www.lagou.com/lgeduarticle/78722.html
- 5、https://blog.csdn.net/evankaka/article/details/44153709