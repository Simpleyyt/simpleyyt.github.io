---
layout: post
categories: Java
title: ReentrantLock非公平锁源码分析
tagline: by 小九
tags: 
  - 小九
---

> hello~各位读者好，我是鸭血粉丝（大家可以称呼我为「阿粉」）。今天，阿粉带着大家来了解一下 `ReentrantLock` 锁的非公平锁的实现原理

### 1.锁

**java中,加锁的方式**

	1. synchronized，这个是 java 底层实现的,也就是 C 语言实现的。
 	2. lock，这个是 java.util.concurrent 包下面的，是 java语言实现的。

### 2.ReentrantLock

ReentrantLock 是 Lock 的一种实现，是一种可重入的公平或非公平锁。默认是非公平锁。

#### 2.1 Lock的创建

首先看下锁的创建和使用代码：

```java
//创建锁
Lock lock  = new ReentrantLock();
//加锁
lock.lock();
//释放锁
lock.unlock();
```

然后看下创建的是 ReentrantLock 的构造函数：

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
```

NonfairSync 就是非公平锁。所以 ReentrantLock 默认是非公平锁的实现

#### 2.2 lock()

加锁的逻辑就比较复杂了，因为存在线程竞争。所以有两种情况，一种是竞争到锁的处理，一种是没有竞争到锁的处理。

首先我们还是来看下 lock() 方法，因为最终是非公平的实现，所以直接看 NonfairSync 里面的 lock 方法。

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

#### 2.3 没有获取到锁的逻辑 acquire()

直接上代码：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

还是3个方法，阿粉一个一个的说。

1. tryAcquire(arg)  ，还是先看代码在分析。

   ```java
   final boolean nonfairTryAcquire(int acquires) {
       final Thread current = Thread.currentThread();
       int c = getState();
       if (c == 0) {
           if (compareAndSetState(0, acquires)) {
               setExclusiveOwnerThread(current);
               return true;
           }
       }
       else if (current == getExclusiveOwnerThread()) {
           int nextc = c + acquires;
           if (nextc < 0) // overflow
               throw new Error("Maximum lock count exceeded");
           setState(nextc);
           return true;
       }
       return false;
   }
   ```

   a. 获取 state ，如果等于0，说明之前获得锁的线程已经释放了，那么这个线程就会再次去竞争锁，这就是非公平锁的体现，如果是公平锁，是没有这个判断的。

   b. 如果前一个获得锁的线程没有释放锁，那么就判断是否是同一个线程，是的话就会将 state 加 1。这个就是重入锁的体现。

   c. 如果都不满足,那么返回 false。

2. acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) ,再次获取锁没有成功,并且又不是可重入锁,那么就存入一个阻塞队列里面。里面还有一点逻辑，就不展开了，有兴趣可以自己看下。

3. selfInterrupt(); 这个是当前线程的中断标志，作用就是在线程在阻塞的是否，客户端通过调用了中断线程的方法 interrupt()，那么该线程被唤醒的时候，就会有响应的处理。具体要看这个线程 run 方法里面的代码逻辑。

#### 2.4 unlock()

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

释放锁的步骤

1. state - 1,如果大于0，说明释放的是重入锁，只需要修改 state 就行了
2. 如果等于0，说明要释放锁，释放锁首先需要把独占线程设置为null，再把state设置为0。

### 3 总结

Lock 锁的实现：

1. 互斥性：需要一个状态来判断是否竞争到锁：  state 并且需要用 volatile修饰，保证线程之间的可见性。

2. 可重入性：Thread exclusiveOwnerThread  这个成员变量来记录当前获得锁的线程。

3. 公平或非公平：默认非公平，NonfairSync。

4. 没有竞争到锁的线程怎么办？放到队列中。

5. 没有竞争到锁的线程怎么释放CPU？park：阻塞线程释放CPU资源，这个操作在 acquireQueued()，阿粉没没有讲这个。

6. 最后来张流程图：

   ![1](http://www.justdojava.com\assets\images\2019\java\image-xiaojiu\20200822\1.png)

