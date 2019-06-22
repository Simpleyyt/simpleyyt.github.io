---
layout: post  
title:  Java 中 hashCode() 和 equals() 的关系   
tagline: by 炭烧生蚝  
categories: Java  
tag: 
    - Java
---

Java 中 hashCode() 和 equals() 的关系是面试中的常考点，如果没有深入思考过两者设计的初衷，这个问题将很难回答。除了应付面试，理解二者的关系更有助于我们写出高质量且准确的代码。

<!--more-->
# 一.基础：hashCode() 和 equals() 简介

> 在学习 hashCode() 和 equals() 之间的关系之前, 我们有必要先单独地了解他俩的特点.

## equals()
equals() 方法用于比较两个对象是否相等，它与 == 相等比较符有着本质的不同。

在万物皆对象的 Java 体系中，系统把判断对象是否相等的权力交给程序员。具体的措施是把 equals() 方法写到 Object 类中，并让所有类继承 Object 类。 这样程序员就能在自定义的类中重写 equals() 方法, 从而实现自己的比较逻辑。

关于 equals() 和 == 的区别你可以[--参考这篇文章--](https://www.cnblogs.com/tanshaoshenghao/p/10896512.html)

## hashCode()
hashCode() 的意思是哈希值, 哈希值是经哈希函数运算后得到的结果，哈希函数能够保证相同的输入能够得到相同的输出(哈希值)，但是不能够保证不同的输入总是能得出不同的输出。

当输入的样本量足够大时，是会产生哈希冲突的，也就是说不同的输入产生了相同的输出。

暂且不谈冲突，就相同的输入能够产生相同的输出这点而言，是及其宝贵的。它使得系统只需要通过简单的运算，在时间复杂度O(1)的情况下就能得出数据的映射关系，根据这种特性，散列表应运而生。

一种主流的散列表实现是：用数组作为哈希函数的输出域，输入值经过哈希函数计算后得到哈希值。然后根据哈希值，在数组种找到对应的存储单元。当发生冲突时，对应的存储单元以链表的形式保存冲突的数据。


# 二. 漫谈：初识 hashCode() 与 equals() 之间的关系

> 下面我们从一个宏观的角度讨论 hashCode() 和 equals() 之间的关系。

在大多数编程实践中，归根结底会落实到数据的存取问题上。 在汇编语言时代，你需要老老实实地对每个数据操作编写存取语句。

而随着时代发展到今天，我们都用更方便灵活的高级语言编写代码，比如 Java。

Java 以面向对象为核心思想，封装了一系列操作数据的 api，降低了数据操作的复杂度。

但在我们对数据进行操作之前，首先要把数据按照一定的数据结构保存到存储单元中，否则操作数据将无从谈起。

然而不同的数据结构有各自的特点，我们在存储数据的时候需要选择合适的数据结构进行存储。 Java 根据不同的数据结构提供了丰富的容器类，方便程序员选择适合业务的容器类进行开发。

通过继承关系图我们看到 Java 的容器类被分为 Collection 和 Map 两大类，Collection 又可以进一步分为 List 和 Set。  其中 Map 和 Set 都是不允许元素重复的，严格来说Map存储的是键值对，它不允许重复的键值。

值得注意的是：Map 和 Set 的绝大多数实现类的底层都会用到散列表结构。

讲到这里我们提取两个关键字**不允许重复**和**散列表结构**，回顾 hashCode() 和 equals() 的特点，你是否想到了些什么东西呢？

# 三. 解密：深入理解 hashCode() 和 equals() 之间的关系

## equals() 会有力不从心的时候

上面提到 Set 和 Map 不存放重复的元素（key），这些容器在存储元素的时必须对元素做出判断：**在当前的容器中有没有和新元素相同的元素？**

你可能会想：这容易呀，直接调用元素对象的 equals() 方法进行比较不就行了吗？

如果容器中的存储的对象数量较少，这确实是个好主意，但是如果容器中存放的对象达到了一定的规模，要调用容器中所有对象的 equals() 方法和新元素进行比较，就不是一件容易的事情了。

就算 equals() 方法的比较逻辑简单无比，总的来说也是一个时间复杂度为 O(n) 的操作啊。

## hashCode() 小力出奇迹

但在散列表的基础上，判断“新对象是否和已存在对象相同”就容易得多了。

由于每个对象都自带有 hashCode()，这个 hashCode 将会用作散列表哈希函数的输入，hashCode 经过哈希函数计算后得到哈希值，新对象会根据哈希值，存储到相应的内存的单元。

我们不妨假设**两个相同的对象，hashCode() 一定相同**，这么一来就体现出哈希函数的威力了。

由于相同的输入一定会产生相同的输出，于是如果新对象，和容器中已存在的对象相同，新对象计算出的哈希值就会和已存在的对象的哈希值产生冲突。

这时容器就能判断：这个新加入的元素已经存在，需要另作处理：覆盖掉原来的元素（key）或舍弃。

按照这个思路，如果这个元素计算出的哈希值所对应的内存单元没有产生冲突，也就是没有重复的元素，那么它就可以直接插入。

**所以当运用 hashCode() 时，判断是否有相同元素的代价，只是一次哈希计算，时间复杂度为O(1)**，这极大地提高了数据的存储性能。

## Java 设计 equals()，hashCode() 时约定的规则

前面我们还提到：当输入样本量足够大时，不相同的输入是会产生相同输出的，也就是形成哈希冲突。

这么一来就麻烦了，原来我们设定的“如果产生冲突，就意味着两个对象相同”的规则瞬间被打破，因为产生冲突的很有可能是两个不同的对象！

而令人欣慰的是我们除了 hashCode() 方法，还有一张王牌：equals() 方法。

也就是说当两个不相同的对象产生哈希冲突后，我们可以用 equals() 方法进一步判断两个对象是否相同。

这时 equals() 方法就相当重要了，这个情况下它必须要能判定这两个对象是不相同的。

- 讲到这里就引出了 Java 程序设计中一个重要原则：

**如果两个对象是相等的，它们的 equals() 方法应该要返回 true，它们的 hashCode() 需要返回相同的结果**。

但有时候面试不会问得这么直接，他会问你：**两个对象的 hashCdoe() 相同，它的 equals() 方法一定要返回 true，对吗？**

那答案肯定不对。因为我们不能保证每个程序设计者，都会遵循编码约定。

有可能两个不同对象的hashCode()会返回相同的结果，但是由于他们是不同的对象，他们的 equals() 方法会返回false。

如果你理解上面的内容，这个问题就很好解答，我们再回顾一下：

如果两个对象的 hashCode() 相同，将来就会在散列表中产生哈希冲突，但是它们不一定是相同的对象呀。

当产生哈希冲突时，我们还得通过 equals() 方法进一步判断两个对象是否相同，equals() 方法不一定会返回 true。

这也是为什么 Java 官方推荐我们在一个类中，最好同时重写 hashCode() 和 equals() 方法的原因。


# 四. 验证：结合 HashMap 的源码和官方文档，验证两者的关系

> 以上的文字，是我经过思考后得出的，它有一定依据但并非完全可靠。下面我们根据 HashMap 的源码（JDK1.8）和官方文档，来验证这些推论是否正确。

通过阅读JDK8的官方文档，我们发现 equals() 方法介绍的最后有这么一段话：

> Note that it is generally necessary to override the hashCode method whenever this method is overridden, so as to maintain the general contract for the hashCode method, which states that equal objects must have equal hash codes.

官方文档提醒我们当重写 equals() 方法的时候，最好也要重写 hashCode() 方法。

也就是说如果我们通过重写 equals() 方法判断两个对象相同时，他们的hash code也应该相同，这样才能让hashCode()方法发挥它的作用。

那它究竟能发会怎样的作用呢？

我们结合部分较为常用的 HashMap 源码进一步分析。（像 HashSet 底层也是通过 HashMap 实现的）

在 HashMap 中用得最多无疑是 put() 方法了，以下是put()的源码：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

我们可以看到 put() 方法实际调用的是 putVal() 方法，继续跟进：

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //在我们创建HashMap对象的时候, 内存中并没有为HashMap分配表的空间, 直到往HashMap中put添加元素的时候才调用resize()方法初始化表
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;//同时确定了表的长度
        
    //((n - 1) & hash)确定了要put的元素的位置, 如果要插入的地方是空的, 就可以直接插入.
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {//如果发生了冲突, 就要在冲突位置的链表末尾插入元素
        Node<K,V> e; K k;
        if (p.hash == hash &&   
            ((k = p.key) == key || (key != null && key.equals(k))))
            //关键!!!当判断新加入的元素是否与已有的元素相同, 首先判断的是hash值, 后面再调用equals()方法. 如果hash值不同是直接跳过的
            e = p;
        else if (p instanceof TreeNode)//如果冲突解决方案已经变成红黑树的话, 按红黑树的策略添加结点. 
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {//解决冲突的方式仍是链表
            for (int binCount = 0; ; ++binCount) {//找到链表的末尾, 插入.
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);//插入之后要判断链表的长度, 如果到达一定的值就可能要转换为红黑树. 
                    break;
                }//在遍历的过程中仍会不停地判定当前key是否与传入的key相同, 判断的第一条件仍然是hash值. 
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
    ++modCount;//修改map的次数增加
    if (++size > threshold)//如果hashMap的容量到达了一定值就要进行扩容
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
我们可以看到每当判断 key 是否相同时，首先会判断 hash 值，如果 hash 值相同（产生了冲突），然后会判断 key 引用所指的对象是否相同，最终会通过 equals() 方法作最后的判定。

如果 key 的 hash 值不同，后面的判断将不会执行，直接认定两个对象不相同。

```java
if (p.hash == hash &&
    ((k = p.key) == key || (key != null && key.equals(k))))
    e = p;
```

# 五. 结束

讲到这里希望大家对 hashCode() 与 equals() 方法能有更深入的理解，明白背后的设计思想与原理。

我之前有一个疑问，可能大家看完这篇文章后也会有：equals() 方法平时我会用到，所以我知道它除了和 hashCode() 方法有密切联系外，还有别的用途。

但是hashCode()呢？**它除了和equals()方法有密切联系外，还有其他用途吗？**

经过在互联网上一番搜寻，我目前给出的答案是没有。

也就是说 hashCode() 仅在散列表中才有用，在其它情况下没用。

当然如果这个答案不正确，或者你还有别的思考，欢迎留言与我交流~