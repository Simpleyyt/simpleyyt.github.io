---
layout: post
title: 【集合系列】- 深入浅出的分析 Hashtable
tagline: by 炸鸡可乐
categories: Java
tags: 
  - Java
---

Hashtable 一个元老级的集合类，早在 JDK 1.0 就诞生了，今天小编想和大家一起来揭开它的面纱！

<!--more-->
### 01、摘要
在集合系列的第一章，咱们了解到，Map 的实现类有 HashMap、LinkedHashMap、TreeMap、IdentityHashMap、WeakHashMap、Hashtable、Properties 等等。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/10bfb0b009e142c7a3aa11f5f92a25f5.jpg)

本文主要从数据结构和算法层面，探讨 Hashtable 的实现，如果有理解不当之处，欢迎指正。

### 02、简介
> Hashtable 一个元老级的集合类，早在 JDK 1.0 就诞生了，而 HashMap 诞生于 JDK 1.2，在实现上，HashMap 吸收了很多 Hashtable 的思想，虽然二者的底层数据结构都是 **数组 + 链表** 结构，具有查询、插入、删除快的特点，但是二者又有很多的不同。

打开 Hashtable 的源码可以看到，Hashtable 继承自 Dictionary，而 HashMap 继承自 AbstractMap。
```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    .....
}
```
HashMap 继承自 AbstractMap，HashMap 类的定义如下：
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    .....
}
```
其中 Dictionary 类是一个已经被废弃的类，翻译过来的意思是**这个类已经过时，新的实现应该实现 Map 接口而不是扩展此类**，这一点我们可以从它代码的注释中可以看到：
```java
/**
 * <strong>NOTE: This class is obsolete.  New implementations should
 * implement the Map interface, rather than extending this class.</strong>
 */
public abstract
class Dictionary<K,V> {
    ......
}
```
Hashtable 和 HashMap 的底层是以数组来存储，同时，在存储数据通过`key`计算数组下标的时候，是以哈希算法为主，因此可能会产生哈希冲突的可能性。

通俗的说呢，就是不同的`key`，在计算的时候，可能会产生相同的数组下标，这个时候，如何将两个对象放入一个数组中呢？

而解决哈希冲突的办法，有两种，一种开放地址方式（当发生 hash 冲突时，就继续以此继续寻找，直到找到没有冲突的hash值），另一种是拉链方式（将冲突的元素放入链表）。

**Java Hashtable 采用的就是第二种方式，拉链法！**

于是，当发生不同的`key`通过一系列的哈希算法计算获取到**相同的数组下标**的时候，会将对象放入一个数组容器中，然后将对象以`单向链表`的形式存储在同一个数组下标容器中，就像链子一样，挂在某个节点上，如下图：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/2502f76befa84f90aa3a6eedd29ac7eb.jpg)

与 HashMap 类似，Hashtable 也包括五个成员变量：
```java
/**由Entry对象组成的数组*/
private transient Entry[] table;

/**Hashtable中Entry对象的个数*/
private transient int count;

/**Hashtable进行扩容的阈值*/
private int threshold;

/**负载因子，默认0.75*/
private float loadFactor;

/**记录修改的次数*/
private transient int modCount = 0;
```
具体各个变量含义如下：
* **table：**表示一个由 Entry 对象组成的链表数组，Entry 是一个单向链表，哈希表的`key-value`键值对都是存储在 Entry 数组中的;
* **count：**表示 Hashtable 的大小，用于记录保存的键值对的数量;
* **threshold：**表示 Hashtable 的阈值，用于判断是否需要调整 Hashtable 的容量，threshold 等于`容量 * 加载因子`;
* **loadFactor：**表示负载因子，默认为 0.75；
* **modCount：**表示记录 Hashtable 修改的次数，用来实现快速失败抛异常处理；


接着来看看`Entry`这个内部类，`Entry`用于存储链表数据，实现了`Map.Entry`接口，本质是就是一个映射（键值对），源码如下：
```java
 private static class Entry<K,V> implements Map.Entry<K,V> {
     /**hash值*/
    final int hash;
    /**key表示键*/
    final K key;
    /**value表示值*/
    V value;
    /**节点下一个元素*/
    Entry<K,V> next;
    ......
}
```
我们再接着来看看 Hashtable 初始化过程，核心源码如下：
```java
public Hashtable() {
    this(11, 0.75f);
}
```
this 调用了自己的构造方法，核心源码如下：
```java
public Hashtable(int initialCapacity, float loadFactor) {
    .....
    //默认的初始大小为 11
    //并且计算扩容的阈值
    this.loadFactor = loadFactor;
    table = new Entry<?,?>[initialCapacity];
    threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}
```
可以看到 **HashTable 默认的初始大小为 11，如果在初始化给定容量大小，那么 HashTable 会直接使用你给定的大小**；

扩容的阈值`threshold`等于`initialCapacity * loadFactor`，我们在来看看 HashTable 扩容，方法如下：
```java
protected void rehash() {
    int oldCapacity = table.length;
    //将旧数组长度进行位运算，然后 +1
    //等同于每次扩容为原来的 2n+1
    int newCapacity = (oldCapacity << 1) + 1;
    
    //省略部分代码......
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];
}
```
可以看到，**HashTable 每次扩充为原来的 2n+1**。

我们再来看看 HashMap，如果是执行默认构造方法，会在扩容那一步，进行初始化大小，核心源码如下：
```java
final Node<K,V>[] resize() {
    int newCap = 0;

    //部分代码省略......
    newCap = DEFAULT_INITIAL_CAPACITY;//默认容量为 16
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
}
```
可以看出 **HashMap 的默认初始化大小为 16**，我们再来看看，HashMap 扩容方法，核心源码如下：
```java
final Node<K,V>[] resize() {
    //获取旧数组的长度
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int newCap = 0;

    //部分代码省略......
    //当进行扩容的时候，容量为 2 的倍数
    newCap = oldCap << 1;
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
}
```
可以看出 **HashMap 的扩容后的数组数量为原来的 2 倍**；

**也就是说 HashTable 会尽量使用素数、奇数来做数组的容量，而 HashMap 则总是使用 2 的幂作为数组的容量。**

我们知道当哈希表的大小为素数时，简单的取模哈希的结果会更加均匀，所以单从这一点上看，HashTable 的哈希表大小选择，似乎更高明些。

Hashtable 的 hash 算法，核心代码如下：
```java
//直接计算key.hashCode()
int hash = key.hashCode();

//通过除法取余计算数组存放下标
// 0x7FFFFFFF 是最大的 int 型数的二进制表示
int index = (hash & 0x7FFFFFFF) % tab.length;
```
**从源码部分可以看出，HashTable 的 key 不能为空，否则报空指针错误！**

但另一方面我们又知道，在取模计算时，如果模数是 2 的幂，那么我们可以直接使用位运算来得到结果，效率要大大高于做除法。**所以在 hash 计算数组下标的效率上，HashMap 却更胜一筹**，但是这也会引入了哈希分布不均匀的问题， HashMap 为解决这问题，又对 hash 算法做了一些改动，具体我们来看看。

HashMap 的 hash 算法，核心代码如下：

```java
/**获取hash值方法*/
static final int hash(Object key) {
	int h;
	// h = key.hashCode() 为第一步 取hashCode值（jdk1.7）
	// h ^ (h >>> 16)  为第二步 高位参与运算（jdk1.7）
	return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);//jdk1.8
}

/**获取数组下标方法*/
static int indexFor(int h, int length) {
    //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
    return h & (length-1);  //第三步 取模运算
}
```
HashMap 由于使用了2的幂次方，所以在取模运算时不需要做除法，只需要位的与运算就可以了。但是由于引入的 hash 冲突加剧问题，HashMap 在调用了对象的 hashCode 方法之后，又做了一些高位运算，也就是第二步方法，来打散数据，让哈希的结果更加均匀。

与此同时，在 jdk1.8 中 HashMap 还引进来红黑树实现，当冲突链表长度大于 8 的时候，会将链表结构改变成红黑树结构，让查询变得更快，**具体实现可以参见《集合系列》中的 HashMap 分析**。

### 03、常用方法介绍
#### 3.1、put方法
> put 方法是将指定的 key, value 对添加到 map 里。

put 流程图如下：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/ebdd7991f4634d4393568b0632d769ab.jpg)

打开 HashTable 的 put 方法，源码如下：
```java
public synchronized V put(K key, V value) {
    //当 value 值为空的时候，抛异常！
    if (value == null) {
        throw new NullPointerException();
    }

    Entry<?,?> tab[] = table;

    //通过key 计算存储下标
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    
    //循环遍历数组链表
    //如果有相同的key并且hash相同，进行覆盖处理
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }
    //加入数组链表中
    addEntry(hash, key, value, index);
    return null;
}
```
put 方法中的 addEntry 方法，源码如下：
```java
private void addEntry(int hash, K key, V value, int index) {
    //新增修改次数
    modCount++;

    Entry<?,?> tab[] = table;
    if (count >= threshold) {
       //数组容量大于扩容阀值，进行扩容
        rehash();
        
        tab = table;
        //重新计算对象存储下标
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }

    //将对象存储在数组中
    Entry<K,V> e = (Entry<K,V>) tab[index];
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}
```
addEntry 方法中的 rehash 方法，源码如下：
```java
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;

    //每次扩容为原来的 2n+1
    int newCapacity = (oldCapacity << 1) + 1;
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            //大于最大阀值，不再扩容
            return;
        newCapacity = MAX_ARRAY_SIZE;
    }
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

    modCount++;
    //重新计算扩容阀值
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    table = newMap;
    //将旧数组中的数据复制到新数组中
    for (int i = oldCapacity ; i-- > 0 ;) {
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;

            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```
总结流程如下：
* **1、通过 key 计算对象存储在数组中的下标；**
* **2、如果链表中有 key，直接进行新旧值覆盖处理；**
* **3、如果链表中没有 key，判断是否需要扩容，如果需要扩容，先扩容，再插入数据；**

有一个值得注意的地方是 **put 方法加了`synchronized`关键字**，所以，在同步操作的时候，是线程安全的。

#### 3.2、get方法
>  get 方法根据指定的 key 值返回对应的 value。

get 流程图如下：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/82a5ff28b5c14da29f3081db4cc2cd23.jpg)

打开 HashTable 的 get 方法，源码如下：
```java
public synchronized V get(Object key) {
    Entry<?,?> tab[] = table;
    //通过key计算节点存储下标
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            return (V)e.value;
        }
    }
    return null;
}
```
同样，有一个值得注意的地方是 **get 方法加了`synchronized`关键字**，所以，在同步操作的时候，是线程安全的。

#### 3.3、remove方法
> remove 的作用是通过 key 删除对应的元素。

remove 流程图如下：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/7486b49ac2f4410099b5764082a6d352.jpg)

打开 HashTable 的 remove 方法，源码如下：
```java
public synchronized V remove(Object key) {
    Entry<?,?> tab[] = table;
    //通过key计算节点存储下标
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    Entry<K,V> e = (Entry<K,V>)tab[index];
    //循环遍历链表，通过hash和key判断键是否存在
    //如果存在，直接将改节点设置为空，并从链表上移除
    for(Entry<K,V> prev = null ; e != null ; prev = e, e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            modCount++;
            if (prev != null) {
                prev.next = e.next;
            } else {
                tab[index] = e.next;
            }
            count--;
            V oldValue = e.value;
            e.value = null;
            return oldValue;
        }
    }
    return null;
}
```
同样，有一个值得注意的地方是 **remove 方法加了`synchronized`关键字**，所以，在同步操作的时候，是线程安全的。
### 04、总结
总结一下 Hashtable 与 HashMap 的联系与区别，内容如下：

* 1、虽然 HashMap 和 Hashtable 都实现了 Map 接口，但 Hashtable 继承于 Dictionary 类，而 HashMap 是继承于 AbstractMap；
* 2、HashMap 可以允许存在一个为 null 的 key 和任意个为 null 的 value，但是 HashTable 中的 key 和 value 都不允许为 null；
* 3、Hashtable 的方法是同步的，因为在方法上加了 synchronized 同步锁，而 HashMap 是非线程安全的；

尽管，Hashtable 虽然是线程安全的，但是我们一般不推荐使用它，因为有比它更高效、更好的选择 ConcurrentHashMap，在后面我们也会讲到它。

最后，引入来自 HashTable 的注释描述：

> If a thread-safe implementation is not needed, it is recommended to use HashMap in place of Hashtable. If a thread-safe highly-concurrent implementation is desired, then it is recommended to use java.util.concurrent.ConcurrentHashMap in place of Hashtable.

简单来说就是，如果你不需要线程安全，那么使用 HashMap，如果需要线程安全，那么使用 ConcurrentHashMap。

**HashTable 已经被淘汰了，不要在新的代码中再使用它。**
### 05、参考
1、JDK1.7&JDK1.8 源码

2、[博客园 - 程序员赵鑫  - HashMap和HashTable到底哪不同？ ](https://www.cnblogs.com/xinzhao/p/5644175.html)
