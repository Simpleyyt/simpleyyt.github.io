---
layout: post  
title: 面试题：Java中如何停止线程？
tagline: by 炭烧生蚝
categories: java线程
tags: 
    - 炭烧生蚝

---

如何停止线程是Java并发面试中的常见问题，本篇文章将从答题思路到答题细节给出一些参考。
<!--more-->

答题思路：
1. 停止线程的正确方式是使用中断
2. 想停止线程需要停止方，被停止方，被停止方的子方法相互配合
3. 扩展到常见的错误停止线程方法:已被废弃的stop/suspend，无法唤醒阻塞线程的volatile

# 1. 正确方式是中断

其实从逻辑上也很好理解的，一个线程正在运行，如何让他停止？

A. 从外部直接调用该线程的stop方法，直接把线程停下来。

B. 从外部通过中断通知线程停止，然后切换到被停止的线程，该线程执行一系列逻辑后自己停止。

很明显B方法要比A方法好很多，A方法太暴力了，你根本不知道被停止的线程在执行什么任务就直接把他停止了，程序容易出问题；而B方法把线程停止交给线程本身去做，被停止的线程可以在自己的代码中进行一些现场保护或者打印错误日志等方法再停止，更加合理，程序也更具健壮性。

下面要讲的是线程如何能够响应中断，第一个方法是通过循环不断判断自身是否产生了中断：

```java
public class Demo1 implements Runnable{

    @Override
    public void run() {
        int num = 0;
        while(num <= Integer.MAX_VALUE / 2 && !Thread.currentThread().isInterrupted()){
            if(num % 10000 == 0){
                System.out.println(num);
            }
            num++;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Demo1());
        thread.start();
        thread.sleep(1000);
        thread.interrupt();
    }
}
```

在上面的代码中，我们在循环条件中不断判断线程本身是否产生了中断，如果产生了中断就不再打印

还有一个方法是通过java内定的机制响应中断：当线程调用sleep()，wait()方法后进入阻塞后，如果线程在阻塞的过程中被中断了，那么线程会捕获或抛出一个中断异常，我们可以根据这个中断异常去控制线程的停止。具体代码如下：

```java
public class Demo3 implements Runnable {
    @Override
    public void run() {
        int num = 0;
        try {
            while(num < Integer.MAX_VALUE / 2){
                if(num % 100 == 0){
                    System.out.println(num);
                }
                num++;
                Thread.sleep(10);
            }
        } catch (InterruptedException e) {//捕获中断异常，在本代码中，出现中断异常后将退出循环
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Demo3());
        thread.start();
        Thread.sleep(5000);
        thread.interrupt();
    }
}
```

# 2. 各方配合才能完美停止

在上面的两段代码中已经可以看到，想通过中断停止线程是个需要多方配合。上面已经演示了中断方和被中断方的配合，下面考虑更多的情况：假如要被停止的线程正在执行某个子方法，这个时候该如何处理中断？

有两个办法：第一个是把中断传递给父方法，第二个是重新设置当前线程为中断。

先说第一个例子：在子方法中把中断异常上抛给父方法，然后在父方法中处理中断：
```java
public class Demo4 implements Runnable{

    @Override
    public void run() {
        try{//在父方法中捕获中断异常
            while(true){
                System.out.println("go");
                throwInterrupt();
            }
        }catch (InterruptedException e) {
            e.printStackTrace();
            System.out.println("检测到中断，保存错误日志");
        }
    }

    private void throwInterrupt() throws InterruptedException {//把中断上传给父方法
        Thread.sleep(2000);
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Demo4());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}
```

第二个例子：在子方法中捕获中断异常，但是捕获以后当前线程的中断控制位将被清除，父方法执行时将无法感知中断。所以此时在子方法中重新设置中断，这样父方法就可以通过对中断控制位的判断来处理中断：

```java
public class Demo5 implements Runnable{

    @Override
    public void run() {
        while(true && !Thread.currentThread().isInterrupted()){//每次循环判断中断控制位
            System.out.println("go");
            throwInterrupt();
        }
        System.out.println("检测到了中断，循环打印退出");
    }

    private void throwInterrupt(){
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();//重新设置中断
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Demo5());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}
```

讲到这里，正确的停止线程方法已经讲的差不多了，下面我们看一下常见的错误停止线程的例子：

# 3. 常见错误停止线程例子：

这里介绍两种常见的错误，先说比较好理解的一种，也就是开头所说的，在外部直接把运行中的线程停止掉。这种暴力的方法很有可能造成脏数据。

看下面的例子：

```java
public class Demo6 implements Runnable{
    /**
     * 模拟指挥军队，以一个连队为单位领取武器，一共有5个连队，一个连队10个人
     */
    @Override
    public void run() {
        for(int i = 0; i < 5; i++){
            System.out.println("第" + (i + 1) + "个连队开始领取武器");
            for(int j = 0; j < 10; j++){
                System.out.println("第" + (j + 1) + "个士兵领取武器");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("第" + (i + 1) + "个连队领取武器完毕");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Demo6());
        thread.start();
        Thread.sleep(2500);
        thread.stop();
    }
}
```

在上面的例子中，我们模拟军队发放武器，规定一个连为一个单位，每个连有10个人。当我们直接从外部通过stop方法停止武器发放后。很有可能某个连队正处于发放武器的过程中，导致部分士兵没有领到武器。

这就好比在生产环境中，银行以10笔转账为一个单位进行转账，如果线程在转账的中途被突然停止，那么很可能会造成脏数据。

另外一个“常见”错误可能知名度不是太高，就是：通过volatile关键字停止线程。具体来说就是通过volatile关键字定义一个变量，通过判断变量来停止线程。这个方法表面上是没问题的，我们先看这个表面的例子：

```java
public class Demo7 implements Runnable {

    private static volatile boolean canceled = false;

    @Override
    public void run() {
        int num = 0;
        while(num <= Integer.MAX_VALUE / 2 && !canceled){
            if(num % 100 == 0){
                System.out.println(num + "是100的倍数");
            }
            num++;
        }
        System.out.println("退出");
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Demo7());
        thread.start();
        Thread.sleep(1000);
        canceled = true;
    }
}
```

但是这个方法有一个潜在的大漏洞，就是若线程进入了阻塞状态，我们将不能通过修改volatile变量来停止线程，看下面的生产者消费者例子：

```java
/**
 * 通过生产者消费者模式演示volatile的局限性，volatile不能唤醒已经阻塞的线程
 * 生产者生产速度很快，消费者消费速度很慢，通过阻塞队列存储商品
 */
public class Demo8 {
    public static void main(String[] args) throws InterruptedException {
        ArrayBlockingQueue storage = new ArrayBlockingQueue(10);

        Producer producer = new Producer(storage);
        Thread producerThread = new Thread(producer);
        producerThread.start();
        Thread.sleep(1000);//1s足够让生产者把阻塞队列塞满

        Consumer consumer = new Consumer(storage);
        while(consumer.needMoreNums()){
            System.out.println(storage.take() + "被消费");
            Thread.sleep(100);//让消费者消费慢一点，给生产者生产的时间
        }

        System.out.println("消费者消费完毕");
        producer.canceled = true;//让生产者停止生产（实际情况是不行的，因为此时生产者处于阻塞状态，volatile不能唤醒阻塞状态的线程）

    }
}

class Producer implements Runnable{

    public volatile boolean canceled = false;

    private BlockingQueue storage;

    public Producer(BlockingQueue storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        int num = 0;
        try{
            while(num < Integer.MAX_VALUE / 2 && !canceled){
                if(num % 100 == 0){
                    this.storage.put(num);
                    System.out.println(num + "是100的倍数，已经被放入仓库");
                }
                num++;
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            System.out.println("生产者停止生产");
        }
    }
}

class Consumer{
    private BlockingQueue storage;

    public Consumer(BlockingQueue storage) {
        this.storage = storage;
    }

    public boolean needMoreNums(){
        return Math.random() < 0.95 ? true : false;
    }
}
```

上面的例子运行后会发现生产线程一直不能停止，因为他处于阻塞状态，当消费者线程退出后，没有任何东西能唤醒生产者线程。

这种错误用中断就很好解决：

```java
/**
 * 通过生产者消费者模式演示volatile的局限性，volatile不能唤醒已经阻塞的线程
 * 生产者生产速度很快，消费者消费速度很慢，通过阻塞队列存储商品
 */
public class Demo8 {
    public static void main(String[] args) throws InterruptedException {
        ArrayBlockingQueue storage = new ArrayBlockingQueue(10);

        Producer producer = new Producer(storage);
        Thread producerThread = new Thread(producer);
        producerThread.start();
        Thread.sleep(1000);//1s足够让生产者把阻塞队列塞满

        Consumer consumer = new Consumer(storage);
        while(consumer.needMoreNums()){
            System.out.println(storage.take() + "被消费");
            Thread.sleep(100);//让消费者消费慢一点，给生产者生产的时间
        }

        System.out.println("消费者消费完毕");
        producerThread.interrupt();
    }
}

class Producer implements Runnable{

    private BlockingQueue storage;

    public Producer(BlockingQueue storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        int num = 0;
        try{
            while(num < Integer.MAX_VALUE / 2 && !Thread.currentThread().isInterrupted()){
                if(num % 100 == 0){
                    this.storage.put(num);
                    System.out.println(num + "是100的倍数，已经被放入仓库");
                }
                num++;
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            System.out.println("生产者停止生产");
        }
    }
}

class Consumer{
    private BlockingQueue storage;

    public Consumer(BlockingQueue storage) {
        this.storage = storage;
    }

    public boolean needMoreNums(){
        return Math.random() < 0.95 ? true : false;
    }
}
```

# 4. 扩展

可能你还会问：如何处理不可中断的阻塞？

只能说很遗憾没有一个通用的解决办法，我们需要针对特定的锁或io给出特定的解决方案。对于这些特殊的例子，api一般会给出可以响应中断的操作方法，我们要选用那些特定的方法，没有万能药。
