---
layout: post
title: 面试官因为线程池，让我出门左拐！
catgories: java
tags:
  - 懿
---

前几天阿粉的朋友面试，在面试的时候，面试官问到了框架，项目，JVM还有一些关于线程池的内容，而这个线程池，让阿粉的朋友分分钟被面试官吊打，只能出门左拐，地铁站回家了。为什么呢？因为线程池他是真的没有下功夫去准备，只能凉凉了。
<!--more-->

## 前序

说实话，阿粉在面试的时候，最开始的时候的面试，面试官只是会问实现多线程的方式都有哪些，但是你说到关于线程池的内容的时候，都是一句带过，而有些面试官对这个也不是很细抓，但是自从阿里的面试官开始问关于线程池的问题之后，这个问题就成了高频热点了。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/05-21/1.jpg)

那么接下来，阿粉就继续带给大家关于这个线程池，如何分分钟摆平面试官。

### 1.什么是线程池

`java.util.concurrent.Executors` 这个类大家不知道有没有仔细的去看过这个，而这个类中给我提供了很多方法来创建线程池。

在代码的开头的注释上就写明了，它可以创建重复使用固定数量线程的线程池，如果在所有线程都处于活动状态时提交了其他任务，那么他们将在队列中等待线程可用。
```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

```

而我们创建线程池就是为了解决处理器单元内多个线程执行的问题，它可以显著减少处理器单元的闲置时间，增加处理器单元的吞吐能力。

而面试的时候，我们肯定不能这么说，面试的时候我们可以这么说：

做Java的，当然知道线程池，我们在做开发的时候有时候需要做的任务慢慢的增多，复杂性也会变得越来越强，所以线程的个数就会一点点的往上增加，而对应的线程占用的资源也就越来越多，多个线程占用资源的释放与注销需要维护，这时候多个线程的管理就显得有尤为重要。针对这一情况，sun公司提供了线程池，对线程集合的管理工具。所以线程池就出现了，接下来面试官的问题就是比较狠了，你平常是怎么使用的，几种常见的都有哪些，毕竟面试官的套路一环套一环。

### 2.常见的线程池都有哪些,使用的场景是哪里呢？

这时候这个`java.util.concurrent.Executors` 类大家就排上用场了，比如：

(1) newSingleThreadExecutor
```
单个线程的线程池，即线程池中每次只有一个线程工作，单线程串行执行任务
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }

```
(2)newFixedThreadPool

下面的两个方法是这个方法的重载，而它的意思很明确，建立一个线程数量固定的线程池，规定的最大线程数量，超过这个数量之后进来的任务，会放到等待队列中，如果有空闲线程，则在等待队列中获取，遵循先进先出原则。
```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

  public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

(3)newCacheThreadExecutor

缓存型线程池，这个线程池的意思是在核心线程达到最大值之前，如果继续有任务进来就会创建新的核心线程，并加入核心线程池，即使有空闲的线程，也不会复用。

而达到最大核心线程数后，新任务进来，如果有空闲线程，则直接拿来使用，如果没有空闲线程，则新建临时线程.

而缓存型的线程池使用的是`SynchronousQueue`作为等待队列，他不保存任何的任务，新的任务加入进来之后，他会创建临时线程来进行使用

```

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

```

(4)newScheduledThreadPool

计划型线程池,在它的注释中给出的很明确的解释，创建一个线程池，该线程池可以计划在给定的延迟，或周期性地执行。

也就是说，在新任务到达的时候，我们看到底有没有空闲线程，如果有，直接拿来使用，如果没有，则新建线程加入池。而这里面使用的就是DelayedWorkQueue作为等待队列，中间进行了一定的等待，等待时间过后，继续执行任务。

```
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
            return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    
    public static ScheduledExecutorService newScheduledThreadPool(
                int corePoolSize, ThreadFactory threadFactory) {
            return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
     }


```

### 3.你看过阿里巴巴开发手册么？里面对线程是怎么说的？

说实话，阿粉是一开始真的没怎么注意过这个在阿里巴巴开发手册上关于线程的使用，是怎么做的，而面试官很明显，问出这个问题的时候，肯定是看过了，之后阿粉看了阿里巴巴开发手册，不得不感慨，阿里巴巴，真的是..

我们在日常使用都是会出现这段代码：

```

ExecutorService cachedThreadPool=Executors.newFixedThreadPool();

```
但是阿里巴巴说，不好意思呀，强制线程池不允许使用 Executors 去创建

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/05-21/2.jpg)

那你说嘛，我该怎么办，而推荐的却是 `ThreadPoolExecutor` 

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
    
```

这个方法里面有几个参数
- corePoolSize  要保留在池中的线程数，也就是线程池核心池的大小

- maximumPoolSize 最大线程数

- keepAliveTime 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间。

- unit  keepAliveTime 参数的时间单位

- workQueue 用来储存等待执行任务的队列。

- threadFactory  线程工厂

- handler  默认的拒绝执行处理程序

而这些参数也是面试中经常会问到的呦，而如何选择合适的线程池，如何合理的配置线程池大小，请继续关注阿粉，阿粉将会在最近几天带个大家，点个再看再走呗

