---
layout: post
title:  Java：并发不易，先学会用
tagline: by 沉默王二
categories: java
tag: 
    - java
---

[我](https://mp.weixin.qq.com/s/feoOINGSyivBO8Z1gaQVOA)从事Java编程已经11年了，绝对是个老兵；但对于Java并发编程，[我](https://mp.weixin.qq.com/s/feoOINGSyivBO8Z1gaQVOA)只能算是个新兵蛋子。我说这话估计要遭到某些高手的冷嘲热讽，但我并不感到害怕。

因为我知道，每年都会有很多很多的新人要加入Java编程的大军，他们对“并发”编程中遇到的问题也会有感到无助的时候。而我，非常乐意与他们一道，对使用Java线程进行并发程序开发的基础知识进行新一轮的学习。

### 01、我们为什么要学习并发？

我的脑袋没有被如来佛祖开过光，所以喜欢一件事接着一件事的想，做不到“一脑两用”。但有些大佬就不一样，比如说诸葛亮，就能够一边想着琴谱一边谈着弹着琴，还能夹带着盘算出司马懿退兵后的打算。

诸葛大佬就有着超强的“并发”能力啊。换做是我，面对司马懿的千万大军，不仅弹不了琴，弄不好还被吓得屁滚尿流。

每个人都只有一个脑子，就像电脑只有一个CPU一样。但一个脑子并不意味着不能“一脑两用”，关键就在于脑子有没有“并发”的能力。

脑子要是有了并发能力，那真的是厉害到飞起啊，想想司马懿被气定神闲的诸葛大佬吓跑的样子就知道了。

对于程序来说，如果具有并发的能力，效率就能够大幅度地提升。你一定注册过不少网站，收到过不少验证码，如果网站的服务器端在发送验证码的时候，没有专门起一个线程来处理（并发），假如网络不好发生阻塞的话，那服务器端岂不是要从天亮等到天黑才知道你有没有收到验证码？如果就你一个用户也就算了，但假如有一百个用户呢？这一百个用户难道也要在那傻傻地等着，那真要等到花都谢了。

可想而知，并发编程是多么的重要！况且，懂不懂Java虚拟机和会不会并发编程，几乎是判定一个Java开发人员是不是高手的不三法则。所以**要想挣得多，还得会并发啊**！

### 02、并发第一步，创建一个线程

通常，启动一个程序，就相当于起了一个进程。每个电脑都会运行很多程序，所以你会在进程管理器中看到很多进程。你会说，这不废话吗？

不不不，在我刚学习编程的很长一段时间内，我都想当然地以为这些进程就是线程；但后来我知道不是那么回事儿。一个进程里，可能会有很多线程在运行，也可能只有一个。

main函数其实就是一个主线程。我们可以在这个主线程当中创建很多其他的线程。来看下面这段代码。

```java
public class Wanger {
	public static void main(String[] args) {
		for (int i = 0; i < 10; i++) {
			Thread t = new Thread(new Runnable() {
				
				@Override
				public void run() {
					System.out.println("我叫" + Thread.currentThread().getName() + "，我超喜欢沉默王二的写作风格");
				}
			});
			t.start();
		}
	}
}
```

创建线程最常用的方式就是声明一个实现了`Runnable`接口的匿名内部类；然后将它作为创建`Thread`对象的参数；再然后调用`Thread`对象的`start()`方法进行启动。运行的结果如下。

```java
我叫Thread-1，我超喜欢沉默王二的写作风格
我叫Thread-3，我超喜欢沉默王二的写作风格
我叫Thread-2，我超喜欢沉默王二的写作风格
我叫Thread-0，我超喜欢沉默王二的写作风格
我叫Thread-5，我超喜欢沉默王二的写作风格
我叫Thread-4，我超喜欢沉默王二的写作风格
我叫Thread-6，我超喜欢沉默王二的写作风格
我叫Thread-7，我超喜欢沉默王二的写作风格
我叫Thread-8，我超喜欢沉默王二的写作风格
我叫Thread-9，我超喜欢沉默王二的写作风格
```

从运行的结果中可以看得出来，线程的执行顺序不是从0到9的，而是有一定的随机性。这是因为Java的并发是抢占式的，**线程0虽然创建得最早，但它的“争宠”能力却一般，上位得比较艰辛**。

### 03、并发第二步，创建线程池

`java.util.concurrent.Executors`类提供了一系列工厂方法用于创建线程池，可把多个线程放在一起进行更高效地管理。示例如下。

```java
public class Wanger {
	public static void main(String[] args) {
		ExecutorService executorService = Executors.newCachedThreadPool();

		for (int i = 0; i < 10; i++) {
			Runnable r = new Runnable() {

				@Override
				public void run() {
					System.out.println("我叫" + Thread.currentThread().getName() + "，我超喜欢沉默王二的写作风格");
				}
			};
			executorService.execute(r);
		}
		executorService.shutdown();
	}
}
```

运行的结果如下。

```java
我叫pool-1-thread-2，我超喜欢沉默王二的写作风格
我叫pool-1-thread-4，我超喜欢沉默王二的写作风格
我叫pool-1-thread-5，我超喜欢沉默王二的写作风格
我叫pool-1-thread-3，我超喜欢沉默王二的写作风格
我叫pool-1-thread-4，我超喜欢沉默王二的写作风格
我叫pool-1-thread-1，我超喜欢沉默王二的写作风格
我叫pool-1-thread-7，我超喜欢沉默王二的写作风格
我叫pool-1-thread-6，我超喜欢沉默王二的写作风格
我叫pool-1-thread-5，我超喜欢沉默王二的写作风格
我叫pool-1-thread-6，我超喜欢沉默王二的写作风格
```

`Executors`的`newCachedThreadPool()`方法用于创建一个可缓存的线程池，调用该线程池的方法`execute()`可以重用以前的线程，只要该线程可用；比如说，`pool-1-thread-4`、`pool-1-thread-5`和`pool-1-thread-6`就得到了重用的机会。我能想到的最佳形象代言人就是女皇武则天。

如果没有可用的线程，就会创建一个新线程并添加到池中。当然了，那些60秒内还没有被使用的线程也会从缓存中移除。

另外，`Executors`的`newFiexedThreadPool(int num)`方法用于创建固定数目线程的线程池；`newSingleThreadExecutor()`方法用于创建单线程化的线程池（你能想到它应该使用的场合吗？）。

但是，故事要转折了。阿里巴巴的Java开发手册（可在「沉默王二」公众号的后台回复关键字「Java」获取）中明确地指出，**不允许**使用Executors来创建线程池。

![](https://upload-images.jianshu.io/upload_images/1179389-155c39e5f2193f80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不能使用`Executors`创建线程池，那么该怎么创建线程池呢？

直接调用`ThreadPoolExecutor`的构造函数来创建线程池呗。其实`Executors`就是这么做的，只不过没有对`BlockQueue`指定容量。我们需要做的就是在创建的时候指定容量。代码示例如下。

```java
ExecutorService executor = new ThreadPoolExecutor(10, 10,
        60L, TimeUnit.SECONDS,
        new ArrayBlockingQueue(10));
```

### 04、并发第三步，解决共享资源竞争的问题

有一次，我陪家人在商场里面逛街，出电梯的时候有一个傻叉非要抢着进电梯。女儿的小推车就压到了那傻叉的脚上，他竟然不依不饶地指着我的鼻子叫嚣。我直接一拳就打在他的鼻子上，随后我们就纠缠在了一起。

这件事情说明了什么问题呢？第一，遇到不讲文明不知道“先出后进”（LIFO）规则的傻叉真的很麻烦；第二，竞争共享资源的时候，弄不好要拳脚相向。

在Java中，解决共享资源竞争问题的首个解决方案就是使用关键字`synchronized`。当线程执行被`synchronized`保护的代码片段的时候，会对这段代码进行上锁，其他调用这段代码的线程会被阻塞，直到锁被释放。

下面这段代码使用`ThreadPoolExecutor`创建了一个线程池，池里面的每个线程会对共享资源count进行+1操作。现在，闭上眼想一想，当1000个线程执行结束后，count的值会是多少呢？

```java
public class Wanger {
	public static int count = 0;
	
	public static int getCount() {
		return count;
	}
	
	public static void addCount() {
		 count++;
	}
	
	public static void main(String[] args) {
		ExecutorService executorService = new ThreadPoolExecutor(10, 1000,
		        60L, TimeUnit.SECONDS,
		        new ArrayBlockingQueue<Runnable>(10));


		for (int i = 0; i < 1000; i++) {
			Runnable r = new Runnable() {

				@Override
				public void run() {
					Wanger.addCount();
				}
			};
			executorService.execute(r);
		}
		executorService.shutdown();
		System.out.println(Wanger.count);
	}
}
```

事实上，共享资源count的值很有可能是996、998，但很少会是1000。为什么呢？

因为一个线程正在写这个变量的时候，另外一个线程可能正在读这个变量，或者正在写这个变量。这个变量就变成了一个“不确定状态”的数据。**这个变量必须被保护起来**。

通常的做法就是在改变这个变量的`addCount()`方法上加上`synchronized`关键字——保证线程在访问这个变量的时候有序地进行排队。

示例如下：

```java
public synchronized static void addCount() {
	 count++;
}
```

还有另外的一种常用方法——读写锁。分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，由Java虚拟机控制。如果代码允许很多线程同时读，但不能同时写，就上读锁；如果代码不允许同时读，并且只能有一个线程在写，就上写锁。

读写锁的接口是`ReadWriteLock`，具体实现类是 `ReentrantReadWriteLock`。`synchronized`属于互斥锁，任何时候只允许一个线程的读写操作，其他线程必须等待；而`ReadWriteLock`允许多个线程获得读锁，但只允许一个线程获得写锁，效率相对较高一些。

我们先使用枚举创建一个读写锁的单例。代码如下：

```java
public enum Locker {

	INSTANCE;

	private static final ReadWriteLock lock = new ReentrantReadWriteLock();

	public Lock writeLock() {
		return lock.writeLock();
	}

}
```

再在`addCount()`方法中对```count++;```上锁。示例如下。

```java
public static void addCount() {
	// 上锁
	Lock writeLock = Locker.INSTANCE.writeLock();
	writeLock.lock();
	count++;
	// 释放锁
	writeLock.unlock();
}
```

使用读写锁的时候，切记最后要释放锁。

### 05、最后

并发编程难学吗？说实话，真的不太容易。来看一下王宝令老师总结的思维导图就能知道。

![](https://upload-images.jianshu.io/upload_images/1179389-ac3c2c57bafd5d10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但你也知道，“**冰冻三尺非一日之寒**”，学习是一件循序渐进的事情。只要你学会了怎么创建一个线程，学会了怎么创建线程池，学会了怎么解决共享资源竞争的问题，你已经在并发编程的领域里迈出去了一大步。

为自己加个油，好吗？








