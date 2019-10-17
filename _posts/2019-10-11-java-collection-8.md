---
layout: post
title: 【集合系列】- 深入浅出的分析 WeakHashMap
tagline: by 炸鸡可乐
categories: Java
tags: 
  - Java
---

在 Map 家族中，WeakHashMap 是一个很特殊的成员，从名字上看与 HashMap 相关，但是与 HashMap 有着很大的差别，翻译成中文后表示弱 HashMap，俗称缓存 HashMap。

<!--more-->
### 01、摘要
在集合系列的第一章，咱们了解到，Map 的实现类有 HashMap、LinkedHashMap、TreeMap、IdentityHashMap、WeakHashMap、Hashtable、Properties 等等。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/10bfb0b009e142c7a3aa11f5f92a25f5.jpg)

本文主要从数据结构和算法层面，探讨 WeakHashMap 的实现。

### 02、简介
> 刚刚咱们也介绍了，在 Map 家族中，WeakHashMap 是一个很特殊的成员，它的特殊之处在于 WeakHashMap 里的元素可能会被 GC 自动删除，即使程序员没有显示调用 remove() 或者 clear() 方法。

换言之，当向 WeakHashMap 中添加元素的时候，再次遍历获取元素，可能发现它已经不见了，我们来看看下面这个例子。
```java
public static void main(String[] args) {
    Map weakHashMap = new WeakHashMap();
    
    //向weakHashMap中添加4个元素
    for (int i = 0; i < 3; i++) {
        weakHashMap.put("key-"+i, "value-"+ i);
    }
    //输出添加的元素
    System.out.println("数组长度："+weakHashMap.size() + "，输出结果：" + weakHashMap);
    
    //主动触发一次GC
    System.gc();
    
    //再输出添加的元素
    System.out.println("数组长度："+weakHashMap.size() + "，输出结果：" + weakHashMap);
}
```
输出结果：
```java
数组长度：3，输出结果：{key-2=value-2, key-1=value-1, key-0=value-0}
数组长度：3，输出结果：{}
```
当主动调用 GC 回收器的时候，再次查询 WeakHashMap 里面的数据的时候，内容为空。

更直观的说，当使用 WeakHashMap 时，即使没有显式的添加或删除任何元素，也可能发生如下情况：

* **调用两次 size() 方法返回不同的值；**
* **两次调用 isEmpty() 方法，第一次返回 false，第二次返回 true；**
* **两次调用 containsKey() 方法，第一次返回 true，第二次返回 false，尽管两次使用的是同一个key；**
* **两次调用 get() 方法，第一次返回一个 value，第二次返回 null，尽管两次使用的是同一个对象。**

要明白 WeekHashMap 的工作原理，还需要引入一个概念：**弱引用**。

我们都知道 Java 中内存是通过 GC 自动管理的，GC 会在程序运行过程中自动判断哪些对象是可以被回收的，并在合适的时机进行内存释放。

GC 判断某个对象是否可被回收的依据是，是否有有效的引用指向该对象。如果没有有效引用指向该对象（基本意味着不存在访问该对象的方式），那么该对象就是可回收的。

#### 2.1、对象引用介绍
从 JDK1.2 版本开始，把对象的引用分为四种级别，从而使程序更加灵活的控制对象的生命周期。这四种级别由高到低依次为：强引用、软引用、弱引用和虚引用。

用表格整理之后，各个引用类型的区别如下：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/5c865f9b44a04fdc8c49e5c5b1c31c42.jpg)

##### 2.1.1、强引用
强引用是使用最普遍的引用，例如，我们创建一个对象：
```java
//强引用类型
Object object=new Object();
```
如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足， Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。

如果不使用时，要手动通过如下方式来弱化引用，如下：
```java
//将对象设置为null，帮助垃圾收集器回收此对象
object=null;
```
这个时候，GC 认为该对象不存在引用，就可以回收这个对象，具体什么时候收集这要取决于 GC 的算法。

##### 2.1.2、软引用
被`SoftReference`指向的对象，属于软引用，如下：
```java
String str=new String("abc");

//软引用
SoftReference<String> softRef=new SoftReference<String>(str);
```
如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会进入垃圾回收器，Java 虚拟机就会把这个软引用加入到与之关联的`引用队列`中，GC 进行回收处理。只要垃圾回收器没有回收它，该对象就可以被程序使用。

当内存不足时，等价于：
```java
If(JVM.内存不足()) {
   str = null;  // 转换为软引用
   System.gc(); // 垃圾回收器进行回收
}
```
软引用的这种特性，比较适合内存敏感的场景，做高速缓存。在某些场景下，比如，系统内存不是很足的情况下，可以使用软引用，GC 会自动回收，再次获取对象的时候，可以对缓存对象进行重建，而又不影响使用。比如：
```java
//创建一个缓存内容cache
String cache = new String("abc");

//进行软引用处理
SoftReference<String> softRef=new SoftReference<String>(cache);

//判断是否被垃圾回收器回收
if(softRef.get()!=null){
    //还没有被回收器回收，直接获取
    cache = (String) softRef.get();
}else{
    //由于内存吃紧，所以对软引用的对象回收了
    //重建缓存对象
    cache = new String("abc");
    SoftReference<String> softRef = new SoftReference<String>(cache);
}
```
##### 2.1.3、弱引用
被`WeakReference `指向的对象，属于弱引用，如下：
```java
String str=new String("abc");

//弱引用
WeakReference<String> abcWeakRef = new WeakReference<String>(str);
```
**弱引用与软引用的区别在于：具有弱引用的对象拥有更短暂的生命周期。**

在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

当垃圾回收器进行扫描回收时，等价于：
```java
str = null;
System.gc();
```
如果这个对象是偶尔的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你应该用 WeakReference 来记住此对象。

同样的，弱引用对象进入垃圾回收器，Java 虚拟机就会把这个弱引用加入到与之关联的`引用队列`中，GC 进行回收处理。

##### 2.1.4、虚引用
被`PhantomReference `指向的对象，属于虚引用。

**虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列联合使用**，如下：
```java
String str=new String("abc");

//创建引用队列
ReferenceQueue<String> queue = new ReferenceQueue<String>();

//创建虚引用
PhantomReference<String> phantomReference = new PhantomReference<String>(str, queue);
```
虚引用，顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

当垃圾回收器准备回收一个对象时，如果发现它是虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中，GC 进行回收处理。

##### 2.1.5、总结
Java 4中引用的级别由高到低依次为：**强引用  >  软引用  >  弱引用  >  虚引用**。

用一张图来看一下他们之间在垃圾回收时的区别：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/f6faf1b769ed41c092ab84403969c2d8.png)

再次回到本文要讲的 WeakHashMap！

WeakHashMap 内部是通过弱引用来管理 entry 的，弱引用的特性对应到 WeakHashMap 上意味着什么呢？将一对 key, value 放入到 WeakHashMap 里，随时都有可能被 GC 回收。

下面，咱们一起来看看 WeakHashMap 的具体实现。
### 03、常用方法介绍
#### 3.1、put方法
put 方法是将指定的 key, value 对添加到 map 里，存储结构类似于 HashMap；
不同的是，WeakHashMap 中存储的 Entry 继承自 WeakReference，实现了弱引用。

打开源码如下：
```java
public V put(K key, V value) {
    Object k = maskNull(key);
    int h = hash(k);
    Entry<K,V>[] tab = getTable();
    int i = indexFor(h, tab.length);

    for (Entry<K,V> e = tab[i]; e != null; e = e.next) {
        if (h == e.hash && eq(k, e.get())) {
            V oldValue = e.value;
            if (value != oldValue)
                e.value = value;
            return oldValue;
        }
    }

    modCount++;
    Entry<K,V> e = tab[i];
    tab[i] = new Entry<>(k, value, queue, h, e);
    if (++size >= threshold)
        resize(tab.length * 2);
    return null;
}
```
WeakHashMap 中存储的 Entry，源码如下：
```java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
    V value;
    final int hash;
    Entry<K,V> next;

    Entry(Object key, V value,
          ReferenceQueue<Object> queue,
          int hash, Entry<K,V> next) {
          
        //将key进行弱引用处理
        super(key, queue);
        this.value = value;
        this.hash  = hash;
        this.next  = next;
    }
    ......
}
```
需要注意的是，Entry 中`super(key, queue)`，传入的是`key`，因此`key`才是进行弱引用的，`value`是直接强引用关联在`this.value`中，`System.gc()`时，对`key`进行了回收，而`value`依然保持。

那`value`是何时被清除的呢？

阅读源码，可以看到，调用`getTable()`函数，对调用`expungeStaleEntries()`函数，**该方法对 jvm 要回收的的 entry(quene 中) 进行遍历，并将 entry 的 value 设置为空，进行内存回收。**
```java
private Entry<K,V>[] getTable() {
    expungeStaleEntries();
    return table;
}
```
`expungeStaleEntries()`函数，源码如下：
```java
private void expungeStaleEntries() {
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
                Entry<K,V> e = (Entry<K,V>) x;
            int i = indexFor(e.hash, table.length);

            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            while (p != null) {
                Entry<K,V> next = p.next;
                if (p == e) {
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    //将value设置为null，方便GC回收
                    e.value = null; // Help GC
                    size--;
                    break;
                }
                prev = p;
                p = next;
            }
        }
    }
}
```
所以效果是 key 在 GC 的时候被清除，value 在 key 清除后，访问数组内容的时候进行清除！
#### 3.2、get方法
get 方法根据指定的 key 值返回对应的 value。

源码如下：
```java
public V get(Object key) {
    Object k = maskNull(key);
    int h = hash(k);
    //访问数组内容
    Entry<K,V>[] tab = getTable();
    int index = indexFor(h, tab.length);
    Entry<K,V> e = tab[index];
    while (e != null) {
        //通过key，进行hash值和equals判断
        if (e.hash == h && eq(k, e.get()))
            return e.value;
        e = e.next;
    }
    return null;
}
```
同样的，get 方法在判断对象之前，也调用了`getTable()`函数，同时，也调用了`expungeStaleEntries()`函数，所以，可能通过 key 获取元素的时候，得到空值；如果 key 没有被 GC 回收，那么就返回对应的 value。
#### 3.3、remove方法
remove 的作用是通过 key 删除对应的元素。

源码如下：
```java
public V remove(Object key) {
    Object k = maskNull(key);
    int h = hash(k);
    
    //访问数组内容
    Entry<K,V>[] tab = getTable();
    int i = indexFor(h, tab.length);
    Entry<K,V> prev = tab[i];
    Entry<K,V> e = prev;
    
    //循环链表，通过key，进行hash值和equals判断
    while (e != null) {
        Entry<K,V> next = e.next;
        if (h == e.hash && eq(k, e.get())) {
            modCount++;
            size--;
            //找到之后，将链表后节点向前移动
            if (prev == e)
                tab[i] = next;
            else
                prev.next = next;
            return e.value;
        }
        prev = e;
        e = next;
    }

    return null;
}
```
同样的，remove 方法在判断对象之前，也调用了`getTable()`函数，同时，也调用了`expungeStaleEntries()`函数，所以，可能通过 key 获取元素的时候，可能被垃圾回收器回收，得到空值。
### 04、总结
WeakHashMap 跟普通的 HashMap 不同，在存储数据时，`key`被设置为`弱引用类型`，而`弱引用类型`在 java 中，可能随时被 jvm 的 gc 回收，所以再次通过获取对象时，可能得到空值，而`value`是在访问数组内容的时候，进行清除。

可能很多人觉得这样做很奇葩，其实不然，WeekHashMap 的这个特点特别适用于需要缓存的场景。

在缓存场景下，由于系统内存是有限的，不能缓存所有对象，可以使用 WeekHashMap 进行缓存对象，即使缓存丢失，也可以通过重新计算得到，不会造成系统错误。

### 05、参考
1、JDK1.7&JDK1.8 源码

2、[知乎 - CarpenterLee  - 浅谈WeakHashMap ](https://zhuanlan.zhihu.com/p/24887482?refer=dreawer)

3、[csdn - Vander丶 - Java四种引用](https://blog.csdn.net/l540675759/article/details/73733763)

4、[csdn - java-er - Java四种引用](https://blog.csdn.net/mazhimazh/article/details/19752475)
