---
layout: post
title:  线程之守护线程
tagline: by 懿
categories: java
tag: 
    - java
---

多线程一直是我们开发中最关注的一个点，因为在并发中，会有各种各样的问题，但是这中多线程中的问题，又是我们需要去解决的，
之前看书看到过一点内容，也和大家分享一下关于守护线程的一些知识。
<!--more-->

## 什么是守护线程

JAVA提供了两种线程：守护线程和用户线程，那么什么是守护线程呢？

守护线程又被称之为“服务进程”， “精灵线程”或者是“后台线程”，是指在程序运行的时候在后台提供一种通用服务的线程。
这种线程并不属于程序设计中不可或缺的部分。我们通俗一点说，任何一个守护线程都是整个JVM中所有的非守护线程的一个“保姆”。

用户线程和守护线程几乎是一样的，唯一的不同之处就是在于如果说用户线程已经全部退出了运行，只剩下守护线程存在了，那么JVM也就回相对应的
退出了，因为当所有的非守护线程结束的时候，没有任何线程需要去被守护，那么守护线程就没工作可以继续做了，那么就说明程序是被终止了，
这时候所有的守护线程都会被“杀死”。也就是说，只要有任何一个非守护线程还处在运行当中的话，那么守护线程就不会终止。

但是我们要注意一点，一个线程默认不是守护线程。

我们可以用一个例子来证明一下：

请看下面的Demo

```

public class DaemonThreadTest extends Thread {
    public static void main(String[] args) {
        DaemonThreadTest daemonThreadTest = new DaemonThreadTest("后台线程");
        System.out.println("是否是守护线程"+"---"+daemonThreadTest.isDaemon());
    }

    public DaemonThreadTest(String name){
       super();
    }
}

```

![](/assets/images/2019/java/image_yi/06-03/1.jpg)

结果就像图中所示的，一个线程不是守护线程。

但是在JAVA语言中，守护线程一般都具有的优先级都是比较低的，他并非是只有JVM内部来提供的，用户在编写程序的时候也可以自己去设置守护
线程，例如，将一个用户线程设置为守护线程的方法就是在调用start（）方法的启动线程之前调用对象的setDaemom(true)这个方法，
若将以上的参数设置为false的时候，则表示的是用户进程模式，需要注意的是，当在一个守护线程中产生了其他的线程，那么这些新产生的
线程默认会是守护线程，用户线程也是如此，

我们看一下代码实例

```

public class DaemonThreadTest2 extends Thread{
    public void  run(){
        System.out.println(Thread.currentThread().getName()+":begain");

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName()+":end");

    }
}
class Test{
    public static void main(String[] args) {
        System.out.println("test1:begain");
        DaemonThreadTest2 daemonThreadTest2 = new DaemonThreadTest2();
        daemonThreadTest2.setDaemon(true);
        daemonThreadTest2.start();
        System.out.println("test1:end");
    }

}

```

大家可以简单的想一下结果是什么？

结果其实是这样的

![](/assets/images/2019/java/image_yi/06-03/2.jpg)

从运行的记过中我们也可以发现，没有输出Thread-0：end，那么为什么结果会是这种样子的呢？

其实就是在启动线程前，我们将其设置为守护线程了，当程序中只有守护线程存在的时候，JVM是可以退出的，
额就是说，当JVM中只有守护线程运行的时候，JVM会自动关闭。

因此，当test1方法在调用的结束后，main线程将会退出，这时候线程daemonThreadTest2还处于休眠状态没有运行结束，但是这个时候只有守护线程在
运行了，JVM关闭了，因此它就不会输出“Thread-0：end”了。

守护线程的一个典型的例子就是垃圾回收器，只要JVM启动，他就会一直运行，实时的区监控可以被回收的资源。

是用守护线程要注意几点：

- thread.setDaemon(true)必须在thread.start()之前设置，否则会跑出一个IllegalThreadStateException异常。你不能把正在运行的常规线程设置为守护线程。

- 在Daemon线程中产生的新线程也是Daemon的。

- 不要认为所有的应用都可以分配给Daemon来进行服务，比如读写操作或者计算逻辑。 


以上就是我所介绍的守护线程，你了解了么？



