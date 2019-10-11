---
layout: post
title: 【集合系列】- 深入浅出分析 LinkedHashMap
tagline: by 炸鸡可乐
categories: Java
tags: 
  - Java
---

在上一章节，咱们深入浅出的分析了 HashMap，如果你已读过 HashMap 的讲解，一定能够想到本文将要讲解的 LinkedHashMap 其实也是一样的，LinkedHashMap 继承于 HashMap，不同的是 LinkedHashMap 插入的元素，可以按照插入的顺序读取！

<!--more-->
### 01、摘要
在集合系列的第一章，咱们了解到，Map 的实现类有 HashMap、LinkedHashMap、TreeMap、IdentityHashMap、WeakHashMap、Hashtable、Properties 等等。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/10bfb0b009e142c7a3aa11f5f92a25f5.jpg)

本文主要从数据结构和算法层面，探讨 LinkedHashMap 的实现。

### 02、简介
> LinkedHashMap 可以认为是 HashMap+LinkedList，它既使用 HashMap 操作数据结构，又使用 LinkedList 维护插入元素的先后顺序，内部采用双向链表（doubly-linked list）的形式将所有元素（ entry ）连接起来。

LinkedHashMap 继承了 HashMap，允许放入 key 为 null 的元素，也允许插入 value 为 null 的元素。从名字上可以看出该容器是 LinkedList 和 HashMap 的混合体，也就是说它同时满足 HashMap 和 LinkedList 的某些特性，可将 LinkedHashMap 看作采用 Linkedlist 增强的 HashMap。

打开 LinkedHashMap 源码，可以看到主要三个核心属性：
```
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>{

	/**双向链表的头节点*/
	transient LinkedHashMap.Entry<K,V> head;

	/**双向链表的尾节点*/
	transient LinkedHashMap.Entry<K,V> tail;

	/**
	  * 1、如果accessOrder为true的话，则会把访问过的元素放在链表后面，放置顺序是访问的顺序
	  * 2、如果accessOrder为false的话，则按插入顺序来遍历
	  */
	  final boolean accessOrder;
}
```

LinkedHashMap  在初始化阶段，默认按**插入顺序**来遍历
```
public LinkedHashMap() {
        super();
        accessOrder = false;
}
```

LinkedHashMap 采用的 Hash 算法和 HashMap 相同，不同的是，它重新定义了数组中保存的元素 Entry，该 Entry 除了保存当前对象的引用外，还保存了其上一个元素 before 和下一个元素 after 的引用，从而在哈希表的基础上又构成了双向链接列表。

源码如下：
```
static class Entry<K,V> extends HashMap.Node<K,V> {
		//before指的是链表前驱节点，after指的是链表后驱节点
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
}
```

![](http://www.justdojava.com/assets/images/2019/java/image-jay/822d1a7ec2cc4db5ba3d770164ffd610.png)

可以直观的看出，双向链表头部插入的数据为链表的入口，迭代器遍历方向是从链表的头部开始到链表尾部结束。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/004c0ab56db84068b944b659b13cf941.jpg)

除了可以保迭代历顺序，这种结构还有一个好处：迭代 LinkedHashMap 时不需要像 HashMap 那样遍历整个 table，而只需要直接遍历 header 指向的双向链表即可，也就是说 LinkedHashMap 的迭代时间就只跟 entry 的个数相关，而跟 table 的大小无关。

### 03、常用方法介绍
#### 3.1、get方法
get 方法根据指定的 key 值返回对应的 value。该方法跟 HashMap.get() 方法的流程几乎完全一样，默认按照插入顺序遍历。
```
public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
}
```
如果`accessOrder`为`true`的话，会把访问过的元素放在链表后面，放置顺序是访问的顺序
```
void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
}
```
测试用例：
```
public static void main(String[] args) {
		//accessOrder默认为false
        Map<String, String> accessOrderFalse = new LinkedHashMap<>();
        accessOrderFalse.put("1","1");
        accessOrderFalse.put("2","2");
        accessOrderFalse.put("3","3");
        accessOrderFalse.put("4","4");
        System.out.println("acessOrderFalse："+accessOrderFalse.toString());
		
		//accessOrder设置为true
        Map<String, String> accessOrderTrue = new LinkedHashMap<>(16, 0.75f, true);
        accessOrderTrue.put("1","1");
        accessOrderTrue.put("2","2");
        accessOrderTrue.put("3","3");
        accessOrderTrue.put("4","4");
        accessOrderTrue.get("2");//获取键2
        accessOrderTrue.get("3");//获取键3
        System.out.println("accessOrderTrue："+accessOrderTrue.toString());
}
```
输出结果：
```
acessOrderFalse：{1=1, 2=2, 3=3, 4=4}
accessOrderTrue：{1=1, 4=4, 2=2, 3=3}
```

#### 3.2、put方法
put(K key, V value) 方法是将指定的 key, value 对添加到 map 里。该方法首先会调用 HashMap 的插入方法，同样对 map 做一次查找，看是否包含该元素，如果已经包含则直接返回，查找过程类似于 get() 方法；如果没有找到，将元素插入集合。
```
/**HashMap 中实现*/
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
}
```
LinkedHashMap 中覆写的方法
```
// LinkedHashMap 中覆写
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    // 将 Entry 接在双向链表的尾部
    linkNodeLast(p);
    return p;
}

private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    // last 为 null，表明链表还未建立
    if (last == null)
        head = p;
    else {
        // 将新节点 p 接在链表尾部
        p.before = last;
        last.after = p;
    }
}
```

![](http://www.justdojava.com/assets/images/2019/java/image-jay/1734595d9b0e404989a073cf7545fce3.jpg)

#### 3.3、remove方法
remove(Object key) 的作用是删除 key 值对应的 entry，该方法实现逻辑主要以 HashMap 为主，首先找到 key 值对应的 entry，然后删除该 entry（修改链表的相应引用），查找过程跟 get() 方法类似，最后会调用 LinkedHashMap 中覆写的方法，将其删除！
```
/**HashMap 中实现*/
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode) {...}
            else {
                // 遍历单链表，寻找要删除的节点，并赋值给 node 变量
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode) {...}
            // 将要删除的节点从单链表中移除
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);    // 调用删除回调方法进行后续操作
            return node;
        }
    }
    return null;
}
```
 LinkedHashMap 中覆写的 afterNodeRemoval 方法
```
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    // 将 p 节点的前驱后后继引用置空
    p.before = p.after = null;
    // b 为 null，表明 p 是头节点
    if (b == null)
        head = a;
    else
        b.after = a;
    // a 为 null，表明 p 是尾节点
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```
![](http://www.justdojava.com/assets/images/2019/java/image-jay/dd32e68682a845e2bb00d5666f8039ad.jpg)

### 04、总结
LinkedHashMap 继承自 HashMap，所有大部分功能特性基本相同，二者唯一的区别是 LinkedHashMap 在 HashMap 的基础上，采用双向链表（doubly-linked list）的形式将所有 entry 连接起来，这样是为保证元素的迭代顺序跟插入顺序相同。

主体部分跟 HashMap 完全一样，多了 header 指向双向链表的头部，tail 指向双向链表的尾部，默认双向链表的迭代顺序就是 entry 的插入顺序。

### 05、参考
1、JDK1.7&JDK1.8 源码

2、[博客园 - CarpenterLee - Java集合框架源码剖析LinkedHashMap](https://www.cnblogs.com/CarpenterLee/p/5541111.html)
