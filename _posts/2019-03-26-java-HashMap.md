---
layout: post
title:  HashMap源码分析
tagline: by 懿
categories: java
tag: 
    - java
---


HashMap是在面试中经常会问的一点，很多时候我们仅仅只是知道HashMap他是允许键值对都是Null，并且是非线程安全的，如果在多线程的环境下使用，是很容易出现问题的。
这是我们通常在面试中会说的，但是有时候问到底层的源码分析的时候，为什么允许为Null，为什么不安全，这些问题的时候，如果没有分析过源码的话，好像很难回答，
这样的话我们来研究一下这个源码。看看原因把。
<!--more-->

  HashMap最早出现在JDK1.2中，它的底层是基于的散列算法。允许键值对都是Null，并且是非线程安全的，我们先看看这个1.8版本的JDK中HashMap的数据结构把。
 
  ##HashMap图解如下
  
![](/assets/images/2019/java/image_yi/HashMap.jpg)

我们都知道HashMap是数组+链表组成的，bucket数组是HashMap的主体，而链表是为了解决哈希冲突而存在的，但是很多人不知道其实HashMap是包含树结构的，但是得有一点
注意事项，什么时候会出现红黑树这种红树结构的呢？我们就得看源码了，源码解释说默认链表长度大于8的时候会转换为树。我们看看源码说的
####结构

```
/**
 * Basic hash bin node, used for most entries.  (See below for
 * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
 */
 /**
    Node是hash基础的节点，是单向链表，实现了Map.Entry接口
 */
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
    //构造函数
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
  public final K getKey()        { return key; }
  public final V getValue()      { return value; }
  public final String toString() { return key + "=" + value; }
  
  public final int hashCode() {
              return Objects.hashCode(key) ^ Objects.hashCode(value);
   }
   
  public final V setValue(V newValue) {
              V oldValue = value;
              value = newValue;
              return oldValue;
  }
  public final boolean equals(Object o) {
      if (o == this)
          return true;
      if (o instanceof Map.Entry) {
          Map.Entry<?,?> e = (Map.Entry<?,?>)o;
          if (Objects.equals(key, e.getKey()) &&
              Objects.equals(value, e.getValue()))
              return true;
      }
      return false;
  }
}
```
####接下来就是树结构了
TreeNode 是红黑树的数据结构。
```
     /**
     * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
     * extends Node) so can be used as extension of either regular or
     * linked node.
     */
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
     /**
      * Returns root of tree containing this node.
      */
     final TreeNode<K,V> root() {
         for (TreeNode<K,V> r = this, p;;) {
             if ((p = r.parent) == null)
                 return r;
             r = p;
         }
     }
```
####我们在看一下类的定义
```
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```
继承了抽象的map，实现了Map接口，并且进行了序列化。

在类里还有基础的变量
####变量
```
/**
 * The default initial capacity - MUST be a power of two.
 *  默认初始容量 16 - 必须是2的幂
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<30.
 * 最大容量 2的30次方
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * The load factor used when none specified in constructor.
 * 默认加载因子，用来计算threshold
 */
 static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * The bin count threshold for using a tree rather than list for a
 * bin.  Bins are converted to trees when adding an element to a
 * bin with at least this many nodes. The value must be greater
 * than 2 and should be at least 8 to mesh with assumptions in
 * tree removal about conversion back to plain bins upon
 * shrinkage.
 * 链表转成树的阈值，当桶中链表长度大于8时转成树 
 * threshold = capacity * loadFactor
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * The bin count threshold for untreeifying a (split) bin during a
 * resize operation. Should be less than TREEIFY_THRESHOLD, and at
 * most 6 to mesh with shrinkage detection under removal.
 * 进行resize操作时，若桶中数量少于6则从树转成链表
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * The smallest table capacity for which bins may be treeified.
 * (Otherwise the table is resized if too many nodes in a bin.)
 * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
 * between resizing and treeification thresholds.
 * 桶中结构转化为红黑树对应的table的最小大小
 * 当需要将解决 hash 冲突的链表转变为红黑树时，
 * 需要判断下此时数组容量，
 * 若是由于数组容量太小（小于　MIN_TREEIFY_CAPACITY　）
 * 导致的 hash 冲突太多，则不进行链表转变为红黑树操作，
 * 转为利用　resize() 函数对　hashMap 扩容
 */
static final int MIN_TREEIFY_CAPACITY = 64;

/**
 * The table, initialized on first use, and resized as
 * necessary. When allocated, length is always a power of two.
 * (We also tolerate length zero in some operations to allow
 * bootstrapping mechanics that are currently not needed.)
 * 保存Node<K,V>节点的数组
 * 该表在首次使用时初始化，并根据需要调整大小。 分配时，
 * 长度始终是2的幂。
 */
transient Node<K,V>[] table;

/**
 * Holds cached entrySet(). Note that AbstractMap fields are used
 * for keySet() and values().
 * 存放具体元素的集
 */
transient Set<Map.Entry<K,V>> entrySet;

/**
 * The number of key-value mappings contained in this map.
 * 记录 hashMap 当前存储的元素的数量
 */
transient int size;

/**
 * The number of times this HashMap has been structurally modified
 * Structural modifications are those that change the number of mappings in
 * the HashMap or otherwise modify its internal structure (e.g.,
 * rehash).  This field is used to make iterators on Collection-views of
 * the HashMap fail-fast.  (See ConcurrentModificationException).
 * 每次更改map结构的计数器
 */
transient int modCount;

/**
 * The next size value at which to resize (capacity * load factor).
 * 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
 * @serial
 */
// (The javadoc description is true upon serialization.
// Additionally, if the table array has not been allocated, this
// field holds the initial array capacity, or zero signifying
// DEFAULT_INITIAL_CAPACITY.)
int threshold;

/**
 * The load factor for the hash table.
 * 负载因子：要调整大小的下一个大小值（容量*加载因子）。
 * @serial
 */
final float loadFactor;
```
我们再看看构造方法
#### 构造方法

```
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and the default load factor (0.75).
 *
 * @param  initialCapacity the initial capacity.
 * @throws IllegalArgumentException if the initial capacity is negative.
 * 传入初始容量大小，使用默认负载因子值 来初始化HashMap对象
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

/**
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity
 * (16) and the default load factor (0.75).
 * 默认容量和负载因子
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 * 传入初始容量大小和负载因子 来初始化HashMap对象
 */
public HashMap(int initialCapacity, float loadFactor) {
     // 初始容量不能小于0，否则报错
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // 初始容量不能大于最大值，否则为最大值    
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //负载因子不能小于或等于0，不能为非数字
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    // 初始化负载因子
    this.loadFactor = loadFactor;
    // 初始化threshold大小
    this.threshold = tableSizeFor(initialCapacity);
}

/**
 * Returns a power of two size for the given target capacity.
 * 找到大于或等于 cap 的最小2的整数次幂的数
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
在这源码中，loadFactor负载因子是一个非常重要的参数，因为他能够反映HashMap桶数组的使用情况，
这样的话，HashMap的时间复杂度就会出现不同的改变。

当这个负载因子属于低负载因子的时候，HashMap所能够容纳的键值对数量就是偏少的，扩容后，重新将键值对
存储在桶数组中，键与键之间产生的碰撞会下降，链表的长度也会随之变短。


但是如果增加负载因子当这个负载因子大于1的时候，HashMap所能够容纳的键值对就会变多，这样碰撞就会增加，
这样的话链表的长度也会增加，

一般情况下负载因子我们都不会去修改。都是默认的0.75。

#### 扩容机制
resize()这个方法就是重新计算容量的一个方法，我们看看源码：

```
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * @return the table
 */
final Node<K,V>[] resize() {
    //引用扩容前的Entry数组
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
       
        // 扩容前的数组大小如果已经达到最大(2^30)了
        //在这里去判断是否达到最大的大小 
        if (oldCap >= MAXIMUM_CAPACITY) {
               //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        
        // 如果扩容后小于最大值 而且 旧数组桶大于初始容量16， 阈值左移1(扩大2倍)
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 如果数组桶容量<=0 且 旧阈值 >0
    else if (oldThr > 0) // initial capacity was placed in threshold
        //新的容量就等于旧的阀值
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
         // 如果数组桶容量<=0 且 旧阈值 <=0
         // 新容量=默认容量
         // 新阈值= 负载因子*默认容量
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 如果新阈值为0
    if (newThr == 0) {
        // 重新计算阈值
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //在这里就会 更新阈值
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
     //创建新的数组
     Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    
    // 覆盖数组桶
    table = newTab;
     // 如果旧数组桶不是空，则遍历桶数组，并将键值对映射到新的桶数组中
    //在这里还有一点诡异的，1.7是不存在后边红黑树的，但是1.8就是有红黑树
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                
                // 如果是红黑树
                else if (e instanceof TreeNode)
                
                    // 重新映射时，然后对红黑树进行拆分
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    // 如果不是红黑树，那也就是说他链表长度没有超过8，那么还是链表，
                    //那么还是会按照链表处理
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    
                    // 遍历链表，并将链表节点按原顺序进行分组
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 将分组后的链表映射到新桶中
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

所以说在经过resize这个方法之后，元素的位置要么就是在原来的位置，要么就是在原来的位置移动2次幂的位置上。
源码上的注释也是可以翻译出来的
```
/**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     
     如果为null，则分配符合字段阈值中保存的初始容量目标。 
     否则，因为我们使用的是2次幂扩展，
     所以每个bin中的元素必须保持相同的索引，或者在新表中以2的偏移量移动。
     
     */
    final Node<K,V>[] resize() .....
```

所以说他的扩容其实很有意思，就有了三种不同的扩容方式了，
1. 在HashMap刚初始化的时候，使用默认的构造初始化，会返回一个空的table，并且
thershold为0，因此第一次扩容的时候默认值就会是16.
同时再去计算thershold = DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY = 16*0.75 = 12.

2. 如果说指定初始容量的初始HashMap的时候，那么这时候计算这个threshold的时候就变成了
threshold = DEFAULT_LOAD_FACTOR * threshold(当前的容量)

3. 如果HashMap不是第一次扩容，已经扩容过了，那么每次table的容量和threshold也会变成原来的2倍。

之前看1.7的源码的时候，是没有这个红黑树的，而是在1.8 之后做了相应的优化。
使用的是2次幂的扩展(指长度扩为原来2倍)。
而且在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，这样子他就剩下了计算hash的时间了。

看完这个源码，翻译了一节节的英文，算是大致明白了一点源码内容了，有什么讨论的问题咱们可以一起讨论一下，感谢观看。


