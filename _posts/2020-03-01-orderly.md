---
layout: post
categories: java并发
title: 诡异的并发之有序性
tagline: by 淼淼之森
tags: 
    - 淼淼之森
---

上一节阿粉我和大家一起打到了并发中的恶霸[可见性](https://mp.weixin.qq.com/s/jPIGL2jvxYvo-1WOu23VAA)和[原子性]()，这一节我们继续讨伐三恶之一的有序性。
<!--more-->
## 序、有序性的阐述

有序性为什么要探讨？因为 Java 是面向对象编程的，关注的只是最终结果，很少去研究其具体执行过程？正如上一篇文章在介绍可见性时描述的一样，操作系统为了提升性能，将 Java 语言转换成机器语言的时候，吩咐编译器对语句的执行顺序进行了一定的修改，以促使系统性能达到最优。所以在很多情况下，访问一个程序变量（对象实例字段，类静态字段和数组元素）可能会使用不同的顺序执行，而不是程序语义所指定的顺序执行。

正如大家所熟知那样，Java语言是运行在 Java 自带的 Jvm(Java Virtual Machine) 环境中，在JVM环境中源代码(.class)的执行顺序与程序的执行顺序(runtime)不一致，或者程序执行顺序与编译器执行顺序不一致的情况下，我们就称程序执行过程中发生了**重排序**。

而编译器的这种修改是自以为能保证最终运行结果！因为在单核时代完全没问题；但是随着多核时代的到来，多线程的环境下，这种优化碰上线程切换就大大的增加了事故的出现几率！

也就是说，**有序性** 指的是在代码顺序结构中，我们可以直观的指定代码的执行顺序, 即从上到下按序执行。但编译器和CPU处理器会根据自己的决策，对代码的执行顺序进行重新排序。优化指令的执行顺序，提升程序的性能和执行速度，使语句执行顺序发生改变，出现重排序，但最终结果看起来没什么变化（单核）。

**有序性问题** 指的是在多线程环境下（多核），由于执行语句重排序后，重排序的这一部分没有一起执行完，就切换到了其它线程，导致的结果与预期不符的问题。这就是编译器的编译优化给并发编程带来的程序有序性问题。

用图示就是：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/03-01/1.png)

阿粉小结：编译优化最终导致了有序性问题。


## 一、导致有序性的原因：

如果一个线程写入值到字段 a，然后写入值到字段 b ，而且b的值不依赖于  a 的值，那么，处理器就能够自由的调整它们的执行顺序，而且缓冲区能够在 a 之前刷新b的值到主内存。此时就可能会出现有序性问题。

例子：
```
import java.time.LocalDateTime;

/**
 * @author ：mmzsblog
 * @description：并发中的有序性问题
 * @date ：2020年2月26日 15:22:05
 */
public class OrderlyDemo {

    static int value = 1;
    private static boolean flag = false;

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 199; i++) {
            value = 1;
            flag = false;
            Thread thread1 = new DisplayThread();
            Thread thread2 = new CountThread();
            thread1.start();
            thread2.start();
            System.out.println("=========================================================");
            Thread.sleep(6000);
        }
    }

    static class DisplayThread extends Thread {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " DisplayThread begin, time:" + LocalDateTime.now());
            value = 1024;
            System.out.println(Thread.currentThread().getName() + " change flag, time:" + LocalDateTime.now());
            flag = true;
            System.out.println(Thread.currentThread().getName() + " DisplayThread end, time:" + LocalDateTime.now());
        }
    }

    static class CountThread extends Thread {
        @Override
        public void run() {
            if (flag) {
                System.out.println(Thread.currentThread().getName() + " value的值是：" + value + ", time:" + LocalDateTime.now());
                System.out.println(Thread.currentThread().getName() + " CountThread flag is true,  time:" + LocalDateTime.now());
            } else {
                System.out.println(Thread.currentThread().getName() + " value的值是：" + value + ", time:" + LocalDateTime.now());
                System.out.println(Thread.currentThread().getName() + " CountThread flag is false, time:" + LocalDateTime.now());
            }
        }
    }
}
```
运行结果：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/03-01/2.png)

从打印的可以看出：在 DisplayThread 线程执行的时候肯定是发生了重排序，导致先为 flag 赋值，然后切换到 CountThread 线程，这才出现了打印的 value 值是1，falg 值是 true 的情况，再为 value 赋值；不过出现这种情况的原因就是这两个赋值语句之间没有联系，所以编译器在进行代码编译的时候就可能进行指令重排序。

用图示，则为：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/03-01/3.png)


## 二、如何解决有序性

### 2.1、volatile
volatile 的底层是使用内存屏障来保证有序性的（让一个Cpu缓存中的状态(变量)对其他Cpu缓存可见的一种技术）。

volatile 变量有条规则是指对一个 volatile 变量的写操作， Happens-Before 于后续对这个 volatile 变量的读操作。并且这个规则具有传递性，也就是说：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/03-01/4.png)

此时，我们定义变量 flag 时使用 volatile 关键字修饰，如：
```
    private static volatile boolean flag = false;
```
此时，变量的含义是这样子的：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/03-01/5.png)

也就是说，只要读取到 `flag=true;` 就能读取到 `value=1024`；否则就是读取到 `flag=false;` 和 `value=1` 的还没被修改过的初始状态；

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/03-01/6.png)

但也有可能会出现线程切换带来的原子性问题，就是读取到 `flag=false;` 而 `value=1024` 的情况；看过上一篇讲述[原子性]()的文章的小伙伴，可能就立马明白了，这是线程切换导致的。

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/03-01/7.png)

### 2.2、加锁

此处我们直接采用Java语言内置的关键字 synchronized，为可能会重排序的部分加锁，让其在宏观上或者说执行结果上看起来没有发生重排序。

代码修改也很简单，只需用 synchronized 关键字修饰run方法即可,代码如下:
```
    public synchronized void run() {
        value = 1024;
        flag = true;
    }
```

同理，既然是加锁，当然也可以使用 Lock 加锁，但 Lock 必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。这点在使用的时候一定要注意!使用该种方式加锁也很简单，代码如下：
```
    readWriteLock.writeLock().lock();
    try {
        value = 1024;
        flag = true;
    } finally {
        readWriteLock.writeLock().unlock();
    }
```

好了，以上内容就是我对并法中的有序性的一点理解与总结了，通过这三篇文章我们也就大致掌握了并发中常见的可见性、有序性、原子性问题以及它们常见的解决方案。



## 最后
最后，阿粉简单总结下三篇文章文章中使用的解决方案之间的区别：

特性	| Atomic变量 | volatile关键字 | Lock接口	| synchronized关键字
--|--|--|--|--                                     
原子性	| 可以保障   | 无法保障	    | 可以保障	| 可以保障	       		
可见性	| 可以保障   | 可以保障	    | 可以保障	| 可以保障	       		
有序性	| 无法保障   | 一定程度保障	| 可以保障	| 可以保障	       		






参考文章：
- 1、极客时间的Java并发编程实战
- 2、https://juejin.im/post/5d52abd1e51d4561e6237124
- 3、https://juejin.im/post/5d89fd1bf265da03e71b3605
- 4、https://www.cnblogs.com/54chensongxia/p/12120117.html
- 5、http://ifeve.com/jmm-faq-reordering/
