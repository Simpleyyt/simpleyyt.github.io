---
layout: post
categories: java并发
title: 面试官没想到，一个 Java 线程生命周期，我可以扯半小时
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
面试官：你不是精通 Java 并发吗？从基础的 Java 线程生命周期开始讲讲吧。

好的，面试官。吧啦啦啦...
<!--more-->

如果要说 Java 线程的生命周期的话，那我觉得就要先说说操作系统的线程生命周期

因为 JVM 是跑在操作系统上面的嘛，所以是绕不过去的，而且可以说， Java 语言中的线程本质上就是操作系统的线程

聪明的你肯定也发现了，不管是操作系统，还是 Java 或者 C# 都有线程的概念。在它们之间，关于线程的生命周期这一部分，肯定是有相同之处的，否则的话，操作系统自己一套生命周期流程， Java 又有自己的一套， C# 又有自己的一套，而且相互之间还要能够互相配合，这种成本想想就大的不行对吧

所以咱们就来看看，通用的线程生命周期都有啥

先直接上张图（阿粉这次的图，可还行？）：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/07/01-system-thread-state.jpg)

可以看到，主要有 new ， ready ， running ， waiting ， terminated 5 种状态

其中：

- new 只是说，这个线程被创建了，但是还不允许分配 CPU 执行。因为这个状态只是说明你在编程语言层面被创建了，操作系统层面还没有被创建，肯定就谈不上分配 CPU 执行了

- ready 这个状态是说，在操作系统层面已经成功创建了，所以接下来就是等待分配 CPU 执行了。还记得那句经典的嘛？ ready ？ go ！

- running 的状态，相信你就知道了，我都已经 ready 了，此时如果再给我分配一下 CPU 我是不是就可以 go 了？那不就是 running 状态了嘛

- waiting 状态，就是线程在 running 状态的时候，突然发现，哎，我需要进行一下 I/O 操作，或者需要等待某个事件发生（比如说需要某个条件变量），这个时候是不是就不能再继续 happy 的 running 了。那咋办？ waiting 一下呗

	- 那你都 waiting 了，占用的 CPU 资源是不是应该释放掉？所以说， waiting 状态的线程是永远没有机会获得 CPU 使用权的
	
	- 你是不是一听「永远没有机会」这几个字就给吓坏了，我该不会永远没有机会执行了吧。放心吧，你不是在 waiting 嘛，等你 wait 的事件发生了，就可以继续到 running 状态
	
- 当整个线程执行完毕，或者出现异常的时候，就进入了 terminated 状态，也就是线程的使命就完成啦，处于 terminated 状态的线程不会再切换到其他状态了

通用的线程生命周期以及它们之间是如何切换的，到这里，应该就比较清楚了

接下来咱们看看 Java 线程的生命周期，在这个基础上是怎么做的优化，有什么区别

# Java 线程的生命周期

咱们先来瞅瞅源码定义的状态（为了突出重点，我把注释都去掉了）：

```
public enum State {
	NEW,
	RUNNABLE,
	BLOCKED,
	WAITING,
	TIMED_WAITING,
	TERMINATED;
}
```

能够清楚的看到，在源码中定义了 6 种线程状态，刚才的通用状态有几种来着？ 5 种对吧，现在是 6 种。

这 6 种是干啥的？刚才的 5 种状态以及它们之间的切换我搞清楚了，这 6 种状态它们之间又是怎么切换的呢？

别急，阿粉这么贴心，肯定也是画好了一张图的：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/07/02-Thread-state.png)

这 6 个状态咱们也是分别来看：

- NEW 到 RUNNABLE ，应该是挺容易理解的，就是 thread 调用了 start 方法

	- Java 刚创建出来的 Thread 对象就是 NEW 状态，创建 Thread 对象主要有两种方法，一种是继承 Thread 对象，重写 run() 方法，一种是实现 Runnable 接口，重写 run() 方法，并将该实现类作为创建 Thread 对象的参数

	- 但是还记得嘛， NEW 只是说，这个线程在编程语言层面创建了，在操作系统层面还没有创建，那当然就不会被操作系统调度了对不对，就更谈不上执行了

	- 所以 Java 线程如果想要执行的话，就必须转换到 RUNNABLE 状态，也就是 thread 调用 start 方法

- RUNNABLE 与 BLOCKED ，如果线程等待 synchronized 的隐式锁时，就会从 RUNNABLE 状态转到 BLOCKED 状态。因为 synchronized 修饰的方法/代码块同一时刻只允许一个线程执行，所以其他线程就只能等待了呗，当等待的线程获得 synchronized 隐式锁时，就会从 BLOCKED 状态转到 RUNNABLE 状态

	- 在这里有没有个疑问？就是线程在 wait 一个条件发生时，在操作系统层面线程会转到 waiting 状态，那么在 JVM 层面呢？在 JVM 层面， Java 线程状态是不会发生变化的。也就是这个时候 Java 线程的状态依然是 RUNNABLE 状态

- RUNNABLE 与 WAITING 状态转换，我感觉图已经说得很好了，在这里不再赘述

- RUNNABLE 与 TIMED_WAITING 状态转换，我感觉图已经说得很好了，在这里也不再赘述，仔细观察下会发现， TIMED_WAITING 与 WAITING 相比，就是多了超时参数，毕竟 TIMED_WAITING 是有时限等待嘛

- RUNNABLE 到 TERMINATED ，这个过程比较好理解，线程执行完 run() 方法之后，就自动到 TERMINATED 状态了，当然了如果在执行 run() 方法过程中有异常抛出，也会导致线程终止

	- 有时候我们可能需要强制中断 run() 方法的执行，怎么办呢?是使用 stop() 方法还是 interrupt() 方法呢？正确的姿势是调用 interrupt() 方法

	- stop() 方法会真的杀死线程，不给线程一点儿喘息的机会，如果被杀死的线程持有 synchronized 隐式锁，那就再也不会释放掉这个锁了，接下来的线程也就没办法获得 synchronized 隐式锁，是不是特别危险？同样 suspend() 和 resume() 这两个方法也是不建议使用

	- interrupt() 方法相比于 stop() 方法就温柔很多，它只是通知线程后续的操作可以不用去执行了，线程可以选择执行现在就不执行，当然也可以选择再执行一段时间后再停止，或者我就不听你的，非要执行完，都没关系， interrupt() 只是通知一下你而已。就比如你要做火车去一个地方，突然通知你这个火车晚点了，你可以选择无视这个通知继续等待，或者选择另外一趟高铁，但是不管你做什么，和火车站都没啥关系，它通知的责任尽到了

看到这里应该就比较清楚了吧

在 Java 线程生命周期中， RUNNABLE 状态是将 ready 和 running 两种状态合并在了一起，而 BLOCKED ， WAITING ， TIMED_WAITING 这三种状态其实就是 waiting 状态，也就是线程要等待某些事件发生，才能继续向下执行下去

关于 Java 线程的生命周期，到这里就说完啦

画个图 + 讲解，和面试官扯半小时应该没问题吧？