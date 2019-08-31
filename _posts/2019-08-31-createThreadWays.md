# 前言

如何创建线程，共有多少种方式可以说是Java并发编程所面临的第一个问题，如果你在网上搜索这个问题，你会看到参差不齐的答案，有的说2种，有的说4种，更有甚者会说6种。下面这篇文章将带你透过现象看创建线程的本质，彻底搞懂这个问题。

# 官方解释

如果你不信任网上查得的答案，可以到Java官网上查看，其中JDK8的官方文档写的很清楚：

There are two ways to create a new thread of execution. One is to declare a class to be a subclass of Thread.The other way to create a thread is to declare a class that implements the Runnable interface.

通过这两种方式创建线程的代码如下：

1. 通过继承Thread类来创建一个线程

```java
/**
 * 用Thread方式实现线程
 */
public class ThreadStyle extends Thread{

    public static void main(String[] args) {
        new ThreadStyle().start();
    }

    @Override
    public void run() {
        System.out.println("通过继承Thread类创建线程");
    }
}
```

2. 通过实现Runnable接口来创建一个线程

```java
/**
 * 用Runnable方式实现线程
 */
public class RunnableStyle implements Runnable{

    public static void main(String[] args) {
        new Thread(new RunnableStyle()).start();
    }

    @Override
    public void run() {
        System.out.println("通过实现Runnable接口创建线程");
    }
}
```

# 两种官方方式比较

既然官方说了创建线程有两种方式，那么哪种创建线程的方式会更好一点呢？

本文的观点是第二种通过Runnable方式创建线程会比第一种通过继承Thread类创建线程的方式好。有以下三点说明：

1. 从代码架构的方向考虑，第一种方式通过调用Thread类本身的run方法来执行线程，使得Thread类和run()方法耦合在一起，没那么好。而第二种方法是通过运行一个Runnable实例的run()方法来执行线程，完成了解耦，提高了扩展性。
2. 第一种方法每执行一个任务就要新创建一个Thread实例，通过Thread实例完成线程的创建，执行和销毁，开销大。但是方法二就不需要每次创建一个Thread实例，线程池的设计就用的是方法二，节省资源。
3. Java只支持单继承，创建线程如果使用方法一继承了Thread类，将无法继承其他的类（比如一些基类）。

#  官方所说的真的是两种方式吗？

其实网上说的有多少种创建线程方式，在某个角度上你也不能说它是错的，因为他们确实能从代码层面给出多种不同的创建方式。如果把不同的代码实现认为是一种创建线程的方式，那说不定还不只6种。

但是大家有没想过创建线程的原理是什么样的呢？

下面我们看源码

使用方法二，用传入Runnable实现类创建线程的代码其实很简短：

```java
    /* What will be run. */
    private Runnable target;

    //...

    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```

代码中的target，就是我们传入的Runnable实现类；也就是说当我们传入一个实现了Runnable接口的实例给Thread类后，Thread类调用的run方法其实调用的是Runnable接口实例的run方法。

但是如果我们使用继承的方式重写run()方法，运行的时候就相当于子类的run方法覆盖了父类的run方法，所以将不再调用`target.run()`，
而是直接调用子类的run方法。

> 下面看一道面试题加深理解

问：下面这段代码最终会输出什么？

```java
/**
 * 同时使用Runnable和Thread两种实现线程的方式
 */
public class BothRunnableThread {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("我来自Runnable");
            }
        }){
            @Override
            public void run() {
                System.out.println("我来自Thread");
            }
        }.start();
    }
}
```

答案是：输出“我来自Thread”。因为重写了匿名类的run()方法，所以就算传入Runnable实例（给target字段赋了值），也无济于事。当执行线程的失手还是会调用子类重写的run()方法。

> 总结

所以其实实现多线程的本质可以说只有一种方法，也就是通过Thread类的run方法实现。只不过run方法的来源有两种，一种是传入Runnable实现类，调用该实现类的run方法；第二种是调用继承了Thread类的子类的run方法。所以官方把创建新线程的方式定义为两种。

网上还有一些创建线程的说法，比如认为通过线程池创建线程，通过Callable和FutureTask创建线程，使用定时器，使用Lambda表达式 等也认为是一种创建线程的方式。其实这些方法追溯源码最终都会发现离不开官方定义的两种方法。