---
layout: post
title:  云阶月地，关锁千重(一.公平和非公平)
categories: java数据结构
tags: 
    - 懿
---

看到文章的标题是不是很诧异，一个搞技术的为什么要搞这么文艺的话题呢？标题说关锁千重，是不是很形象，我们在开发中的锁不也是多种多样么？
<!--more-->

### Lock

既然之前说了锁千重，那锁到底有多少种，他们的分类又是怎么区分的，为什么这么区分？我来给大家解释一下。

### 为什么加锁？

面试中有很多时候会问到，为什么加锁？加锁是起到什么作用？

而实际上在我们的开发过程中会出现并发的情况，比如说两个人几乎同时点击了某一个按钮，这个时候就可以简单的理解成并发，那么到底谁先谁后？
程序中就很可能出现错误，当资源出现共享的时候，就会开始涉及到并发了，这个时候我们就可能会用到锁了，来锁住某一个资源，等我用过之后，你才能动。
这就是为什么使用锁。

### 锁的分类

 1. 公平锁/非公平锁
 2. 可重入锁
 3. 独享锁/共享锁
 4. 互斥锁/读写锁
 5. 乐观锁/悲观锁
 6. 分段锁
 7. 偏向锁/轻量级锁/重量级锁
 8. 自旋锁
 
 第一次分享，我们就先说这个公平锁和非公平锁。之后会在后序的文章中继续解析！
 
 何为公平？何为非公平？在我们日常生活中的理解不就是对等的就是公平，不对等的就是不公平？
 其实差不多的。
 
 公平锁是指多个线程按照申请锁的顺序来获取锁
 
 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。
 
 在JAVA的代码中什么是公平锁什么又是非公平的锁呢？
 
 一种是使用Java自带的关键字synchronized对相应的类或者方法以及代码块进行加锁，
 
 而另一种是ReentrantLock，前者只能是非公平锁，而后者是默认非公平但可实现公平的一把锁。

![](/assets/images/2019/java/image_yi/05-01/rentlock.jpg)

上面的类图看起来很不舒服，因为关于ReentrantLock这个锁，确实是没有很好的配图，我们可以自己画出来理解一下

我们先看非公平锁，我画图大家理解一下，就像公共厕所，不要嫌弃恶心，但是绝对容易理解

![](/assets/images/2019/java/image_yi/05-01/tu1.jpg)

上面这幅图，加入说图中的管理员是Lock，然后现在A来了，说我要去厕所，这时候管理员一看，厕所没人，那好你进去把，然后A就进去了。

![](/assets/images/2019/java/image_yi/05-01/tu3.jpg)

这个时候WC里面是有A的，正在进行式，这时候B来了，B也想去厕所

![](/assets/images/2019/java/image_yi/05-01/tu2.jpg)

但是这个时候WC里面是有A的，然后管理员Lock看了一下，里面有人，排队把。。。

然后这B就得憋着，去进入队列去排队，然后又来了个C，这时候A还在里面，C只能也去排队，

就是这个样子的

![](/assets/images/2019/java/image_yi/05-01/tu4.jpg)

这时候又过了一小会来了个D，也想去WC，这时候A恰好结束了，

这时候非公平锁就上场了，Lock管理员一看，里面没人，D你进去把，这时候就是这样的。

![](/assets/images/2019/java/image_yi/05-01/tu5.jpg)

然后这时候因为A出来了之后说，我结束了，然后B正准备进去的时候，里面又有了D，这就是出现了非公平锁

而非公平锁的调用就是
A被Lock --> B去wait--> C在B后面wait --> A结束了，unlock -->非公平锁compareAndSetState(0, acquires),里面没人 -->D被Lock
这时候A释放锁lock.release，-->结果晚了一步，没进去，只能继续等着

这就是非公平锁的简单的一种理解，道理其实挺简单的，
而公平锁就不一样了，Lock管理员会告诉新来的D，你前面已经有好几个在排号的了，你想进去，去后边排队。


![](/assets/images/2019/java/image_yi/05-01/tu6.jpg)

公平锁的调用是这样子的。

前面其实都是一样的，但是当D来的时候就不一样了，

D来的时候-->lock知道了-->直接调用hasQueuedPredecessors()告诉D有人在排队，你去后边排队，这样子下来就实实在在的保证了公平了。

接下来我们看看ReentrantLock的源代码实现，然后解释一下

```非公平

public class ReentrantLock implements Lock, java.io.Serializable {
        /*同步器提供所有实现机制 */
        private final Sync sync;
        
       /**
        此锁的同步控制基础。将转换为下面的公平和非公平版本。使用AQS状态表示锁定的保持数。
        */
       abstract static class Sync extends AbstractQueuedSynchronizer {
           private static final long serialVersionUID = -5179523762034025860L;
   
           /**
            执行{@link Lock＃lock}。子类化x的主要原因是允许非公平版本的快速路径。
            */
           abstract void lock();
   
           /**
            * 执行非公平的tryLock。 tryAcquire在子类中实现，但两者都需要tryf方法的非公平尝试。
                注意在这里执行的是非公平锁也就是说在判断方法compareAndSerState的时候，
                新的线程可能抢占已经排队的线程的锁的使用权
            */
           final boolean nonfairTryAcquire(int acquires) {
               final Thread current = Thread.currentThread();
               int c = getState();
               //状态为0，说明当前没有线程占有锁
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
   
           protected final boolean isHeldExclusively() {
               // While we must in general read state before owner,
               // we don't need to do so to check if current thread is owner
               return getExclusiveOwnerThread() == Thread.currentThread();
           }
   
           final ConditionObject newCondition() {
               return new ConditionObject();
           }
   
           // Methods relayed from outer class
   
           final Thread getOwner() {
               return getState() == 0 ? null : getExclusiveOwnerThread();
           }
   
           final int getHoldCount() {
               return isHeldExclusively() ? getState() : 0;
           }
   
           final boolean isLocked() {
               return getState() != 0;
           }
   
           /**
            从流中重构实例（即反序列化它）。
            */
           private void readObject(java.io.ObjectInputStream s)
               throws java.io.IOException, ClassNotFoundException {
               s.defaultReadObject();
               setState(0); // reset to unlocked state
           }
       }
}}
 
```

上面的源码都是说的是非公平锁的，我们在看看源码中对公平锁又是怎么进行定义的！

```公平

    /**
        同步对象以进行公平锁定
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         tryAcquire的公平版本。除非递归呼叫或没有服务员或是第一个，否则不授予访问权限。
            在这里有一个hasQueuedPredecessors的方法，就是这个方法来保证我们不论是新的线程还是已经在进行排队的线程都顺序的去使用锁
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }

```
解释完了，我们来做个总结
总结一：
1. 公平锁：老的线程排队使用锁，新线程仍然排队使用锁。 
2. 非公平锁：老的线程排队使用锁；但是新的线程是极有可能抢占已经在排队的线程的锁。

以后我会更新关于锁的一些文章，希望大家能够指点一下，也能够共同的讨论进步，谢谢！




