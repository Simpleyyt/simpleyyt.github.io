---
layout: post
categories: 程序生活
title: 面试官让我手写一个读写锁出来，我...
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
扶我起来，我还能学
<!-- more -->

上周六发了一篇文章： [有个程序媛女朋友是什么体验？](https://mp.weixin.qq.com/s/Txp4zY4HcIZ7DYPARQTdYQ) ，把阿粉酸的，周六日都没缓过来。但是阿粉又是一个比较负责任的博主，酸归酸，文章还是要更新的

题目是个标题党啦，就是想带你过一遍 ReentrantReadWriteLock ，为了让可爱的读者们多看几眼阿粉的文章，可真是费劲了心思，就问你我暖不暖

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/20-哄我.gif)

## ReentrantReadWriteLock 与 ReentrantLock 区别？

在这篇文章中: [阿粉写了八千多字，就是为了把 ReentrantLock 讲透](https://mp.weixin.qq.com/s/Yy_mz5DNB6dq-TSfrmkvUQ) 阿粉对 ReentrantLock 已经做了非常详细的讲解了

那么，今天想要说的 ReentrantReadWriteLock 和 ReentrantLock 有什么区别呢？如果只是从名字上来说的话，就是多了一个 ReadWrite 嘛

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/21-机灵鬼.gif)

如果对 ReentrantLock 比较熟的话，那么当阿粉问你， ReentrantLock 是独占锁还是共享锁，你的第一反应就是: 独占锁

ReentrantReadWriteLock 是在 ReentrantLock 的基础上做的优化，什么优化呢？ ReentrantLock 就是不管操作是读操作还是写操作都会对资源进行加锁，但是聪明的你想想嘛，如果好几个操作都只是读的话，并没有让数据的状态发生改变，这样的话是不是可以允许多个读操作同时运行？这样去处理的话，相对来说是不是就提高了并发呢

很多事情都是说起来容易，具体是怎么实现的呢？

啥也不多说，咱们直接上源码好吧

在使用 ReentrantReadWriteLock 时，一般都是调用 `writeLock` 和 `readLock` 两种方法，它在源码中定义如下:

```java
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
```

而 `writeLock` 和 `readLock` 是 ReentrantReadWriteLock 的两个内部类，其中这两种锁的实现如下（其中省略了一些代码）：

```java
public static class ReadLock implements Lock, java.io.Serializable {
    public void lock() {
    	// 共享锁
        sync.acquireShared(1);
    }
		
	public void unlock() {
		// 共享锁
        sync.releaseShared(1);
    }
}

public static class WriteLock implements Lock, java.io.Serializable {
    public void lock() {
    	// 独占锁
        sync.acquire(1);
    }
	
	public void unlock() {
		// 独占锁
        sync.release(1);
    }
}
```

从源码就能够看出，对于读锁 readLock 它使用的是共享锁，也就是多个线程读没问题
但是对于写锁 writeLock 它使用的是独占锁，就是当一个线程要进行写操作时，其他的线程都要停下来等待

简单点儿说就是：一个资源可以被多个读线程访问，或者被一个写线程访问，但是不能同时存在读线程和写线程，这也是读写锁的定义

## ReadLock 和 WriteLock 共享一个变量

如果让你设计一个读写锁的话，会怎样设计？

阿粉还真的认真想了想这个问题，如果让我设计的话，我应该会用两个变量去控制读和写，当线程获取到读锁时就对读变量进行 +1 操作，当获取到写锁时，就对写变量进行 +1 操作

但是通过看 ReentrantReadWriteLock 源码发现，它只是通过一个 state 来实现的

具体实现如下:

```java
static final int SHARED_SHIFT   = 16;
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

/** Returns the number of shared holds represented in count  */
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
/** Returns the number of exclusive holds represented in count  */
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

有两个关键方法 `sharedCount` 和 `exclusiveCount` ，乖，光是从名字意思来看应该也是可以猜出来的吧: `sharedCount` 就是共享锁的数量，而 `exclusiveCount` 则是独占锁的数量

通过看源码，能够看出来，对于 `sharedCount` 来说，它的计算方式就是无符号右移 16 位，空位都以 0 来补齐( `c >>> SHARED_SHIFT;` )

对于 `exclusiveCount` 来说，它的计算方式就是将传进来的 c 和 `EXCLUSIVE_MASK `做 “&” 运算，那么 `EXCLUSIVE_MASK` 的值是什么呢？就是 `(1 << SHARED_SHIFT) - 1` ，如果对位运算比较熟的话，应该会很容易看出来 `(1 << SHARED_SHIFT) - 1` 的值就是 65535 ，化成二进制就是 16 个 1，传进来 c 的值，和 16 位全为 1 做 “&” 运算的话，只有 1 & 1 才为 1 ，也就是说，传进来的 c 值经过这样转换之后，还是原来的值

说到这里，可能有点儿懵了，没关系，咱们来个总结就好说了(为了好理解，我就用大白话说了，争取你们都能看懂

对于 `sharedCount` 来说，只要传进来的值不大于 65535 ，那么经过计算之后，值都是 0

对于 `exclusiveCount` 来说，传进来的值是多少，经过计算之后还是多少

不管是 `sharedCount` 还是 `exclusiveCount` ，最大值都是 65535 ，因为是和 16 做位运算，其实这个数字也是相当够用了

那么，看到这里，各位应该就比较了解了吧，对于 ReadLock 和 WriteLock 来说，在源码层次其实并不是用两个变量去做的，而是通过一个 state 来实现的，思路真的是非常的巧妙

对于阿粉上面说的，如果还是不清楚的话，可以自己写个 demo 去验证一下，很简单的，就比如下面这样:

```java
public static void main(String[] args) {
    int shareCount = 3000 >>> 16;
    System.out.println("shareCount : " + shareCount);

    int exclusiveCount = 1 & ((1 << 16) - 1);
    System.out.println("exclusiveCount : " + exclusiveCount);
}
```

等你运行完之后，你就发现，哇，怎么和我说的一样，哈哈哈哈

对于 `sharedCount` 来说，它是针对读锁的，所以不管多少进程进行读取资源，都没关系，所以它的值就是 0

对于 `exclusiveCount`来说，它是针对写锁的，那么只要有一个进程在进行写入，其他线程都要停下来等待，所以它的值就是传进来的值

综上，使用一个状态的话，我们只需要去判断 state 这个状态就可以了

## WriteLock 的具体实现

OK ，既然你都看到了这里，我就默认上面的内容你都理解了

WriteLock 说白了就是独占锁，所以在获取 WriteLock 时，不能只考虑是否有写锁在占用，还要考虑有没有读锁.接下来咱们就去探究一下， WriteLock 它具体是怎么实现的

```java
public void lock() {
    sync.acquire(1);
}
        
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    // 获取到锁的状态
    int c = getState();
    // 获取写锁的数量
    int w = exclusiveCount(c);
    // c != 0 说明有读锁/写锁
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        // w == 0 说明此时没有写锁,有读锁 或者 持有写锁的线程不是当前线程
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        // 如果写锁数量超出了最大值,没啥说的,抛异常就完事儿了
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        // 当前线程持有写锁,为重入锁,直接 +acquires 即可
        setState(c + acquires);
        return true;
    }
    // CAS 操作,确保修改值成功
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

如果对 ReentrantLock 比较熟的话，你会发现，上面的代码大部分都是见过的
有一点区别就是调用了 `exclusiveCount` 方法，看当前是否有写锁存在，接下来通过 `c != 0 and w == 0` 判断了当前是否有读锁存在

## ReadLock 的具体实现

WriteLock 探究完了，接下来瞅瞅 ReadLock ，话不多说，直接上源码

```java
public void lock() {
    sync.acquireShared(1);
}

public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    // 写锁不等于 0 时,看看当前写锁是否在尝试获取读锁
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    // 获取读锁数量
    int r = sharedCount(c);
    // 读锁不需要阻塞,而且读锁需要小于最大读锁数量,同时 cas 操作进行加 1 操作
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        // 当前线程是第一个并且第一次获取读锁时
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            // 如果当前线程再次获取读锁,则直接进行 ++ 操作即可
            firstReaderHoldCount++;
        } else {
            // 当前线程不是第一个获取读锁的线程,就放入线程本地变量
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

看完有没有觉得和写锁那块挺像的，不同就在于因为是读锁嘛，所以只要没有写锁占用，而且读锁的数量没有超过最大的获取数量，就都可以获取读锁

在上面， `firstReader` `firstReaderHoldCount` `cachedHoldCounter` 都是为 `readHolds` 服务的，它是为了获取当前线程持有锁的数量，在 T`hreadLocal` 基础上添加了 Int 变量来统计，这样比较方便嘛

具体实现如下:

```java
static final class ThreadLocalHoldCounter
    extends ThreadLocal<HoldCounter> {
    public HoldCounter initialValue() {
        return new HoldCounter();
    }
}

static final class HoldCounter {
    // 当前线程持有锁的数量
    int count = 0;
    // Use id, not reference, to avoid garbage retention
    // 当前线程 ID
    final long tid = getThreadId(Thread.currentThread());
}
```

## 回到题目，手写一个读写锁出来？

接下来，再回到题目，如果面试官让手写一个读写锁出来，你会如何实现呢？

在读了源码之后，相信你心里应该有谱了

首先来个 state 变量，然后高 16 位设置为读锁数量，低 16 位设置为写锁数量低，然后在进行读锁时，先判断下是不是有写锁，如果没有，直接读取即可，如果有的话那就需要等待；在写锁想要拿到锁的时候，就要判断写锁和读锁是不是都存在了，如果存在那就等着，如果不存在才进行接下来的操作

在这里阿粉给出一个简单版的实现:

```java
 public static class ReadWrite{
    // 定义一个读写锁共享变量 state
    private int state = 0;

    // state 高 16 位为读锁数量
    private int getReadCount(){
        return state >>> 16;
    }

    // state 低 16 位为写锁数量
    private int getWriteCount(){
        return state & (( 1 << 16 ) - 1 );
    }

    // 获取读锁时,先判断是否有写锁
    // 如果有写锁,就等待
    // 如果没有,进行加 1 操作
    public synchronized void lockRead() throws InterruptedException{
        while ( getWriteCount() > 0){
            wait();
        }

        System.out.println("lockRead --- " + Thread.currentThread().getName());
        state = state + ( 1 << 16);
    }

    // 释放读锁数量减 1 ,通知其他线程
    public synchronized void unLockRead(){
        state = state - ( 1 << 16 );
        notifyAll();
    }

    // 获取写锁时需要判断读锁和写锁是否都存在,有则等待,没有则将写锁数量加 1
    public synchronized void lockWrite() throws InterruptedException{

        while (getReadCount() > 0 || getWriteCount() > 0) {
            wait();
        }
        System.out.println("lockWrite --- " + Thread.currentThread().getName());
        state ++;
    }

    // 释放写锁数量减 1 ,通知所有等待线程
    public synchronized void unlockWriters(){
        state --;
        notifyAll();
    }
}
```

阿粉自己测试了下，没啥大问题

但是如果细究的话，还是有问题的，就比如，如果现在我有好多个读锁，如果一直不释放的话，那么写锁是一直没办法获取到的，这样就造成了饥饿现象的产生嘛

解决的话也蛮好解决的，就是在上面添加一个记录写锁数量的变量，然后在读锁之前，去判断一下是否有线程要获取写锁，如果有的话，优先处理，没有的话再进行读锁操作

这块聪明的读者，就试试自己实现吧，阿粉这里就不给具体实现了

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/22-可爱.gif)
