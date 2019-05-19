---
layout: post
title:  云阶月地，关锁千重(一.独享锁/共享锁)
tagline: by 懿
categories: java
tag: 
    - java
---

之前在的文章中已经写了公平锁和非公平锁了，接下来就该介绍第二种锁了，他就是共享锁和独享锁，顾名思义，独享，只能被一个线程
所持有，而共享，就是说可以被多个线程所共有。
<!--more-->

## 锁的分类

 1. 公平锁/非公平锁
 2. 可重入锁
 3. 独享锁/共享锁
 4. 互斥锁/读写锁
 5. 乐观锁/悲观锁
 6. 分段锁
 7. 偏向锁/轻量级锁/重量级锁
 8. 自旋锁
 
 之前的第一次分享中我们已经说过了公平锁和非公平锁了，这次我们组要是来解析一下这个独享锁和共享锁。
 
 ### 独享锁
 
 独享锁其实有很多名称的，有人称它为独享锁，有人也称它为独占锁，其实大致上都是一个意思，
 
 独享锁，只能够被一个线程所持有，
 
 而他的实例我们之前的公平锁和非公平锁也都说过一次，我们可以再看一下这个实例，
 
### ReentrantLock(独享)

ReentrantLock是基于AQS来实现的，那什么是AQS呢？ 

AQS全称AbstractQueuedSynchronizer,如果说使用翻译软件来看“摘要排队同步器”，但是很多人喜欢称它为抽象队列同步器。
其实叫什么倒是没有那么重要，只要记住英文，这才是最重要的。

AQS它定义了一套多线程访问共享资源的同步器框架，很多类都是依赖于AQS来比如说我们一会将要介绍的ReentrantLock。

你看源码

```
/*
    查询是否有任何线程正在等待与此锁相关联的给定条件。
    请注意，由于超时和*中断可能随时发生，
    此方法主要用于监视系统状态
*/
 public boolean hasWaiters(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
}

```

这里就指明了我们说的ReentrantLock是依赖AQS的，而AQS它是JUC并发包中的一个核心的一个组件。
也是不可或缺的组件。

AQS解决了子啊实现同步器时涉及当的大量细节问题，例如获取同步状态、FIFO同步队列。基于AQS来构建同步器可以带来很多好处。它不仅能够极大地减少实现工作，而且也不必处理在多个位置上发生的竞争问题。

在基于AQS构建的同步器中，只能在一个时刻发生阻塞，从而降低上下文切换的开销，提高了吞吐量。

AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态。

咱们可以看一下

```

public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {

    private static final long serialVersionUID = 7373984972572414691L;
    
    
```

而它典型的例子ReentrantLock中：

使用一个int类型的成员变量state来表示同步状态，当state>0时表示已经获取了锁

这就是我们之前看的int c = getState()；

而当c等于0的时候说明当前没有线程占有锁，它提供了三个方法（getState()、setState(int newState)、compareAndSetState(int expect,int update)）来对同步状态state进行操作，所以AQS可以确保对state的操作是安全的。

关于AQS我就解释这么多把，如果想深入了解的可以仔细的研究一下，而在这个ReentrantLock中的源码是这样的

```
/**
它默认是非公平锁
*/
public ReentrantLock() {
    sync = new NonfairSync();
}
   /**
   创建ReentrantLock，公平锁or非公平锁
   */
public ReentrantLock(boolean fair) {
     sync = fair ? new FairSync() : new NonfairSync();
 }
 /**
 而他会分别调用lock方法和unlock方法来释放锁
 */
 public void lock() {
         sync.lock();
     }
  
 public void unlock() {
         sync.release(1);
     }

```

但是其实他不仅仅是会调用lock和unlock方法，因为我们的线程不可能一点问题没有，如果说进入到了waiting状态，在这个时候如果没有unpark()方法，就没有办法来唤醒他，
所以，也就接踵而至出现了tryLock(),tryLock(long,TimeUnit)来做一些尝试加锁或者说是超市来满足某些特定的场景的需求了。

ReentrantLock会保证method-body在同一时间只有一个线程在执行这段代码，或者说，同一时刻只有一个线程的lock方法会返回。其余线程会被挂起，直到获取锁。

从这里我们就能看出，其实ReentrantLock实现的就是一个独占锁的功能：有且只有一个线程获取到锁，其余线程全部挂起，直到该拥有锁的线程释放锁，被挂起的线程被唤醒重新开始竞争锁。

而在源码中通过AQS来获取独享锁是通过调用acquire方法，其实这个方法是阻塞的, 

```
/**
*以独占模式获取，忽略中断。通过至少调用tryAcquire实现
成功返回。否则线程排队，可能重复阻塞和解除阻塞，
调用tryAcquire直到成功。
此方法可用于实现方法lock。
*/
 public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    
```
它通过tryAcquire（由子类Sync实现）尝试获取锁，这也是上面源码中的lock方法的实现步骤

而没有获取到锁则调用AQS的acquireQueued方法：

```

final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
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

这段的意思大致就是说
当前驱节点是头节点，并且独占时才返回

而在下面的if判断中，他会去进行阻塞，而且还要去判断是否打断，如果我们的节点状态是Node.SIGNAL时，
完蛋了，线程将会执行parkAndCheckInterrupt方法，知道有线程release的时候，这时候就会进行一个unpark来循环的去获取锁。
而这个方法通过LockSupport.park(this)将当前的线程挂起到WATING的状态，就需要我们去执行unpark方法了来唤醒他，也就是我说的那个release，
通过这样的一种FIFO机制的等待就实现了LOCK的操作。

这上面的代码只是进行加锁，但是没有释放锁，如果说我们获得了锁不进行释放，那么很自然的出现一种情况，死锁！

所以必须要进行一个释放，

我们来看看内部是怎么释放锁的

```

 public void unlock()              { sync.release(1); }
 
 public final boolean release(int arg) {
         if (tryRelease(arg)) {
             Node h = head;
             if (h != null && h.waitStatus != 0)
                 unparkSuccessor(h);
             return true;
         }
         return false;
     }

```

unlock方法间接调用AQS的release(1)来完成释放

tryRelease(int)方法进行了特殊的判定，如果成立则会将head传入unparkSuccessor(Node)
方法中并且返回true，否则返回的就是false。

```

public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

```

而他在执行了unparkSuccessor方法中的时候，就已经意味着要真正的释放锁了。
这其实就是独享锁进行获取锁和释放锁的一个过程！有兴趣的可以去源码中把注释翻译一下看看。


### 共享锁

 从我们之前的独享所就能看得出来，独享锁是使用的一个状态来进行锁标记的，共享锁其实也差不多，但是JAVA中有不想定力两个状态，所以区别出现了，
 他们的锁状态时不一样的。
 
 基本的流程是一样的，主要区别在于判断锁获取的条件上，由于是共享锁，也就允许多个线程同时获取，所以同步状态的数量同时的大于1的，如果同步状态为非0，则线程就可以获取锁，只有当同步状态为0时，才说明共享数量的锁已经被全部获取，其余线程只能等待。
 
 最典型的就是ReentrantReadWriteLock里的读锁，它的读锁是可以被共享的，但是它的写锁确每次只能被独占。 

我们来看一下他的获取锁和释放锁的代码体现。

```
    //获取锁指定离不开这个lock方法，
    public void lock() {
                sync.acquireShared(1);
    }
    //acquireShared()首先会通过tryAcquireShared()来尝试获取锁。
    //如果说获取不到那么他就回去执行  doAcquireShared(arg);直到获取到锁才会返回
    //你看方法名do是不是想到了do-while呢？
    public final void acquireShared(int arg) {
            if (tryAcquireShared(arg) < 0)
                doAcquireShared(arg);
    }
    
    // tryAcquireShared()来尝试获取锁。
    protected int tryAcquireShared(int arg) {
            throw new UnsupportedOperationException();
    }
    
    //只有这个方法获取到锁了才会进行返回
    private void doAcquireShared(int arg) {
            final Node node = addWaiter(Node.SHARED);
            boolean failed = true;
            try {
                boolean interrupted = false;
                for (;;) {
                    final Node p = node.predecessor();
                    if (p == head) {
                        int r = tryAcquireShared(arg);
                        if (r >= 0) {
                            setHeadAndPropagate(node, r);
                            p.next = null; // help GC
                            if (interrupted)
                                selfInterrupt();
                            failed = false;
                            return;
                        }
                    }
                    if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
                        interrupted = true;
                }
            } finally {
                if (failed)
                    cancelAcquire(node);
            }
        }
       
     //上面的这些方法全部都是在AbstractQueuedSynchronizer中
     
     //而他通过Sync来调用的acquireShared
     
     //而Sync则是继承的AbstractQueuedSynchronizer
     
     abstract static class Sync extends AbstractQueuedSynchronizer 
     
     而他调用的tryAcquireShared则是在ReentrantReadWriteLock中
     
     protected final int tryAcquireShared(int unused) {
                 
                 Thread current = Thread.currentThread();
                 //获取状态
                 int c = getState();
                 //如果说锁状态不是0 并且获取锁的线程不是current线程 返回-1
                 if (exclusiveCount(c) != 0 &&
                     getExclusiveOwnerThread() != current)
                     return -1;
                 //统计读锁的次数
                 int r = sharedCount(c);
                 //若无需等待，并且共享读锁共享次数小于MAX_COUNT，则会把锁的共享次数加一，
                 //否则他会去执行fullTryAcquireShared
                 if (!readerShouldBlock() &&
                     r < MAX_COUNT &&
                     compareAndSetState(c, c + SHARED_UNIT)) {
                     if (r == 0) {
                         firstReader = current;
                         firstReaderHoldCount = 1;
                     } else if (firstReader == current) {
                         firstReaderHoldCount++;
                     } else {
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
    
        /** fullTryAcquireShared()会根据是否需要阻塞等待
        读取锁的共享计数是否超过限制”等等进行处理。
        如果不需要阻塞等待，并且锁的共享计数没有超过限制，
        则通过CAS尝试获取锁，并返回1。*/
      final int fullTryAcquireShared(Thread current) {
          /*
           * This code is in part redundant with that in
           * tryAcquireShared but is simpler overall by not
           * complicating tryAcquireShared with interactions between
           * retries and lazily reading hold counts.
           */
          HoldCounter rh = null;
          for (;;) {
              int c = getState();
              if (exclusiveCount(c) != 0) {
                  if (getExclusiveOwnerThread() != current)
                      return -1;
                  // else we hold the exclusive lock; blocking here
                  // would cause deadlock.
              } else if (readerShouldBlock()) {
                  // Make sure we're not acquiring read lock reentrantly
                  if (firstReader == current) {
                      // assert firstReaderHoldCount > 0;
                  } else {
                      if (rh == null) {
                          rh = cachedHoldCounter;
                          if (rh == null || rh.tid != getThreadId(current)) {
                              rh = readHolds.get();
                              if (rh.count == 0)
                                  readHolds.remove();
                          }
                      }
                      if (rh.count == 0)
                          return -1;
                  }
              }
              if (sharedCount(c) == MAX_COUNT)
                  throw new Error("Maximum lock count exceeded");
              if (compareAndSetState(c, c + SHARED_UNIT)) {
                  if (sharedCount(c) == 0) {
                      firstReader = current;
                      firstReaderHoldCount = 1;
                  } else if (firstReader == current) {
                      firstReaderHoldCount++;
                  } else {
                      if (rh == null)
                          rh = cachedHoldCounter;
                      if (rh == null || rh.tid != getThreadId(current))
                          rh = readHolds.get();
                      else if (rh.count == 0)
                          readHolds.set(rh);
                      rh.count++;
                      cachedHoldCounter = rh; // cache for release
                  }
                  return 1;
              }
          }
      }
      
```

以上的源码就是共享锁的一个获取锁的过程

接下来肯定是要进行锁的释放了

`unlock()`  

```

    public void unlock() {
            sync.releaseShared(1);
    }
    //和获取锁的过程类似，他首先会通过tryReleaseShared()去尝试释放共享锁。尝试成功，则直接返回；尝试失败，
    //则通过doReleaseShared()去释放共享锁。
     public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
    
    //是尝试释放共享锁第一步。
    
    protected final boolean tryReleaseShared(int unused) {
        Thread current = Thread.currentThread();
        if (firstReader == current) {
            // assert firstReaderHoldCount > 0;
            if (firstReaderHoldCount == 1)
                firstReader = null;
            else
                firstReaderHoldCount--;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                rh = readHolds.get();
            int count = rh.count;
            if (count <= 1) {
                readHolds.remove();
                if (count <= 0)
                    throw unmatchedUnlockException();
            }
            --rh.count;
        }
        for (;;) {
            int c = getState();
            int nextc = c - SHARED_UNIT;
            if (compareAndSetState(c, nextc))
                // Releasing the read lock has no effect on readers,
                // but it may allow waiting writers to proceed if
                // both read and write locks are now free.
                return nextc == 0;
        }
    }
    
    //持续执行释放共享锁
    private void doReleaseShared() {
            /*
             * Ensure that a release propagates, even if there are other
             * in-progress acquires/releases.  This proceeds in the usual
             * way of trying to unparkSuccessor of head if it needs
             * signal. But if it does not, status is set to PROPAGATE to
             * ensure that upon release, propagation continues.
             * Additionally, we must loop in case a new node is added
             * while we are doing this. Also, unlike other uses of
             * unparkSuccessor, we need to know if CAS to reset status
             * fails, if so rechecking.
             */
            for (;;) {
                Node h = head;
                if (h != null && h != tail) {
                    int ws = h.waitStatus;
                    if (ws == Node.SIGNAL) {
                        if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                            continue;            // loop to recheck cases
                        unparkSuccessor(h);
                    }
                    else if (ws == 0 &&
                             !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                        continue;                // loop on failed CAS
                }
                if (h == head)                   // loop if head changed
                    break;
            }
        }

```

以上的代码就是共享锁和非共享锁的源码。需要注意的时候，在这里其实很乱，有些方法是定义在ReentrantReadWriteLock中的，
而有一些方法是定义在AbstractQueuedSynchorizer中的，所以在来回切换看代码的时候尤其要注意，不要出现失误。

###总结

独享锁：同时只能有一个线程获得锁。

共享锁：可以有多个线程同时获得锁。

关于独享锁和共享锁，你明白了吗？
