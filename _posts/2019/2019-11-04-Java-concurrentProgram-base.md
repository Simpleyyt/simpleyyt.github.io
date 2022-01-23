---
layout: post  
title: java并发编程之线程基础
tagline: by 小九
categories: java线程
tags: 
    - 小九

---

### 01、简介

百丈高楼平地起，要想学好多线程，首先还是的了解一下线程的基础，这边文章将带着大家来了解一下线程的基础知识。

### 02、线程的创建方式

1. 实现 `Runnable` 接口
2. 继承 `Thread` 类
3. 实现 `Callable` 接口通过 `FutureTask` 包装器来创建线程
4. 通过线程池创建线程

​	下面将用线程池和 ``Callable` ` 的方式来创建线程

```java
public class CallableDemo implements Callable<String> {

    @Override
    public String call() throws Exception {
        int a=1;
        int b=2;
        System. out .println(a+b);
        return "执行结果:"+(a+b);
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建一个可重用固定线程数为1的线程池
        ExecutorService executorService = Executors.newFixedThreadPool (1);
        CallableDemo callableDemo=new CallableDemo();
        //执行线程,用future来接收线程的返回值
        Future<String> future = executorService.submit(callableDemo);
        //打印线程的返回值
        System. out .println(future.get());
        executorService.shutdown();
    }
}
```

执行结果

```java
3
执行结果:3
```

### 03、线程的生命周期

1. NEW：初始状态，线程被构建，但是还没有调用 start 方法。
2. RUNNABLED：运行状态，JAVA 线程把操作系统中的就绪和运行两种状态统一称为“运行中”。调用线程的 `start()` 方法使线程进入就绪状态。
3. BLOCKED：阻塞状态，表示线程进入等待状态,也就是线程因为某种原因放弃了 CPU 使用权。比如访问 `synchronized` 关键字修饰的方法，没有获得对象锁。
4.  Waiting ：等待状态，比如调用wait()方法。
5. TIME_WAITING：超时等待状态，超时以后自动返回。比如调用 `sleep(long millis)` 方法
6. TERMINATED：终止状态，表示当前线程执行完毕。

看下源码：

```java
public enum State {
        
        NEW,

        RUNNABLE,
        
        BLOCKED,
         
        WAITING,
       
        TIMED_WAITING,
        
        TERMINATED;
}
```



### 04、线程的优先级

1. 线程的最小优先级：1
2. 线程的最大优先级：10
3. 线程的默认优先级：5
4. 通过调用 `getPriority()` 和 `setPriority(int newPriority)` 方法来获得和设置线程的优先级

看下源码：

```java
	/**
     * The minimum priority that a thread can have.
     */
    public final static int MIN_PRIORITY = 1;

    /**
     * The default priority that is assigned to a thread.
     */
    public final static int NORM_PRIORITY = 5;

    /**
     * The maximum priority that a thread can have.
     */
    public final static int MAX_PRIORITY = 10;
```

看下代码：

```java
public class ThreadA extends Thread {

    public static void main(String[] args) {
        ThreadA a = new ThreadA();
        System.out.println(a.getPriority());//5
        a.setPriority(8);
        System.out.println(a.getPriority());//8
    }
}
```

###### 线程优先级特性：

1. 继承性：比如A线程启动B线程，则B线程的优先级与A是一样的。
2. 规则性：高优先级的线程总是大部分先执行完，但不代表高优先级线程全部先执行完。
3. 随机性：优先级较高的线程不一定每一次都先执行完。

### 05、线程的停止

1. `stop()` 方法，这个方法已经标记为过时了，强制停止线程，相当于kill -9。
2. `interrupt()` 方法，优雅的停止线程。告诉线程可以停止了，至于线程什么时候停止，取决于线程自身。

看下停止线程的代码：

```java
public class InterruptDemo {
    private static int i ;
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            //默认情况下isInterrupted 返回 false、通过 thread.interrupt 变成了 true
            while (!Thread.currentThread().isInterrupted()) {
                i++;
            }
            System.out.println("Num:" + i);
        }, "interruptDemo");
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        thread.interrupt(); //不加这句,thread线程不会停止
    }
}
```

看上面这段代码，主线程main方法调用 `thread`线程的 `interrupt()` 方法，就是告诉 `thread` 线程，你可以停止了（其实是将 `thread` 线程的一个属性设置为了true），然后 `thread` 线程通过 `isInterrupted()` 方法获取这个属性来判断是否设置为了true。这里我再举一个例子来说明一下，

看代码：

```java
public class ThreadDemo {
    private volatile static Boolean interrupt = false ;
    private static int i ;

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (!interrupt) {
                i++;
            }
            System.out.println("Num:" + i);
        }, "ThreadDemo");
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        interrupt = true;
    }
}
```

是不是很相似，再简单总结一下：

当其他线程通过调用当前线程的 interrupt 方法，表示向当前线程打个招呼，告诉他可以中断线程的执行了，并不会立即中断线程，至于什么时候中断，取决于当前线程自己。

线程通过检查资深是否被中断来进行相应，可以通过 isInterrupted() 来判断是否被中断。

这种通过标识符来实现中断操作的方式能够使线程在终止时有机会去清理资源，而不是武断地将线程停止，因此这种终止线程的做法显得更加安全和优雅。

### 06、线程的复位

两种复位方式：

1. Thread.interrupted()
2. 通过抛出 InterruptedException 的方式

然后了解一下什么是复位：

线程运行状态时 Thread.isInterrupted() 返回的线程状态是 false，然后调用 thread.interrupt() 中断线程 Thread.isInterrupted() 返回的线程状态是 true，最后调用 Thread.interrupted() 复位线程Thread.isInterrupted() 返回的线程状态是 false 或者抛出 InterruptedException 异常之前，线程会将状态设为 false。

下面来看下两种方式复位线程的代码，首先是 Thread.interrupted() 的方式复位代码：

```java
public class InterruptDemo {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (true) {
                //Thread.currentThread().isInterrupted()默认是false,当main方式执行thread.interrupt()时,状态改为true
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("before:" + Thread.currentThread().isInterrupted());//before:true
                    Thread.interrupted(); // 对线程进行复位，由 true 变成 false
                    System.out.println("after:" + Thread.currentThread().isInterrupted());//after:false
                }
            }
        }, "interruptDemo");
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        thread.interrupt();
    }
}
```

抛出 InterruptedException 复位线程代码：

```java
public class InterruptedExceptionDemo {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    // break;
                }
            }
        }, "interruptDemo");
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        thread.interrupt();
        System.out.println(thread.isInterrupted());
    }
}
```

结果：

```java
false
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at com.cl.concurrentprogram.InterruptedExceptionDemo.lambda$main$0(InterruptedExceptionDemo.java:16)
	at java.lang.Thread.run(Thread.java:748)
```

需要注意的是，InterruptedException 异常的抛出并不意味着线程必须终止，而是提醒当前线程有中断的操作发生，至于接下来怎么处理取决于线程本身，比如
1. 直接捕获异常不做任何处理
2. 将异常往外抛出
3. 停止当前线程，并打印异常信息

像我上面的例子,如果抛出 InterruptedException 异常,我就break跳出循环让 thread 线程终止。

###### 为什么要复位：
Thread.interrupted()是属于当前线程的，是当前线程对外界中断信号的一个响应，表示自己已经得到了中断信号，但不会立刻中断自己，具体什么时候中断由自己决定，让外界知道在自身中断前，他的中断状态仍然是 false，这就是复位的原因。