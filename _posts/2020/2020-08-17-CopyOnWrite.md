---
layout: post
categories: 多线程
title: CopyOnWrite 了解吗？
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
# 概念
<!-- more -->

CopyOnWrite 只是看字面意思就能看出来，就是在写入时复制，说得轻巧，写入时复制，具体是怎么实现的呢？

先来说说思想，具体怎么实现等下分析

CopyOnWrite 的思想就是：当向一个容器中添加元素的时候，不是直接在当前这个容器里面添加的，而是复制出来一个新的容器，在新的容器里面添加元素，添加完毕之后再将原容器的引用指向新的容器，这样就实现了写入时复制

你还记得在提到数据库的时候，一般都会说主从复制，读写分离吗？ CopyOnWrite 的设计思想是不是和经常说的主从复制，读写分离如出一撤？

# 优缺点

了解概念之后，对它的优缺点应该就比较好理解了

优点就是，读和写可以并行执行，因为读的是原来的容器，写的是新的容器，它们之间互不影响，所以读和写是可以并行执行的，在某些高并发场景下，可以提高程序的响应时间

但是呢，你也看到了， CopyOnWrite 是在写入的时候，复制了一个新的容器出来，所以要考虑它的内存开销问题，又回到了在学算法时一直强调的一个思想:拿空间换时间

还有一点就是，它只保证数据的最终一致性。因为在读的时候，读取的内容是原容器里面的内容，新添加的内容是读取不到的

基于它的优缺点应该就可以得出一个结论： CopyOnWrite 适用于写操作非常少的场景，而且还能够容忍读写的暂时不一致
如果你的应用场景不适合，那还是考虑使用别的方法来实现吧

还有一点需要注意的是：在写入时，它会复制一个新的容器，所以如果有写入需求的话，最好可以批量写入，因为每次写入的时候，容器都会进行复制，如果能够减少写入的次数，就可以减少容器的复制次数

在 JUC 包下，实现 CopyOnWrite 思想的就是 CopyOnWriteArrayList & CopyOnWriteArraySet 这两个方法，本篇文章侧重于讲清楚 CopyOnWriteArrayList

# CopyOnWriteArrayList

在 CopyOnWriteArrayList 中，需要注意的是 add 方法：

```java
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        // 在写入的时候,需要加锁,如果不加锁的话,在多线程场景下可能会被 copy 出 n 个副本出来
        // 加锁之后,就能保证在进行写时,只有一个线程在操作
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            // 复制原来的数组
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            // 将要添加的元素添加到新数组中
            newElements[len] = e;
            // 将对原数组的引用指向新的数组
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

在写的时候需要加锁，但是在读取的时候不需要添加

因为读取的是原数组的元素，对新数组没有什么影响，加了锁反而会增加性能开销

```java
public E get(int index) {
	return get(getArray(), index);
}
```

# 举个例子：

CopyOnWrite 在 JUC 包下，那么它就保证了线程安全

咱们来做个小 demo 验证一下：

```java
@Slf4j
public class ArrayListExample {

    // 请求总数
    public static int clientTotal = 5000;

    // 同时并发执行的线程数
    public static int threadTotal = 200;

    private static List<Integer> list = new ArrayList<>();

    public static void  main(String[] args) throws Exception{
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal; i++) {
            final int count = i;
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    update(count);
                    semaphore.release();
                } catch (Exception e) {
                    log.error("exception",e);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        log.info("size:{}",list.size());
    }
    private static void update(int i){
        list.add(i);
    }
}
```

上面是客户端请求 5000 次，有 200 个线程在同时请求，我使用的是 ArrayList 实现，咱们看下打印结果：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/19-ArrayListDemo.jpg)

如果是线程安全的话，那么最后的结果应该是 5000 才对，多运行几次你会发现，每次程序的执行结果都是不一样的

如果是 CopyOnWriteArrayList 呢？

```java
@Slf4j
public class CopyOnWriteArrayListExample {

    // 请求总数
    public static int clientTotal = 5000;

    // 同时并发执行的线程数
    public static int threadTotal = 200;

    private static List<Integer> list = new CopyOnWriteArrayList<>();

    public static void  main(String[] args) throws Exception{
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal; i++) {
            final int count = i;
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    update(count);
                    semaphore.release();
                } catch (Exception e) {
                    log.error("excepiton",e);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        log.info("size:{}",list.size());
    }
    private static void update(int i){
        list.add(i);
    }
}
```

多运行几次，结果都是一样的：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/20-CopyOnWriteArrayListDemo.jpg)

由此可见， CopyOnWriteArrayList 是线程安全的

以上，感谢阅读~