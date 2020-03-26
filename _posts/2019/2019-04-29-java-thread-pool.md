---
layout: post
categories: java
title: 聊聊面试中的 Java 线程池
tagline: by 子悠
tags: 
  - 子悠

---

### 背景
关于 Java 的线程池我想大家肯定不会陌生，在工作中或者自己平时的学习中多多少少都会用到，那你真的有了解过底层的实现原理吗？还是说只停留在用的阶段呢？而且关于 Java 线程池也是在面试中的一个高频的面试题，就像 HashMap 的实现原理一样，基本上面试必问，估计都已经被问烂大街了。
 <!--more-->
> 题外话：HashMap 的实现原理真的已经被问烂了，在我自身的多次面试中都不知道被问了几遍了，有的时候想想很奇怪，为什么这个被问的烂大街的问题还是会一直被问呢？但是从面试官的角度来想一下，如果一个被问的都烂大街的问题你都不好好准备对待，那怎么能好好的对待工作呢（个人愚见）。

 
### 常用的几种线程池
我们先来看下常用的几种线程池的创建方式，以及底层采用的实现原理

单个线程: `Executors.newSingleThreadExecutor();`

```

public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

```


缓存线程: `Executors.newCachedThreadPool();`

```

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

```


固定线程`Executors.newFixedThreadPool(2);`

```

public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

```


定时线程: `Executors.newScheduledThreadPool(3);`（父类中）

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

### 核心 `ThreadPoolExecutor`
通过上面的几个线程池的底层实现，我们可以发现底层都是通过 `ThreadPoolExecutor` 类来实现的，只是参数不一样，那我们就很有必要来看一下`ThreadPoolExecutor` 这个类了
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/thread-pool1.jpg)

```

public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

```

通过 JDK 的源码我们可以看到 `ThreadPoolExecutor` 在 Java 的 concurrent 包下面，并且有四个构造方法，下面依次介绍下各个参数的含义：
- corePoolSize: 核心线程数的大小
- maximumPoolSize: 线程池中允许的最大线程数
- keepAliveTime: 空闲线程允许的最大的存活时间
- unit: 存活时间的单位
- workQueue: 阻塞任务队列
- threadFactory: 线程工厂用来创建线程
- handler: 拒绝策略，针对当队列满了时新来任务的处理方式

通过上面参数的分析，我们可以知道，单个线程的线程池就是线程池中只有一个线程负责任务，所以 corePoolSize 和 maximumPoolSize 的数值都是为 1；当这个线程出现任何异常后，线程池会自动创建一个线程，始终保持线程池中有且只有一个存活的线程。而且其他线程池也只是参数的设置不一样而已。
我们还需要知道几个常见的线程池类和接口的关系，以及一些方法，如下图
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/thread-pool2.png)
> `ThreadPoolExecutor` 继承 `AbstractExecutorService`；`AbstractExecutorService` 实现 `ExecutorService`， `ExecutorService` 继承 `Executor`

### 源码分析
根据源码可以发现整个线程池大致分为 3 个部分，1. 是创建 worker 线程，2. 添加任务到 workQueue; 3.worker 线程执行具体任务

创建 worker 线程，现在我们来看下核心的 `execute(Runnable command)` 方法，如果工作线程小于指定的核心线程数时会尝试去创建新的线程，

```

public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();

    int c = ctl.get();
    //如果工作线程比核心线程数少，则创建新线程
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}


```

再看下`addWorker(Runnable firstTask, boolean core)` 方法

```

private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}

```
添加任务到 workQueue，这个阻塞队列内部的方法

```

public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    final AtomicInteger count = this.count;
    if (count.get() == capacity)
        return false;
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        if (count.get() < capacity) {
            enqueue(node);
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        }
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
    return c >= 0;
}

```

worker 线程执行具体任务，阻塞或者超时去获取队列中的任务，进行执行

```

final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        //阻塞循环获取任务
        while (task != null || (task = getTask()) != null) {
            w.lock();
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}


private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}


```

在刚刚创建线程池的时候，内部线程的数量是 0，当首个任务进行添加的时候，会根据参数的配置进行线程的创建，并随着任务数的增加，会逐渐创建新的线程直到不能创建新的线程为止。不能创建新的线程后，会将来的任务存放到阻塞队列中，让空闲的线程去处理。当没有空闲线程并且队列满了时候就会采用拒绝策略去丢弃或者其他策略来处理。
拒绝策略主要有四种，不能的拒绝策略有不同的使用场景，需要根据情况决定使用。
- `CallerRunsPolicy` : 调用线程处理任务
- `AbortPolicy` : 抛出异常
- `DiscardPolicy` : 直接丢弃
- `DiscardOldestPolicy` : 丢弃队列中最老的任务，执行新任务

### 小结
线程池在工作中的使用必不可少，如何优雅的使用线程池能很大程度的提升性能和效率。根据实际的应用场景，配置合适的线程池参数可以很大的提升项目的性能，也可以充分利用服务器的性能。
