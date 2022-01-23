---
layout: post
categories: 并发
title: ThreadLocal 你怎么动不动就内存泄漏？
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
如果说 ThreadLocal 的话，那肯定就会涉及到内存泄漏，为啥嘞

因为 吧啦吧啦 ~
<!-- more -->

## ThreadLocal 解决了什么问题呢？

它是为了解决对象不能被多线程共享访问的问题，通过 threadLocal.set() 方法将对象实例保存在每个线程自己所拥有的 threadLocalMap 中，这样的话每个线程都使用自己的对象实例，彼此不会影响从而达到了隔离的作用，这样就解决了对象在被共享访问时带来的线程安全问题

啥意思呢？打个比方，现在公司所有人都要填写一个表格，但是只有一支笔，这个时候就只能上个人用完了之后，下个人才可以使用，为了保证"笔"这个资源的可用性，只需要保证在接下来每个人的获取顺序就可以了，这就是 lock 的作用，当这支笔被别人用的时候，我就加 lock ，你来了那就进入队列排队等待获取资源（非公平方式那就另外说了），这支笔用完之后就释放 lock ，然后按照顺序给下个人使用

但是完全可以一个人一支笔对不对，这样的话，你填写你的表格，我填写我的表格，咱俩谁都不耽搁谁。这就是 ThreadLocal 在做的事情，因为每个 Thread 都有一个副本，就不存在资源竞争，所以也就不需要加锁，这不就是拿空间去换了时间嘛

在开始之前，咱们先把 Thread， ThreadLocal， ThreadLocalMap 的关系捋一捋：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/01-thread.jpg)

可以看到，在 Thread 中持有一个 ThreadLocalMap ， ThreadLocalMap 又是由 Entry 来组成的，在 Entry 里面有 ThreadLocal 和 value

## ThreadLocal 为啥动不动就内存泄漏呢？

在这里先给个解释，后面咱们再详细分析：

首先是因为 ThreadLocal 是基于 ThreadLocalMap 实现的，其中 ThreadLocalMap 的 Entry 继承了 WeakReference ，而 Entry 对象中的 key 使用了 WeakReference 封装，也就是说， Entry 中的 key 是一个弱引用类型，对于弱引用来说，它只能存活到下次 GC 之前

如果此时一个线程调用了 ThreadLocalMap 的 set 设置变量，当前的 ThreadLocalMap 就会新增一条记录，但由于发生了一次垃圾回收，这样就会造成一个结果: key 值被回收掉了，但是 value 值还在内存中，而且如果线程一直存在的话，那么它的 value 值就会一直存在

这样被垃圾回收掉的 key 就会一直存在一条引用链: Thread -> ThreadLocalMap -> Entry -> Value :

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/02-引用链.jpg)

就是因为这条引用链的存在，就会导致如果 Thread 还在运行，那么 Entry 不会被回收，进而 value 也不会被回收掉，但是 Entry 里面的 key 值已经被回收掉了

这只是一个线程，如果再来一个线程，又来一个线程…多了之后就会造成内存泄漏

知道是怎么造成内存泄漏之后，接下来要做的事情就好说了，不是因为 value 值没有被回收掉所以才会导致内存泄露的嘛

那使用完 key 值之后，将 value 值通过 remove 方法 remove 掉，这样的话内存中就不会有 value 值了，也就防止了内存泄漏嘛

## ThreadLocal 是基于 ThreadLocalMap 实现的？

OK ，上面的内容讲完了，接下来一一来看

首先，你怎么知道 ThreadLocal 是基于 ThreadLocalMap 实现的呢？

从源码知道的~

在源码中能够看到下面这几行代码：

```java
public class ThreadLocal<T> {
    static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
    }
}
```

代码中说的很清楚了，在 ThreadLocal 内部维护着 ThreadLocalMap ，而它的 Entry 则继承自 WeakReference 的 ThreadLocal<?> ，其中 Entry 的 k 为 ThreadLocal ， v 为 Object ，在调用 super(k) 时就将 ThreadLocal 实例包装成了一个 WeakReference

强弱引用这块内容阿粉就直接放一个表格吧：

|  引用类型   |  功能特点   |
| :-----:|:----: |
| 强引用 ( Strong Reference )  |  被强引用关联的对象永远不会被垃圾回收器回收掉 |
| 软引用( Soft Reference )  |  软引用关联的对象，只有当系统将要发生内存溢出时，才会去回收软引用引用的对象 |
| 弱引用 ( Weak Reference ) | 只被弱引用关联的对象，只要发生垃圾收集事件，就会被回收 |
| 虚引用 ( Phantom Reference ) | 被虚引用关联的对象的唯一作用是能在这个对象被回收器回收时收到一个系统通知 |

从表格中应该能够看出来，弱引用的对象只要发生垃圾收集事件，就会被回收

所以弱引用的存活时间也就是下次 GC 之前了

在这里阿粉就有个问题想问问了：为什么 ThreadLocal 采用弱引用，而不是强引用嘞？

在 ThreadLocalMap 上面有些注释，我在这里摘录一部分，或许可以从中窥探一二：

`To help deal with very large and long-lived usages， the hash table entries use WeakReferences for keys`

翻译一下就是:（虽然我英语不是很好

为了解决非常大且长期使用的问题，哈希表使用了弱引用的 key

假设，假设， ThreadLocal 使用的是强引用，会怎样呢？

如果是强引用的话，在表格中也能够看出来，被强引用关联的对象，永远都不会被垃圾回收器回收掉

如果引用的 ThreadLocal 对象被回收了，但是 ThreadLocalMap 还持有对 ThreadLocal 的强引用，如果没有 remove 的话， 在 GC 时进行可达性分析， ThreadLocal 依然可达，这样就不会对 ThreadLocal 进行回收，但是我们期望的是引用的 ThreadLocal 对象被回收，这样不就达不到目的了嘛

使用弱引用的话，虽然会出现内存泄漏的问题，但是在 ThreadLocal 生命周期里面，都有对 key 值为 null 时进行回收的处理操作

所以，使用弱引用的话，可以在 ThreadLocal 生命周期中尽可能保证不出现内存泄漏的问题

啥？在 ThreadLcoal 生命周期里面，都有对 key 值为 null 时进行回收的处理操作？有证据么？

那必须得有证据，毕竟阿粉可是个负责任的博主，不过阿粉考虑到这篇文章内容已经是比较多的了，所以下篇文章阿粉再带你进行源码分析好不好

乖~

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/03-可爱.gif)
