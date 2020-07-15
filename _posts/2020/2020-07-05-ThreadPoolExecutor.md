---
layout: post
categories: java并发
title: 源码级分析 ThreadPoolExecutor ，可能是最详细的一篇
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
阿粉万字长文带你解析 ThreadPoolExecutor
<!--more-->

# 为什么要用线程池

你有没有这样的疑惑，为什么要用线程池呢？可能你会说，我可以复用已经创建的线程呀；线程是个重量级对象，为了避免频繁创建和销毁，使用线程池来管理最好了

没毛病，各位都很懂哈~

不过使用线程池还有一个重要的点：可以控制并发的数量。如果并发数量太多了，导致消耗的资源增多，直接把服务器给搞趴下了，肯定也是不行的

# 绕不过去的几个参数

提到 ThreadPoolExecutor 那么你的小脑袋肯定会想到那么几个参数，咱们来瞅瞅源码（我就直接放有 7 个参数的那个方法了）：

```
public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler)
```

咱们分别来看：

- corePoolSize ：

核心线程数，在线程池中有两种线程，核心线程和非核心线程。在线程池中的核心线程，就算是它什么都不做，也会一直在线程池中，除非设置了 allowCoreThreadTimeOut 参数

- maximumPoolSize：

线程池能够创建的最大线程数。这个值 = 核心线程数 + 非核心线程数

- keepAliveTime & unit ：

线程池是可以撤销线程的，那么什么时候撤销呢？一个线程如果在一段时间内，都没有执行任务，那说明这个线程很闲啊，那是不是就可以把它撤销掉了？

所以呢，如果一个线程不是核心线程，而且在 keepAliveTime & unit 这段时间内，还没有干活，那么很抱歉，只能请你走人了
核心线程就算是很闲，也不会将它从线程池中清除，没办法谁让它是 core 线程呢~

- workQueue ：

工作队列，这个队列维护的是等待执行的 Runnable 任务对象

常用的几个队列： LinkedBlockingQueue ， ArrayBlockingQueue ， SynchronousQueue ， DelayQueue

大厂的编码规范，相信各位都知道，并不建议使用 Executors ，最重要的一个原因就是： Executors 提供的很多方法默认使用的都是无界的 LinkedBlockingQueue ，在高负载情况下，无界队列很容易就导致 OOM ，而 OOM 会让所有请求都无法处理，所以在使用时，<strong>强烈建议使用有界队列</strong>，因为如果你使用的是有界队列的话，当线程数量太多时，它会走拒绝策略

- threadFactory ：

创建线程的工厂，用来批量创建线程的。如果不指定的话，就会创建一个默认的线程工厂

- handler ：

拒绝处理策略。在 workQueue 那里说了，如果使用的是有界队列，那么当线程数量大于最大线程数的时候，拒绝处理策略就起到作用了

常用的有四种处理策略：

	- AbortPolicy ：默认的拒绝策略，会丢弃任务并抛出 RejectedExecutionException 异常
	
	- CallerRunsPolicy ：提交任务的线程，自己去执行这个任务
	
	- DiscardOldestPolicy ：直接丢弃新来的任务，也没有任何异常抛出
	
	- DiscardOldestPolicy ：丢弃最老的任务，然后将新任务加入到工作队列中

默认拒绝策略是 AbortPolicy ，会 throw RejectedExecutionException 异常，但是这是一个运行时异常，对于运行时异常编译器不会强制 catch 它，所以就会比较容易忽略掉错误。

所以，如果线程池处理的任务非常重要，<strong>尽量自定义自己的拒绝策略</strong>

# 线程池的几个状态

在源码中，能够清楚地看到线程池有 5 种状态：

```
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

同时，使用 AtomicInteger 修饰的变量 ctl 来控制线程池的状态，而 ctl 保存了 2 个变量：一个是 rs 即 runState ，线程池的运行状态；一个是 wc 即 workerCount ，线程池中活动线程的数量

```
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

- 线程池创建之后就处于 RUNNING 状态

- 调用 shutdown() 方法之后处于 SHUTDOWN 状态，此时线程池不再接受新的任务，清除一些空闲 worker ，等待阻塞队列的任务完成

- 调用 shutdownNow() 方法后处于 STOP 状态，此时线程池不再接受新的任务，中断所有的线程，阻塞队列中没有被执行的任务也会被全部丢弃

- 当线程池中执行的任务为空时，也就是此时 ctl 的值为 0 时，线程池会变为 TIDYING 状态，接下来会执行 terminated() 方法

- 执行完 terminated() 方法之后，线程池的状态就由 TIDYING 转到 TERMINATED 状态

懵了？别急，有张图呢~

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/07/01-线程池状态转换.jpg)

# 线程池处理任务 

## execute

做到线程复用，肯定要先 execute 起来吧

线程池处理任务的核心方法是 execute ，大概思路就是：

- 如果 command 为 null ，没啥说的，直接抛出异常就完事儿了

- 如果当前线程数小于 corePoolSize ，会新建一个核心线程执行任务

- 如果当前线程数不小于 corePoolSize ，就会将任务放到队列中等待，如果任务排队成功，仍然需要检查是否应该添加线程，所以需要重新检查状态，并且在必要时回滚排队；如果线程池处于 running 状态，但是此时没有线程，就会创建线程

- 如果没有办法给任务排队，说明这个时候，缓存队列满了，而且线程数达到了 maximumPoolSize 或者是线程池关闭了，系统没办法再响应新的请求，此时会执行拒绝策略

来瞅瞅源码具体是如何处理的：

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
		
    int c = ctl.get();
    // 当前线程数小于 corePoolSize 时,调用 addWorker 创建核心线程来执行任务
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 当前线程数不小于 corePoolSize ,就将任务添加到 workQueue 中
    if (isRunning(c) && workQueue.offer(command)) {
    	// 获取到当前线程的状态,赋值给 recheck ,是为了重新检查状态
        int recheck = ctl.get();
        // 如果 isRunning 返回 false ,那就 remove 掉这个任务,然后执行拒绝策略,也就是回滚重新排队
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 线程池处于 running 状态,但是没有线程,那就创建线程执行任务
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果放入 workQueue 失败,尝试通过创建非核心线程来执行任务
    // 如果还是失败,说明线程池已经关闭或者已经饱和,会拒绝执行该任务
    else if (!addWorker(command, false))
        reject(command);
}
```

在上面源码中,判断了两次线程池的状态,为什么要这么做呢?

这是因为在多线程环境下,线程池的状态是时刻发生变化的,可能刚获取线程池状态之后,这个状态就立刻发生了改变.如果没有二次检查的话,线程池处于非 RUNNING 状态时, command 就永远不会执行

有点儿懵？阿粉都懂你，一张图走起~

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/07/02-线程执行任务过程.jpg)

## addWorker

从上面能够看出来，主要是 addWorker 方法

addWorker 主要是用来创建核心线程的，它主要的实现逻辑是:

- 判断线程数量有没有超过规定的数量，如果超过了就返回 false

- 如果没有超过，就会创建 worker 对象，并初始化一个 Thread 对象，然后启动这个线程对象

接下来瞅瞅源码:

```
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
		// 线程池状态 >= SHUTDOWN 时,不再接受新的任务,直接返回 false
		// 如果 rs == SHUTDOWN && firstTask == null && ! workQueue.isEmpty() 同样不接受新的任务,返回 false
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
                firstTask == null &&
                ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
			// wc >= CAPACITY 说明线程数不够,所以就返回 false
			// wc >= (core ? corePoolSize : maximumPoolSize) 是在做判断
				// 如果 core 为 true ,说明要创建的线程是核心线程,接下来判断 wc 是否大于 核心线程数 ,如果大于返回 false
				// 如果 core 为 false ,说明要创建的线程是非核心线程,接下来判断 wc 是否大于 最大线程数 ,如果大于返回 false
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
			// CAS 操作增加 workerCount 的值,如果成功跳出循环
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
			// 判断线程池状态有没有变化,如果有变化,则重试
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

	// workerCount 增加成功之后开始走下面的代码
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
		// 创建一个 worker 对象
        w = new Worker(firstTask);
		// 实例化一个 Thread 对象
        final Thread t = w.thread;
        if (t != null) {
			// 接下来的操作需要加锁进行
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
					// 将任务线程添加到线程池中
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
				// 启动任务线程,开始执行任务
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
			// 如果任务线程启动失败调用 addWorkerFailed 
			// addWorkerFailed 方法里面主要做了两件事:将该线程从线程池中移除;将 workerCount 的值减 1
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

## Worker 类

在 addWorker 中，主要是由 Worker 类去做一些相应处理， worker 继承 AQS ，实现 Runnable 接口

线程池维护的是 HashSet<Worker> ，一个由 worker 对象组成的 HashSet

```
private final HashSet<Worker> workers = new HashSet<Worker>();
```

worker 继承 AQS 主要是利用 AQS 独占锁机制，来标识线程是否空闲；另外， worker 还实现了 `Runnable` 接口，所以它本身就是一个线程任务，在构造方法中创建了一个线程，线程的任务就是自己 `this。thread = getThreadFactory().newThread(this);`

咱们瞅瞅里面的源码:

```
	private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        /**
         * This class will never be serialized, but we provide a
         * serialVersionUID to suppress a javac warning.
         */
        private static final long serialVersionUID = 6138294804551838833L;

        // 处理任务的线程
        final Thread thread;
        // worker 传入的任务
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;

        /**
         * Creates with given first task and thread from ThreadFactory.
         * @param firstTask the first task (null if none)
         */
        Worker(Runnable firstTask) {
        	// 将 state 设为 -1 ,避免 worker 在执行前被中断
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
			// 创建一个线程,来执行任务
            this.thread = getThreadFactory().newThread(this);
        }

        /** Delegates main run loop to outer runWorker  */
        public void run() {
            runWorker(this);
        }

        // Lock methods
        //
        // The value 0 represents the unlocked state.
        // The value 1 represents the locked state.

        protected boolean isHeldExclusively() {
            return getState() != 0;
        }

        protected boolean tryAcquire(int unused) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        protected boolean tryRelease(int unused) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        public void lock()        { acquire(1); }
        public boolean tryLock()  { return tryAcquire(1); }
        public void unlock()      { release(1); }
        public boolean isLocked() { return isHeldExclusively(); }

        void interruptIfStarted() {
            Thread t;
            if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                }
            }
        }
    }
```

## runWorker

worker 类在执行 run 方法时，实际上调用的是 runWorker 方法

```
	final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        // 允许中断
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
        	// 判断 task 是否为空,如果不为空直接执行
        	// 如果 task 为空,调用 getTask() 方法,从 workQueue 中取出新的 task 执行
            while (task != null || (task = getTask()) != null) {
            	// 加锁,防止被其他线程中断
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                // 检查线程池的状态,如果线程池处于 stop 状态,则需要中断当前线程
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                	// 执行 beforeExecute 
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                    	// 执行任务
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                    	// 执行 afterExecute 方法
                        afterExecute(task, thrown);
                    }
                } finally {
                	// 将 task 设置为 null ,循环操作
                    task = null;
                    w.completedTasks++;
                    // 释放锁
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
```

在 runWorker 方法中，首先会去执行创建这个 worker 时就有的任务，当执行完这个任务之后， worker 并不会被销毁，而是在 while 循环中， worker 会不断的调用 getTask 方法从阻塞队列中获取任务然后调用 `task。run()` 来执行任务，这样就达到了复用线程的目的。通过循环条件 `while (task != null || (task = getTask()) != null)` 可以看出，只要 getTask 方法返回值不为 null ，就会一直循环下去，这个线程也就会一直在执行，从而达到了线程复用的目的

## getTask

咱们来看看 getTask 方法的实现:

```
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
            // allowCoreThreadTimeOut 变量默认为 false ,也就是核心线程就算是空闲也不会被销毁
            // 如果为 true ,核心线程在 keepAliveTime 内是空闲的,就会被销毁
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

            // 如果运行线程数大于最大线程数,但是缓存队列已经空了,此时递减 worker 数量
            // 如果有设置允许线程超时或者线程数量超过了核心线程数量,并且线程在规定时间内没有 poll 到任务并且队列为空,此时也递减 worker 数量
            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }

            try {
                // 如果 timed 为 true ,会调用 workQueue 的 poll 方法
                	// 超时时间为 keepAliveTime ,如果超过 keepAliveTime 时长的话, poll 就会返回 null 
                	// 如果返回为 null ,在 runWorker 中 
                	// while (task != null || (task = getTask()) != null) 循环条件被打破,从而跳出循环,此时线程执行完毕
                // 如果 timed 为 false ( allowCoreThreadTimeOut 为 false ,并且 wc > corePoolSize 为 false )
                	// 会调用 workQueue 的 take 方法阻塞到当前
                	// 当队列中有任务加入时,线程被唤醒, take 方法返回任务,开始执行
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

源码分析到这里就差不多清楚了

线程复用主要体现在 runWorker 方法中的 while 循环中，在 while 循环里面， worker 会不断的调用 getTask 方法，而在 getTask 方法里，如果任务队列中没有了任务，此时如果线程是核心线程则会一直卡在 workQueue。take 方法，这个时候会被阻塞并挂起，不会占用 CPU 资源，直到拿到任务然后返回 true ， 此时 runWorker 中得到这个任务来继续执行任务，从而实现了线程复用

呦，没想到吧，一不小心就看完了

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/07/03-请勿打扰.gif)