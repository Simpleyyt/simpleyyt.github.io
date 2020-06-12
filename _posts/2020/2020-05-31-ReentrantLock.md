---
layout: post
categories: Java并发
title: 阿粉写了八千多字，就是为了把 ReentrantLock 讲透
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
关于 ReentrantLock 看这篇文章就够了
<!--more-->

# ReentrantLock 是可重入锁

啥是可重入锁呢？比如：线程 1 通过调用 lock() 方法获取锁之后，再调用 lock 时，就不会再进行阻塞获取锁，而是直接增加重试次数。

还记得 synchronized 吗？它有 monitorenter 和 monitorexit 两种指令来保证锁，而它们的作用可以理解为每个锁对象拥有一个锁计数器，也就是如果再次调用 lock() 方法，计数器会进行加 1 操作

所以， synchronized 和 ReentrantLock 都是可重入锁

![](http://www.justdojava.com/assets/images/2019/java/image-zll/picture/01-biu特否.gif)

# ReentrantLock 与 synchronized 区别

既然 synchronized 和 ReentrantLock 都是可重入锁，那 ReentrantLock 与 synchronized 有什么区别呢？

synchronized 是 Java 语言层面提供的语法，所以不需要考虑异常； ReentrantLock 是 Java 代码实现的锁，所以必须先要获取锁，然后再正确释放锁

synchronized 在获取锁时必须一直等待没有额外的尝试机制； ReentrantLock 可以尝试获取锁（这一点等下分析源码时会看到）

ReentrantLock 支持获取锁时的公平和非公平选择

不 BB 了，直接上源码

![](http://www.justdojava.com/assets/images/2019/java/image-zll/picture/02-noBB.gif)

# lock & NonfairSync & FairSync 详解

## lock

锁的入口是 lock() 方法:

```java
public void lock() {
    sync.lock();
}
```

其中， sync 是 ReentrantLock 的静态内部类，它继承 AQS 来实现重入锁的逻辑， Sync 有两个具体实现类: NonfairSync 和 FairSync

## NonfairSync

先来看一下 NonfairSync :

```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
    * Performs lock.  Try immediate barge, backing up to normal
    * acquire on failure.
    */
    // 重写 Sync 的 lock 方法
    final void lock() {
    	// 先不管其他,上来就先 CAS 操作,尝试抢占一下锁
        if (compareAndSetState(0, 1))
        	// 如果抢占成功,就获得了锁
            setExclusiveOwnerThread(Thread.currentThread());
        else
        	// 没有抢占成功,调用 acquire() 方法,走里面的逻辑
            acquire(1);
    }
	// 重写了 AQS 的 tryAcquire 方法
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

## FairSync

接下来看一下 FairSync :

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

	// 重写 Sync 的 lock 方法
    final void lock() {
        acquire(1);
    }

    /**
    * Fair version of tryAcquire.  Don't grant access unless
    * recursive call or no waiters or is first.
    */
    // 重写了 Sync 的 tryAcquire 方法
    protected final boolean tryAcquire(int acquires) {
    	// 获取当前执行的线程
        final Thread current = Thread.currentThread();
        // 获取 state 的值
        int c = getState();
        // 在无锁状态下
        if (c == 0) {
        	// 没有前驱节点且替换 state 的值成功时
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                // 保存当前获得锁的线程,下次再来时,就不需要尝试竞争锁,直接重入即可
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
        	// 如果是同一个线程来获得锁,直接增加重入次数即可
            int nextc = c + acquires;
            // nextc 小于 0 ,抛异常
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            // 获取锁成功
            return true;
        }
        // 获取锁失败
        return false;
    }
}
```

## 总结 NonfairSync 与 FairSync

到这里，应该就比较清楚了， Sync 有两个具体的实现类，分别是:

- NonfairSync :可以抢占锁，调用 NonfairSync 时，不管当前队列上有没有其他线程在等待，上来我就先 CAS 操作一番，成功了就获得了锁，没有成功就走 acquire 的逻辑；在释放锁资源时，走的是 Sync.nonfairTryAcquire 方法

- FairSync :所有线程按照 FIFO 来获取锁，在 lock 方法中，没有 CAS 尝试，直接就是 acquire 的逻辑；在释放资源时，走的是自己的 tryAcquire 逻辑

接下来咱们看看 NonfairSync 和 FairSync 是如何获取锁的

# ReentrantLock 获取锁

## NonfairSync.lock()

在 NonfairSync 中，获取锁的方法是:

```java
final void lock() {
	// 不管别的,上来就先 CAS 操作,尝试抢占一下锁
    if (compareAndSetState(0, 1))
    	// 如果抢占成功,就获得了锁
        setExclusiveOwnerThread(Thread.currentThread());
    else
    	// 没有抢占成功,调用 acquire() 方法,走里面的逻辑
        acquire(1);
}
```

if 里面没啥说的，咱们来看看 acquire() 方法

### AQS.acquire()

acquire 是 AQS 的核心方法：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

在这里，会先 tryAcquire 去尝试获取锁，如果获取成功，那就返回 true ，如果失败就通过 addWaiter 方法，将当前线程封装成 Node 插入到等待队列中

先来看 tryAcquire 方法:

### NonfairSync.tryAcquire(arg)

在 AQS 中 tryAcquire 方法没有具体实现，只是抛出了异常:

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

NonfairSync 中的 tryAcquire() 方法，才是我们想要看的:

```java
final boolean nonfairTryAcquire(int acquires) {
	// 获取当前执行的线程
    final Thread current = Thread.currentThread();
    // 获取 state 的值
    int c = getState();
    // 当 state 为 0 是,说明此时为无锁状态
    if (c == 0) {
    	// CAS 替换 state 的值,如果 CAS 成功,则获取锁成功
        if (compareAndSetState(0, acquires)) {
        	// 保存当前获得锁的线程,当该线程再次获得锁时,直接重入即可
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 判断是否是同一个线程来竞争锁
    else if (current == getExclusiveOwnerThread()) {
    	// 如果是,直接增加重入次数
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        // 获取锁成功
        return true;
    }
    // 获取锁失败
    return false;
}
```

有没有一种似曾相识的赶脚？在 FairSync 那里，分析过 90% 的代码（好像说分析过 99% 的代码也不过分），只是 FairSync 多了一个判断就是，是否有前驱节点

tryAcquire 分析完毕了，接下来看 addWaiter 方法

### AQS.addWaiter

如果 tryAcquire() 方法获取锁成功，那就直接执行线程的任务就可以了，执行完毕释放锁

如果获取锁失败，就会调用 addWaiter 方法，将当前线程插入到等待队列中，插入的逻辑大概是这样的:

- 将当前线程封装成 Node 节点

- 当前链表中 tail 节点(也就是下面的 pred )是否为空，如果不为空，则 CAS 操作将当前线程的 node 添加到 AQS 队列

- 如果为空，或者 CAS 操作失败，则调用 enq 方法，再次自旋插入

咱们看具体的代码实现:

```java
private Node addWaiter(Node mode) {
	// 生成该线程所对应的 Node 节点
    Node node = new Node(Thread.currentThread(), mode);
    // 将 Node 插入队列中
    Node pred = tail;
    // 如果 pred 不为空
    if (pred != null) {
        node.prev = pred;
        // 使用 CAS 操作,如果成功就返回
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 如果 pred == null 或者 CAS 操作失败,则调用 enq 方法再次自旋插入
    enq(node);
    return node;
}

// 自旋 CAS 插入等待队列
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
        	// 必须初始化,使用 CAS 操作进行初始化
            if (compareAndSetHead(new Node()))
            	// 初始化状态时,头尾节点指向同一节点
                tail = head;
        } else {
            node.prev = t;
            // 如果刚开始就是初始化好的,直接 CAS 操作,将 Node 插入到队尾即可
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

### AQS.acquireQueued

通过 addWaiter 将当前线程加入到队列中之后，会走 `acquireQueued(addWaiter(Node.EXCLUSIVE), arg)` 方法

acquireQueued 方法实现的主要逻辑是:

- 获取当前节点的前驱节点 p

- 如果节点 p 为 head 节点，说明当前节点为第二个节点，那么它就可以尝试获取锁，调用 tryAcquire 方法尝试进行获取

- 调用 tryAcquire 方法获取锁成功之后，就将 head 指向自己，原来的节点 p 就需要从队列中删除

- 如果获取锁失败，则调用 shouldParkAfterFailedAcquire 或者 parkAndCheckInterrupt 方法来决定后面操作

最后，通过 cancelAcquire 方法取消获得锁
看具体的代码实现:

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            // 如果 Node 的前驱节点 p 是 head,说明 Node 是第二个节点,那么它就可以尝试获取锁
            if (p == head && tryAcquire(arg)) {
            	// 如果锁获取成功,则将 head 指向自己
                setHead(node);
                // 锁获取成功之后,将 next 指向 null ,即将节点 p 从队列中移除
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 节点进入等待队列后,调用 shouldParkAfterFailedAcquire 或者 parkAndCheckInterrupt 方法
            // 进入阻塞状态,即只有头结点的线程处于活跃状态
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### shouldParkAfterFailedAcquire

线程获取锁失败之后，会通过调用 shouldParkAfterFailedAcquire 方法，来决定这个线程要不要挂起

shouldParkAfterFailedAcquire 方法实现的主要逻辑:

- 首先判断 pred 的状态是否为 SIGNAL ，如果是，则直接挂起即可

- 如果 pred 的状态大于 0 ，说明该节点被取消了，那么直接从队列中移除即可

- 如果 pred 的状态不是 SIGNAL 也不大于 0 ，进行 CAS 操作修改节点状态为 SIGNAL ，返回 false ，也就是不需要挂起

看一下代码是如何实现的:

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
	// 获取 pred 的状态
    int ws = pred.waitStatus;
    // 如果状态为 SIGNAL ,那么直接返回 true ,挂起线程即可
    if (ws == Node.SIGNAL)
        return true;
    // 如果状态大于 0 ,说明线程被取消
    if (ws > 0) {
    	// 从链表中移除被 cancel 的线程,使用循环来保证移除成功
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
    	// CAS 操作修改 pred 节点状态为 SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    // 不需要挂起线程
    return false;
}
```

到这里，关于 NonfairSync 的获取锁就结束了

接下来咱们看看 FairSync 的获取锁和它有什么不同

## FairSync.lock()

在 FairSync.lock() 方法中是这样的:

```java
final void lock() {
    acquire(1);
}
```

因为 FairSync 是公平锁，所以不存在 CAS 操作去竞争，直接就是调用 acquire 方法

接下来的逻辑就和上面一样了，这里我就不重复了

咱们瞅瞅 ReentrantLock 是怎么释放锁的

# ReentrantLock 释放锁

在 ReentrantLock 释放锁时，调用的是 sync.release() 方法:

```java
public void unlock() {
    sync.release(1);
}
```

点进去发现调用的是 AQS 的 release 方法

## AQS.release()

AQS 的 release 方法比较好理解，就直接看源码了:

```java
public final boolean release(int arg) {
	// 如果释放锁成功
    if (tryRelease(arg)) {
    	// 获取 AQS 队列的头结点
        Node h = head;
        // 如果头结点不为空,且状态 != 0
        if (h != null && h.waitStatus != 0)
        	// 调用 unparkSuccessor 方法唤醒后续节点
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

## ReentrantLock.tryRelease()

在 AQS 中的 tryRelease 方法，只是抛出了异常而已，说明具体实现是由子类 ReentrantLock 来实现的

就直接看 ReentrantLock 中的 tryRelease 方法了

在 ReentrantLock 中实现 tryRelease 方法主要逻辑是:

- 首先，如果是同一个线程获取的同一个锁，那么它有可能被重入多次，所以需要获取到要释放线程的重入次数即 getState()
然后判断，该线程是否为获取到锁的线程，只有获取到锁的线程，才有释放锁一说

- 进行 unlock 释放锁，即:将 state 的值减到 0 ，才算是释放掉了锁，此时才能将 owner 置为 null 同时返回 true

看一下具体实现:

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    // 判断当前线程是否为获取到锁的线程,如果不是则抛出异常
    // 只有获取到锁的线程才释放锁
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 次数为 0 ,说释放锁完毕
    if (c == 0) {
        free = true;
        // 释放之后,当前线程置为 null
        setExclusiveOwnerThread(null);
    }
    // 更新重入次数
    setState(c);
    return free;
}
```

## AQS.unparkSuccessor
释放锁成功之后，接下来要做的就是唤醒后面的进程，这个方法是在 AQS 中实现的

主要逻辑是:

- 获取当前节点状态，如果小于 0 ，则置为 0

- 获取当前节点的下一个节点，如果不为空，直接唤醒

- 如果为空，或者节点状态大于 0 ，则寻找下一个状态小于 0 的节点

代码的具体实现

```java
private void unparkSuccessor(Node node) {
	// 获取当前节点的状态
    int ws = node.waitStatus;
    // 如果节点状态小于 0 ,则进行 CAS 操作设置为 0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
   	// 获取当前节点的下一个节点 s
    Node s = node.next;
    // 如果 s 为空,则从尾部节点开始,或者s.waitStatus 大于 0 ,说明节点被取消
    // 从尾节点开始,寻找到距离 head 节点最近的一个 waitStatus <= 0 的节点
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
    	// next 节点不为空,直接唤醒即可
        LockSupport.unpark(s.thread);
}
```

为什么要从尾节点开始寻找距离 head 节点最近的一个 waitStatus <= 0 的节点呢？

这是因为在 enq() 构建节点的方法中，最后是 `t.next = node` （忘了就再往上翻翻看），设置原来的 tail 的 next 节点指向新的节点

如果在 CAS 操作之后， `t.next = node` 操作之前，有其他线程调用 unlock 方法从 head 开始向后遍历，因为此时 `t.next = node` 还没有执行结束，意味着链表的关系还没有建立好，这样就会导致遍历的时候到 t 节点这里发生中断，因为此时 tail 还没有指向新的尾节点

如果从后向前遍历的话，就不会存在这样的问题

接下来下一个线程就被唤醒了，然后程序会把它当成新的节点开始执行

而原来执行结束的线程，则会将它从队列中移除，然后开始循环循环

这篇文章终于讲完了，阿粉的头发都快秃了

![](http://www.justdojava.com/assets/images/2019/java/image-zll/picture/03-掉头发.gif)

参考：JDK 源码( 1.8 )