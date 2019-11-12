---
layout: post
title:  读写锁，你难道不需要了解一下吗？
tagline: by 懿
categories: java数据结构
tag: 
    - 懿
---

之前在的文章中已经写了公平锁、非公平锁，独享锁、共享锁，那么接下来我们就得介绍互斥锁和读写锁了。那我们我就来了解一波把！
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
 
 
 ## 互斥锁
 
 首先我们先说什么是互斥？
 
 互斥：事件A和B的交集为空，A与B就是互斥事件，也叫互不相容事件。这是百度百科中对互斥的的说法，比较官方，而其实所谓互斥，是指散布在不同进程之间的若干程序片断，
 当某个进程运行其中一个程序片段时，其它进程就不能运行它们之中的任一程序片段，只能等到该进程运行完这个程序片段后才可以运行。
 
 这样就是说互斥的两个线程之间是不可以同时运行，他们相互之间会排斥，必须是A线程运行完毕之后，B线程才能进行。
 
 在我们通常使用的线程过程中遇到的最多的就是同步，实现同步的方法，我们就可以使用synchronized，而 synchronized 就是互斥锁。
 但是这是很早之前我们使用过的了，其实还有一个是显式的，使用Lock 对象
 
 ####synchronized
 
 我们先看看synchronized。
 
 synchronized机制提供了对与每个对象相关的隐式监视器锁的访问, 并强制所有锁获取和释放均要出现在一个块结构中, 
 当获取了多个锁时, 它们必须以相反的顺序释放. synchronized机制对锁的释放是隐式的, 只要线程运行的代码超出了synchronized语句块范围, 锁就会被释放. 
 
 而Lock机制必须显式的调用Lock对象的unlock()方法才能释放锁, 这为获取锁和释放锁不出现在同一个块结构中, 以及以更自由的顺序释放锁提供了可能. 
 
 我们看一下下面的几个场景：
 
 (1) 普通方法前面加synchronized
 
 synchronized public void test()｛....｝
 
 这个操作就等价于在方法体前后包装了一个synchronized(this),或者说是给当前的类所在的对象加上了锁的标记，
 
 而与它互斥的情况就会有三种，(也就是相互之间是的串的)
 
- 在该类的所有的静态方法中发生synchronized(this);

- 在该类的所有的静态方法前面加上synchronized关键字；

- 在其他类中得到该对象的引用，并对该对象进行synchronized操作；
 
 
 synchronized public static void test(){....}
 
 这个synchronized操作就等价于锁住了当前类的class对象，比如说这个类是A，那么相当于执行了synchronized(A.class)操作，
 而与它互斥的场景就十分的明显了。
 
- 代码中任何一个地方发生了synchronized(A.class);

- 在该类的所有的静态的方法前面增加了synchronized关键字；

我们需要注意的是，锁住了类，并不是说我们锁住了类所在的对象，类本身也是一种对象呀。
它与类的实例是完全不同的两个对象，在加锁的时候不是相互以来的，换句话说，我们对类进行加锁并不与前面的一个案例锁描述的加锁互斥。

锁住了“子类”或“子类的对象”，与锁住“父类”或“父类的对象”是完全不想管的，他们彼此独立！

你看我们常说的synchronized的代码块加锁

```

synchronized （object）｛
    //代码
｝

```

这一段代码其实锁住的并不是代码块，而是锁住的object对象，因此在其他的代码中发生synchronized(object)时就会发生互斥。

### 读写锁

有很多时候会有人有疑问？读写锁是为了什么而存在的？这个如果你不看源码的话，你是不知道为什么的，如果你看了，那么就会很清晰的理解为什么了。

读写锁，是分为读锁和写锁的，那么他是为什么存在，其实最好理解的就是为了解决性能问题！

性能问题一直都是我们开发中最担心的一个问题，而JAVA提供了读写锁，在读的时候使用读锁，在写的时候使用写锁，如果在没有写锁的情况下，
读是无阻塞的，在一定程度上是它能提高程序的执行效率，读写锁之间，多个读锁不互斥，读锁和写锁确实互斥，这是JVM自己来控制的，而
JVM帮我们解决了，我们只需要去加锁即可。

我们来看看读写锁中经典的源码实例ReentrantReadWriteLock,
其实之前我已经不经意的提到过了，话不多来，来low一眼。

```

    /**
    获取读锁，如果写锁不是由其他线程持有，则获取并立即返回； 
    如果写锁被其他线程持有，阻塞，直到读锁被获得。 
    */
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
              //首次获取读锁失败后，重试获取
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

上面的源码是ReentrantReadWriteLock,中对读锁的解释，也是获取锁的过程，解锁过程我就不多说了，又兴趣的可以去之前的文章中仔细的去看。

而写锁相对于读锁来说，可能就没有那么复杂了

```

    public void lock() {
        sync.acquire(1);
    }
    //此方法在AQS中
    public final void acquire(int arg) {
            if (!tryAcquire(arg) &&
            // 如果 tryAcquire 失败，那么进入到阻塞队列等待
                acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                selfInterrupt();
     }
     //方法在 Sync中
     protected final boolean tryAcquire(int acquires) {
             /*
              * Walkthrough:
              * 1. If read count nonzero or write count nonzero
              *    and owner is a different thread, fail.
              * 2. If count would saturate, fail. (This can only
              *    happen if count is already nonzero.)
              * 3. Otherwise, this thread is eligible for lock if
              *    it is either a reentrant acquire or
              *    queue policy allows it. If so, update state
              *    and set owner.
              */
             Thread current = Thread.currentThread();
             int c = getState();
             int w = exclusiveCount(c);
             if (c != 0) {
                 //   c != 0 && w == 0: 写锁可用，但是有线程持有读锁(也可能是自己持有)
                 //   c != 0 && w !=0 && current != getExclusiveOwnerThread(): 其他线程持有写锁
                 //   也就是说，只要有读锁或写锁被占用，这次就不能获取到写锁
                 
                 if (w == 0 || current != getExclusiveOwnerThread())
                     return false;
                 if (w + exclusiveCount(acquires) > MAX_COUNT)
                     throw new Error("Maximum lock count exceeded");
                 // Reentrant acquire
                 setState(c + acquires);
                 return true;
             }
             // 如果写锁获取不需要 block，那么进行 CAS，成功就代表获取到了写锁
             if (writerShouldBlock() ||
                 !compareAndSetState(c, c + acquires))
                 return false;
             setExclusiveOwnerThread(current);
             return true;
         }

```

上面的代码是写锁加锁的过程了，其实相对于读锁来说稍微简单一点点。

那么我们再来看一下写锁是怎么释放的。

```

    public void unlock() {
        sync.release(1);
    }
    //方法在AQS中
     public final boolean release(int arg) {
            if (tryRelease(arg)) {
                Node h = head;
                if (h != null && h.waitStatus != 0)
                    unparkSuccessor(h);
                return true;
            }
            return false;
        }
        
      protected final boolean tryRelease(int releases) {
          if (!isHeldExclusively())
              throw new IllegalMonitorStateException();
          int nextc = getState() - releases;
          boolean free = exclusiveCount(nextc) == 0;
          if (free)
              setExclusiveOwnerThread(null);
          setState(nextc);
          // 如果 exclusiveCount(nextc) == 0，所有的写锁就都释放了，
          // 那么返回 true，这样会进行唤醒后继节点的操作。
          return free;
      }

```
 
 看完之后我们就能发现，他确实相对于读锁来说比较简单。
 
以上就是互斥锁和读写锁的所有解析过程，

在看文章的过程中，首先要先去看一下AQS到底是什么，不然很多东西会看不明白！

##总结

互斥锁是一种简单的加锁的方法来控制对共享资源的访问，互斥锁只有两种状态,即上锁( lock )和解锁( unlock )。

互斥锁的特点：

- 原子性：把一个互斥量锁定为一个原子操作，保证了如果一个线程锁定了一个互斥量，没有其他线程在同一时间可以成功锁定这个互斥量；

- 唯一性：如果一个线程锁定了一个互斥量，在它解除锁定之前，没有其他线程可以锁定这个互斥量；

读写锁是为了让程序的性能更加优越而存在的，

读写锁特点：

- 多个读者可以同时进行读

- 写者必须互斥（只允许一个写者写，也不能读者、写者同时进行）

- 写者优先于读者（一旦有写者，则后续读者必须等待，唤醒时优先考虑写者）
