---
layout: post
title: 【集合系列】- 深入浅出的分析 IdentityHashMap
tagline: by 炸鸡可乐
categories: 数据结构
tags: 
  - 炸鸡可乐
---

IdentityHashMap 从它的名字上可以看出来用于表示唯一的 HashMap，但是分析了其源码，发现其数据结构与 HashMap 使用的数据结构完全不同。

<!--more-->
### 01、摘要
在集合系列的第一章，咱们了解到，Map 的实现类有 HashMap、LinkedHashMap、TreeMap、IdentityHashMap、WeakHashMap、Hashtable、Properties 等等。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/10bfb0b009e142c7a3aa11f5f92a25f5.jpg)

应该有很多人不知道 IdentityHashMap 的存在，其中不乏工作很多年的 Java 开发者，本文主要从数据结构和算法层面，探讨 IdentityHashMap 的实现。

### 02、简介
IdentityHashMap 的数据结构很简单，底层实际就是一个 Object 数组，但是在存储上并没有使用**链表**来存储，而是将 K 和 V 都存放在 Object 数组上。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/745b08468d764907a497792155b9015c.jpg)

当添加元素的时候，会根据 Key 计算得到散列位置，如果发现该位置上已经有改元素，直接进行新值替换；如果没有，直接进行存放。当元素个数达到一定阈值时，Object 数组会自动进行扩容处理。

打开 IdentityHashMap 的源码，可以看到 IdentityHashMap 继承了 AbstractMap 抽象类，实现了 Map 接口、可序列化接口、可克隆接口。
```java
public class IdentityHashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, java.io.Serializable, Cloneable
{
    /**默认容量大小*/
    private static final int DEFAULT_CAPACITY = 32;
    
    /**最小容量*/
    private static final int MINIMUM_CAPACITY = 4;
    
    /**最大容量*/
    private static final int MAXIMUM_CAPACITY = 1 << 29;
    
    /**用于存储实际元素的表*/
    transient Object[] table;
    
    /**数组大小*/
    int size;

    /**对Map进行结构性修改的次数*/
    transient int modCount;

    /**key为null所对应的值*/
    static final Object NULL_KEY = new Object();
    
    ......
}
```
可以看到类的底层，使用了一个 Object 数组来存放元素；在对象初始化时，IdentityHashMap 容量大小为`64`；
```java
public IdentityHashMap() {
    //调用初始化方法
    init(DEFAULT_CAPACITY);
}
```
```java
private void init(int initCapacity) {
    //数组大小默认为初始化容量的2倍
    table = new Object[2 * initCapacity];
}
```
### 03、常用方法介绍
#### 3.1、put方法
put 方法是将指定的 key, value 对添加到 map 里。该方法首先会对map做一次查找，通过`==`判断是否存在`key`，如果有，则将`旧value`返回，将`新value`覆盖`旧value`；如果没有，直接插入，数组长度`+1`，返回`null`。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/13e3e93303eb4679b1b93621cef7e4cf.jpg)

源码如下：
```java
public V put(K key, V value) {
    //判断key是否为空，如果为空，初始化一个Object为key
    final Object k = maskNull(key);

    retryAfterResize: for (;;) {
        final Object[] tab = table;
        final int len = tab.length;
        //通过key、length获取数组小编
        int i = hash(k, len);
        
        //循环遍历是否存在指定的key
        for (Object item; (item = tab[i]) != null;
             i = nextKeyIndex(i, len)) {
             //通过==判断，是否数组中是否存在key
            if (item == k) {
                    V oldValue = (V) tab[i + 1];
                    //新value覆盖旧value
                tab[i + 1] = value;
                //返回旧value
                return oldValue;
            }
        }
        
        //数组长度 +1
        final int s = size + 1;
        //判断是否需要扩容
        if (s + (s << 1) > len && resize(len))
            continue retryAfterResize;

        //更新修改次数
        modCount++;
        //将k加入数组
        tab[i] = k;
        //将value加入数组
        tab[i + 1] = value;
        size = s;
        return null;
    }
}
```
maskNull 函数，判断 key 是否为空
```java
private static Object maskNull(Object key) {
    return (key == null ? NULL_KEY : key);
}
```
hash 函数，通过 key 获取 hash 值，结合数组长度通过位运算获取数组散列下标
```java
private static int hash(Object x, int length) {
    int h = System.identityHashCode(x);
    // Multiply by -127, and left-shift to use least bit as part of hash
    return ((h << 1) - (h << 8)) & (length - 1);
}
```
nextKeyIndex 函数，通过 hash 函数计算得到的数组散列下标，进行加2；因为一个 key、value 都存放在数组中，所以一个 map 对象占用两个数组下标，所以加2。
```java
private static int nextKeyIndex(int i, int len) {
    return (i + 2 < len ? i + 2 : 0);
}
```
resize 函数，通过数组长度，进行扩容处理，扩容之后的长度为当前长度的2倍
```java
private boolean resize(int newCapacity) {
    //扩容后的数组长度，为当前数组长度的2倍
    int newLength = newCapacity * 2;

    Object[] oldTable = table;
    int oldLength = oldTable.length;
    if (oldLength == 2 * MAXIMUM_CAPACITY) { // can't expand any further
        if (size == MAXIMUM_CAPACITY - 1)
            throw new IllegalStateException("Capacity exhausted.");
        return false;
    }
    if (oldLength >= newLength)
        return false;

    Object[] newTable = new Object[newLength];
    //将旧数组内容转移到新数组
    for (int j = 0; j < oldLength; j += 2) {
        Object key = oldTable[j];
        if (key != null) {
            Object value = oldTable[j+1];
            oldTable[j] = null;
            oldTable[j+1] = null;
            int i = hash(key, newLength);
            while (newTable[i] != null)
                i = nextKeyIndex(i, newLength);
            newTable[i] = key;
            newTable[i + 1] = value;
        }
    }
    table = newTable;
    return true;
}
```
#### 3.2、get方法
get 方法根据指定的 key 值返回对应的 value。同样的，该方法会循环遍历数组，通过`==`判断是否存在`key`，如果有，直接返回value，因为 key、value 是相邻的存储在数组中，所以直接在当前数组`下标+1`，即可获取 value；如果没有找到，直接返回`null`。

**值得注意的地方是**，在循环遍历中，是通过`==`判断当前元素是否与`key`相同，如果相同，则返回`value`。咱们都知道，在 java 中，`==`对于对象类型参数，判断的是`引用地址`，确切的说，是堆内存地址，所以，这里判断的是`key`的引用地址是否相同，如果相同，则返回对应的 value；如果不相同，则返回`null`。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/9931e2aabf1b41ca8ee1d60e348fe3b6.jpg)

源码如下：
```java
public V get(Object key) {
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);
    
    //循环遍历数组，直到找到key或者，数组为空为值
    while (true) {
        Object item = tab[i];
        //通过==判断，当前数组元素与key相同
        if (item == k)
            return (V) tab[i + 1];
        //数组为空
        if (item == null)
            return null;
        i = nextKeyIndex(i, len);
    }
}
```

#### 3.3、remove方法
remove 的作用是通过 key 删除对应的元素。该方法会循环遍历数组，通过`==`判断是否存在`key`，如果有，直接将`key`、`value`设置为`null`，对数组进行重新排列，返回旧 value。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/e5c95938b4f0472792331be81d92db29.jpg)

源码如下：
```java
public V remove(Object key) {
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);

    while (true) {
        Object item = tab[i];
        if (item == k) {
            modCount++;
            //数组长度减1
            size--;
                V oldValue = (V) tab[i + 1];
            //将key、value设置为null
            tab[i + 1] = null;
            tab[i] = null;
            //删除该元素后，需要把原来有冲突往后移的元素移到前面来
            closeDeletion(i);
            return oldValue;
        }
        if (item == null)
            return null;
        i = nextKeyIndex(i, len);
    }
}
```
closeDeletion 函数，删除该元素后，需要把原来有冲突往后移的元素移到前面来，对数组进行重写排列；
```java
private void closeDeletion(int d) {
    // Adapted from Knuth Section 6.4 Algorithm R
    Object[] tab = table;
    int len = tab.length;

    Object item;
    for (int i = nextKeyIndex(d, len); (item = tab[i]) != null;
         i = nextKeyIndex(i, len) ) {
        int r = hash(item, len);
        if ((i < r && (r <= d || d <= i)) || (r <= d && d <= i)) {
            tab[d] = item;
            tab[d + 1] = tab[i + 1];
            tab[i] = null;
            tab[i + 1] = null;
            d = i;
        }
    }
}
```
### 04、总结
1. `IdentityHashMap` 的实现不同于`HashMap`，虽然也是数组，不过`IdentityHashMap`中没有用到链表，解决冲突的方式是计算下一个有效索引，并且将数据`key`和`value`紧挨着存在`map`中，即`table[i]=key`、`table[i+1]=value`；

2. `IdentityHashMap` 允许`key`、`value`都为`null`，当`key`为`null`的时候，默认会初始化一个`Object`对象作为`key`；

3. `IdentityHashMap`在保存、删除、查询数据的时候，以`key`为索引，通过`==`来判断数组中元素是否与`key`相同，本质判断的是对象的引用地址，如果引用地址相同，那么在插入的时候，会将`value`值进行替换；


IdentityHashMap 测试例子：
```java
public static void main(String[] args) {
    Map<String, String> identityMaps = new IdentityHashMap<String, String>();

    identityMaps.put(new String("aa"), "aa");
    identityMaps.put(new String("aa"), "bb");
    identityMaps.put(new String("aa"), "cc");
    identityMaps.put(new String("aa"), "cc");
    //输出添加的元素
    System.out.println("数组长度："+identityMaps.size() + "，输出结果：" + identityMaps);
}
```
输出结果：
```java
数组长度：4，输出结果：{aa=aa, aa=cc, aa=bb, aa=cc}
```
尽管`key`的内容是一样的，但是`key`的堆地址都不一样，所以在插入的时候，插入了4条记录。
### 05、参考
1、JDK1.7&JDK1.8 源码

2、[简书 - 骑着乌龟去看海 - IdentityHashMap源码解析 ](https://www.jianshu.com/p/1b441546078a)

3、[博客园 - leesf - IdentityHashMap源码解析 ](https://www.cnblogs.com/leesf456/p/5253094.html)