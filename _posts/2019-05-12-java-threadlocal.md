---
layout: post
category: java
title: 聊聊面试中的 ThreadLocal 原理和使用场景
tagline: by 子悠
tags: 
  - java
published: true
---

相信大家不管是在网上做题还是在面试中都经常被问过 ThreadLocal 的原理和用法，虽然一直知道这个东西的存在但是一直没有好好的研究一下原理，没有自己的知识体系。今天花点时间好好学习了一下，分享给有需要的朋友。
 <!--more-->

 
### `ThreadLocal` 是什么
ThreadLocal 是 JDK `java.lang` 包中的一个用来实现相同线程数据共享不同的线程数据隔离的一个工具。
我们来看下 JDK 源码中是如何解释的：

> This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its get or set method) has its own, independently initialized copy of the variable. ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).
>
> Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the ThreadLocal instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).

大致的意思是

> ThreadLocal 这个类提供线程局部变量，这些变量与其他正常的变量的不同之处在于，每一个访问该变量的线程在其内部都有一个独立的初始化的变量副本；ThreadLocal 实例变量通常采用`private static `在类中修饰。
> 
> 只要 ThreadLocal 的变量能被访问，并且线程存活，那每个线程都会持有 ThreadLocal 变量的副本。当一个线程结束时，它所持有的所有 ThreadLocal 相对的实例副本都可被回收。

一句话说就是 ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用（相同线程数据共享），也就是变量在线程间隔离（不同的线程数据隔离）而在方法或类间共享的场景。

### `ThreadLocal` 使用
我们先通过两个例子来看一下 `ThreadLocal` 的使用

例子 1 普通变量

```

import java.util.concurrent.CountDownLatch;


public class MyStringDemo {
    private String string;

    private String getString() {
        return string;
    }

    private void setString(String string) {
        this.string = string;
    }

    public static void main(String[] args) {
        int threads = 9;
        MyStringDemo demo = new MyStringDemo();
        CountDownLatch countDownLatch = new CountDownLatch(threads);
        for (int i = 0; i < threads; i++) {
            Thread thread = new Thread(() -> {
                demo.setString(Thread.currentThread().getName());
                System.out.println(demo.getString());
                countDownLatch.countDown();
            }, "thread - " + i);
            thread.start();
        }

    }

}


```

程序的运行的随机结果如下：

```

thread - 1
thread - 2
thread - 1
thread - 3
thread - 4
thread - 5
thread - 6
thread - 7
thread - 8

Process finished with exit code 0

```

从结果我们可以看出多个线程在访问同一个变量的时候出现的异常，线程间的数据没有隔离。下面我们来看下采用 `ThreadLocal` 变量的方式来解决这个问题的例子。

例子 2 `ThreadLocal` 变量

```

import java.util.concurrent.CountDownLatch;


public class MyThreadLocalStringDemo {
    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    private String getString() {
        return threadLocal.get();
    }

    private void setString(String string) {
        threadLocal.set(string);
    }

    public static void main(String[] args) {
        int threads = 9;
        MyThreadLocalStringDemo demo = new MyThreadLocalStringDemo();
        CountDownLatch countDownLatch = new CountDownLatch(threads);
        for (int i = 0; i < threads; i++) {
            Thread thread = new Thread(() -> {
                demo.setString(Thread.currentThread().getName());
                System.out.println(demo.getString());
                countDownLatch.countDown();
            }, "thread - " + i);
            thread.start();
        }
    }

}


```

程序运行结果

```
thread - 0
thread - 1
thread - 2
thread - 3
thread - 4
thread - 5
thread - 6
thread - 7
thread - 8

Process finished with exit code 0

```

从结果来看，这次我们很好的解决了多线程之间数据隔离的问题，十分方便。

这里可能有的朋友会觉得在例子 1 中我们完全可以通过加锁来实现这个功能。是的没错，加锁确实可以解决这个问题，但是在这里我们强调的是**线程数据隔离的问题，并不是多线程共享数据的问题**。假如我们这里除了`getString()` 之外还有很多其他方法也要用到这个 `String`，这个时候各个方法之间就没有显式的数据传递过程了，都可以直接中 `ThreadLocal` 变量中获取，这才是 `ThreadLocal` 的核心，**相同线程数据共享不同的线程数据隔离**。

由于`ThreadLocal` 是支持泛型的，这里采用的是存放一个 `String` 来演示，其实可以存放任何类型，效果都是一样的。

### `ThreadLocal` 源码分析
在分析源码前我们明白一个事那就是**对象实例与 `ThreadLocal` 变量的映射关系是由线程 `Thread` 来维护的**，**对象实例与 `ThreadLocal` 变量的映射关系是由线程 `Thread` 来维护的**，**对象实例与 `ThreadLocal` 变量的映射关系是由线程 `Thread` 来维护的**。重要的事情说三遍。换句话说就是对象实例与 `ThreadLocal` 变量的映射关系是存放的一个 `Map` 里面（这个 `Map` 是个抽象的 `Map` 并不是 `java.util` 中的 `Map` ），而这个 `Map` 是 `Thread` 类的一个字段！而真正存放映射关系的 `Map` 就是 `ThreadLocalMap`。下面我们通过源码的中几个方法来看一下具体的实现。

```

//set 方法
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

//获取线程中的ThreadLocalMap 字段！！
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

//创建线程的变量
void createMap(Thread t, T firstValue) {
     t.threadLocals = new ThreadLocalMap(this, firstValue);
}

```

在 `set` 方法中首先获取当前线程，然后通过 `getMap` 获取到当前线程的 `ThreadLocalMap` 类型的变量 `threadLocals`，如果存在则直接赋值，如果不存在则给该线程创建 `ThreadLocalMap` 变量并赋值。赋值的时候这里的 `this` 就是调用变量的对象实例本身。

```

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}


private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

```

`get` 方法也比较简单，同样也是先获取当前线程的 `ThreadLocalMap` 变量，如果存在则返回值，不存在则创建并返回初始值。

### `ThreadLocalMap` 源码分析
`ThreadLocal` 的底层实现都是通过 `ThreadLocalMap` 来实现的，我们先看下 `ThreadLocalMap` 的定义，然后再看下相应的 `set` 和 `get` 方法。

```

static class ThreadLocalMap {

    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    
    /**
     * The table, resized as necessary.
     * table.length MUST always be a power of two.
     */
    private Entry[] table;
}

```

`ThreadLocalMap` 中使用 `Entry[]` 数组来存放对象实例与变量的关系，并且实例对象作为 key，变量作为 value 实现对应关系。并且这里的 key 采用的是对实例对象的弱引用，（因为我们这里的 key 是对象实例，每个对象实例有自己的生命周期，这里采用弱引用就可以在不影响对象实例生命周期的情况下对其引用）。

```
private void set(ThreadLocal<?> key, Object value) {

    Entry[] tab = table;
    int len = tab.length;
    //获取 hash 值，用于数组中的下标
    int i = key.threadLocalHashCode & (len-1);

    //如果数组该位置有对象则进入
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        //k 相等则覆盖旧值
        if (k == key) {
            e.value = value;
            return;
        }

        //此时说明此处 Entry 的 k 中的对象实例已经被回收了，需要替换掉这个位置的 key 和 value
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    //创建 Entry 对象
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}


//获取 Entry
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

        
```

至此我们看完了 `ThreadLocal` 相关的 JDK 源码，我自己也有了更深入的了解，也希望能帮助到大家。

### 小结
在平时忙碌的工作中我们经常解决的是一个业务的需求，往往很少会涉及到底层的源码或者框架的具体实现代码。
其实这是很不好的，其实很多的东西的原理都是一样的，我们需要经常去看一下源码，了解一些底层的实现，不能总是停留在表层，代码看到多了，才能写出好的代码，并且还能学到很多东西。
随着我们知道的越来越多，我们会发现我们不知道的也越来越多。加油，共勉！
