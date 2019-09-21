---
layout: post
category: Java
title: Java 集合框架全整理
tagline: by cxuan
tags:
    - Java
---

本篇文章的形式结构：

* 以一个总图开头，了解Java集合框架都包括哪些`主要`部件；
* 分别对各个部件进行大致的描述，描述其主要特征；
* 以总结的形式结尾，并给出各个部件的优劣性对比表格；
* 部分关于集合框架的面试题。

<!--more-->

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/collection/01.png)

集合在我们的日常开发中所使用的次数简直太多了，你已经把它们都用的熟透了，但是作为一名合格的程序员，你不仅要了解它的基本用法，你还要了解它的源码；存在即合理，你还要了解它是如何设计和实现的，你还要了解它的衍生过程。

这篇博客就来详细介绍一下Collection这个庞大集合框架的家族体系和成员，让你了解它的设计与实现。

**是时候祭出这张神图了**

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/collection/02.png)

首先来介绍的就是列表爷爷辈儿的接口- **Iterator**

## 01、Iterable 接口

**JavaDoc 解释：实现此接口允许对象成为 for-each 循环的目标**

也就是说，实现了此接口，就能使用 for-each 进行循环遍历，for-each 是增强型的 for 循环，是 java 提供的一种语法糖，它的基本遍历方式如下：

```java
List<Object> list = new ArrayList();
for (Object obj: list){}
```

除了实现此接口的对象外，数组也可以用 for-each 循环遍历，如下：

```java
Object[] list = new Object[10];
for (Object obj: list){}
```

那么，为什么实现了此接口的对象可以进行 for-each 遍历呢？上述的循环遍历经过反编译后如下：

```java
// 数组
Object[] list = new Object[10];
Object[] var14 = list;
int var15 = list.length;
    
for (int var16 = 0; var16 < var15; ++var16) {
   Object var10000 = var14[var16];
}
```

```java
// 对象
List<Object> list = new ArrayList();

Object var15;
for (Iterator var14 = list.iterator(); var14.hasNext(); var15 = var14.next()) {
    ;
}
```

>也就是说，for-each 循环经过反编译后，会自动创建 iterator 来实现对数组和对象的循环遍历。

**其他遍历方式**

jdk1.8之前`Iterator`只有iterator一个方法，就是

```java
Iterator<T> iterator();
```

实现次接口的方法能够创建一个轻量级的迭代器，用于安全的遍历元素，移除元素，添加元素。

>为什么说是安全的遍历元素，移除元素，添加元素？请参考[for 、foreach 、iterator 三种遍历方式的比较](https://www.cnblogs.com/cxuanBlog/p/10927538.html)

也可以使用迭代器的方式进行遍历

```java
for(Iterator it = coll.iterator(); it.hasNext(); ){
    System.out.println(it.next());
}
```

本篇先不探讨相关 jdk1.8 的新特性

## 02、Collection 接口

Collection 位于继承体系的顶级接口，它的上面只有 Iterable 接口，但是 Iterable 和集合的继承体系并无直接关系，我们一说集合框架的顶级接口其实就是指的是 Collection 接口，一说映射的顶级接口就指的是 Map 接口，而Iterable 接口更多扮演辅助的作用。

Collection 接口更多的是制定标准的作用。它主要指定的标准有

* Collection 中的一些集合允许重复元素，而其他的则不允许。一些集合有序而一些集合无序。JDK没有直接提供这个接口的实现，它提供了更多实现特性的接口，比如 List、Set，**不包括 Map，因为 Map 不是Collection 的实现**。

* 无序集合(包含重复元素)应直接实现这个接口
* 一般 Collection 的实现类应该提供两个标准的构造器，一个无参构造器，用于创建一个空集合；和一个持有单个 Collection 类型参数的构造器。例如

```java
// ArrayList.java
public ArrayList() {...}
public ArrayList(Collection<? extends E> c) {...}

// LinkedList.java
public LinkedList() {}
public LinkedList(Collection<? extends E> c) {...}

// LinkedHashSet.java
public LinkedHashSet() {...}
public LinkedHashSet(Collection<? extends E> c) {...}

// HashSet.java
public HashSet() {...}
public HashSet(Collection<? extends E> c) {...}

// TreeSet.java
public TreeSet() {...}
public TreeSet(Collection<? extends E> c) {...}

```

>个人理解添加 Collection 类型的构造函数其实就是为了集合的复制和集合的相互转化。

* Collection 接口包含一些改变集合结构的"破坏性"方法，如果实现的集合不支持此种方法，请抛出**UnsupportedOperationException 异常**。
* 一些collection的实现对元素有一些限制。例如，一些实现类禁止空元素，一些则在元素类型上有一些限制。
  试图添加不合格的元素会引发未经检查的异常。特别是空指针异常和类型转换异常。尝试查询不合格元素的
  存在可能会抛出异常，或者可能返回 false。一些实现将展现前者的行为，一些实现将展现后者的行为。
  更进一步来说，尝试将一个不符合条件的元素进行操作，不会使操作完成，将不合格的元素插入集合中可能
  会导致错误，有一些例外可能会取得成功，这取决于实现类。
* 对于线程安全性来讲，Collection 没有提供线程安全的方法，这完全交由子类自己去实现。
* 对集合执行递归遍历的某些集合操作可能会失败，并且集合直接间接包含自身的自引用实例会出现异常。
  这包括 clone(), equals(), hashCode() 和 toString() 实现可以可选地处理自引用场景，但是大多数当前实现不这样做。

### List 接口

List 也被称为列表或者有序序列，它继承了 Collection 接口，提供了很多与 Collection 相同的方法，同时也是ArrayList、LinkedList 等的父类。List 定义了一些列表的标准，它的具体特性如下：

* 使用该接口可以有序的控制每个元素的插入次序，使用者也可以通过索引访问元素，并寻找list中的元素。
* 与 set 不同，list 允许重复元素。list 列表中的元素保证插入的次序是因为其存储在 list 中的元素都满足 **e1.equals(e2)**，并且允许多个空元素。
* List除了使用 Iterator 作为迭代器之外，还提供了一种特殊的迭代器 `ListIterator `，是 List 接口所独有的。ListIterator 除了允许正常的操作外，增加了元素的插入和替换，还允许双向访问，提供了一种方法来获得从列表中的指定位置开始的序列迭代器

```java
ListIterator<E> listIterator();
// 从指定位置处开始
ListIterator<E> listIterator(int index);
```

* List 接口提供了两种方法寻找指定的对象，从性能的角度来看，应谨慎使用这些方法。它们将执行昂贵的线性搜索

```java
int indexOf(Object o);
int lastIndexOf(Object o);
```

* 虽然列表允许将自己包含为元素，但建议及其谨慎使用，equals 和 hashCode 方法不再这样的列表中很好的定义。
* 某些列表的实现对它们可能包含的元素有些限制，例如，一些实现允许空元素，一些实现对他们的元素有严格的类型限制。试图添加不合规定的元素会抛出未经检查的异常，比如 NullPointerException 和ClassCastException。

### Set 接口

Set 接口位于与 List 接口同级的层次上，它同时也继承了 Collection 接口。

* Set属于集合框架的一个子类，它不允许重复元素。也就是说，它包含在内的每两个元素的比较都不能满足e1.equals(e2)，所以最多只允许一个空元素。
* Set接口提供了额外的规定，超出了从 Collection 接口继承的那些。它对 add,equals,hashCode 方法提供了额外的标准。其他从 Collection 继承下来的方法在这仍然用起来很方便。
* 对于构造函数的约定，也就能理解，所有的构造函数必须构建一个不包含重复元素的集合。
* 注意：如果将可变元素用于 set 对象的集合，则必须非常小心。
* 一些set的实现对元素有严格的控制。例如，一些实现禁止空元素，一些实现对元素类型有严格对限制。尝试添加一些不合法的元素会抛出未经检查的异常。特别是 NullPointerException 或者 ClassCastException 。尝试查询不合法的元素也会抛出异常，或者可能仅仅返回 false。一些将展示前者的行为一些将展示后者的行为。大致上来说，尝试对不合格的元素进行操作，其完成的操作不会导致将不合格的元素插入到集合中。在实现中可以选择是当插入不合法元素时抛出异常还是仅仅只返回 false。

### Queue 接口

Queue(队列) 是和 List、Set 接口并列的 Collection 的三大接口之一。Queue 的设计用来在处理之前保持元素的访问次序。除了 Collection 基础的操作之外，队列提供了额外的插入，读取，检查操作。这些当中每一个方法都会有两种形式：如果失败就抛出异常，其他方式返回特殊的值(null 或者 false)，后者组成的插入操作被设计成用来严格控制队列容量的实现；在大部分的实现中，插入操作不能直接失败。

> Queue 接口提供了几种功能相似但细节不同的方法：
>
> add & offer() , remove() & poll(), element() & peek() 
>
> 前者失败都会抛出异常，后者会返回特殊的值。

* 队列通常（但不一定）以FIFO（先进先出）方式对元素进行排序。PriorityQueues 就是一个特例。它的元素的顺序是遵从提供的比较器，或者元素的自然排序，以及对元素进行排序的 LIFO 队列（或堆栈）（后进先出）不论使用顺序如何，调用 remove() 或者 poll() 都会移除队列的头元素。在 FIFO 队列中，所有新添加的元素都会插入到队列的末尾。
* Offer 方法会在允许的情况下插入一个元素，否则返回 false。这与 Collection.add() 方法不同，后者只能通过抛出未经检查的异常来添加元素。offer 方法被设计成，添加失败是一个正常的情况，而不是通过异常的形式出现。
* Remove() 和 poll() 方法移除并返回队列的头元素。remove() 和 poll() 方法不同的地方在于： remove() 方法在队列为 null 时抛出异常而 poll() 方法则返回 null。
* Element() 方法和 peek() 方法返回，但不移除队列的头元素。
* Queue 的实现不允许插入 null 元素，即使一些实现像是 LinkedList，没有限制插入空元素。即使实现允许，null 元素也不应该插入到队列中，因为 null 也被 poll 方法用作特殊的返回值。以指示队列包含输任何的返回值。

### SortedSet 接口

SortedSet 接口直接继承于 Set 接口，提供了一个进一步对元素进行排序的 set。使用 Comparable 对元素进行自然排序或者使用 Comparator 在创建时对元素提供定制的排序规则。set 的迭代器将按升序元素顺序遍历集合。

* 所有插入到 SortedSet 中的元素都必须实现 Comparable 接口(或者接口定制的构造器)。进一步来说，所有内部的元素都必须相互比较：e1.compareTo(e2) 或者 comparator.compare(e1, e2) 并且不会抛出ClassCastException。违反此限制将导致方法抛出 ClassCastException。
* 请注意：如果排序集要正确实现 Set 接口，则排序集维护的排序必须与equals一致。(查阅 Comparable 接口或者 Comparator 接口)。这是因为 Set 接口是根据 equals 操作定义的。但是有序集合使用 compareTo 执行所有元素的比较。因此从排序集的角度看，这种方法认为相等的两个元素是相等的。即使排序与 equals 不一致，排序集的行为也是正常运行的。它只是不遵守一般的Set接口定义。
* 所有 SortedSet 实现类都应该提供四个标准的构造器：
  * 一个 void（无参数）构造函数，它根据元素的自然顺序创建一个空的有序集。
  * 一个创建了单个 Comparator 类型参数的构造函数，它创建一个根据指定比较器排序的空排序集
  * 一个创建了单个 Comparator 类型参数的构造函数，它创建一个新的有序集合，其元素与其参数相同，并根据元素的自然顺序进行排序
  * 一个创建了单个 SortedSet 类型参数的构造器，它创建了一个新的有序集，其元素和输入有序集的顺序相同。由于接口不能包含构造函数，因此无法强制执行。

### AbstractCollection 抽象类

AbstractCollection 这个接口是 Collection 接口的一个核心实现，采用抽象方法提供部分代码的实现，用来简化实现此接口所需要的工作量，此接口提供了一些对于实现类的说明，并提供了一系列标准。

- 为了实现不可修改的 collection，程序员应该继承这个类并提供 iterator 和 size 方法(迭代器需要实现hasNext() 和 next() 方法)
- 为了实现可修改的 collection，程序员需要额外重写类的 add 方法，iterator 方法返回的 Iterator 迭代器也必须实现 remove 方法

### AbstractList 抽象类

AbstractList 接口位于中间层，向下是具体的实现类，向上是定义的抽象标准，AbstractList 继承了**AbstractCollection** 并实现了**List **接口，具体继承体系见下图

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/collection/03.png)



​																	AbstractCollection 继承体系

此接口也是 List 继承类层次的核心接口，以求最大限度的减少实现此接口的工作量，由顺序访问数据存储(数组)支持。对于序列访问的数据(像是链表)，应该优先使用 AbstractSequentialList。

- 为了实现不可修改的 list，程序员仅需要扩展这个类，并提供 get 和 list.size() 的实现就可以了。
- 如果要实现可修改的 list，程序员必须额外重写 set(int,Object) set(int,E) 方法(否则会抛出UnsupportedOperationException 的异常)，如果 list 是可变大小的，程序员必须额外重写 add(int,Object) , add(int, E) and remove(int) 方法

#### ArrayList 类

ArrayList 是实现了 List 接口的可扩容数组(动态数组)，它的内部是基于数组实现的。它的具体定义如下：

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {...}
```

- ArrayList 可以实现所有可选择的列表操作，允许所有的元素，包括空值。ArrayList 还提供了内部存储 list 的方法，它能够完全替代 Vector，只有一点除外，ArrayList 不是线程安全的容器。

- 每一个 ArrayList 实例都有一个容量的概念，这个数组的容量就是list用来存储元素的容量。它内部的 capacity属性能够快速增长，依照元素添加进 ArrayList 后，它总是能够自动增长。

- 注意这个实现类不是线程安全的，如果多个线程中至少有两个线程修改了 ArrayList 的结构的话那么最终ArrayList 中元素的个数和值可能就会发生变化。

- 如果不存在此对象，则应使用 Collections.synchronizedList "包装"该对象，最好在创建时期就这么做,防止意外的不同步列表访问。

- ```java
  List list = Collections.synchronizedList(new ArrayList(...))
  ```

- Iterator 和 listIterator 都会返回 iterators，并且都是支持 fail-fast 机制的。如果创建后再进行结构化修改的话，除了通过迭代器自己以外的任何方式，都会抛出 ConcurrentModificationException。因此，面对并发修改的情况，iterator 能够快速触发 fail-fast 机制，而不是冒着潜在的风险。

  更多关于 ArrayList 的用法，请参考[ArrayList相关方法介绍及源码分析](https://www.cnblogs.com/cxuanBlog/p/10949552.html)

#### Vector 类

Vector 类实现了可增长的对象数组。像数组一样，它包含可以使用整数索引访问元素。但是，Vector 的大小可以根据需要增大或减小，以便在创建 Vector 后添加和删除。

- 每一个 vector 都试图去维护 capacity 和 capacityIncrement 来优化内部存储。capacity 总是要保持和 vector一样的大小。随着新元素的增加，它的容量也在一直变大。vector 的存储以 capacityIncrement 的大小增加。应用程序可以在插入大量元素之前增加 vector 的容量，这就回减少重新分配的次数，降低重新分配带来的影响。
- 这个类 iterator() 方法和 listIterator() 方法返回的 iterators 是有快速失败机制的：如果 vector 在创建了 iterator 之后经过了结构化的修改，除了调用迭代器内部的 remove 或者 add 方法之外的其他方法，都会抛出ConcurrentModificationException
- elements 方法返回的 Enumeration 则没有 fail-fast 机制。

#### Stack 堆栈类

Stack类代表了后进先出 (LIFO) 对象的堆栈。它继承了 Vector 类，并且有五个操作允许 vector 像 stack 一样看待。提供了通常用的 push 和 pop 操作，以及在栈顶的 peek 方法，测试 stack 是否为空的 empty 方法，和一个寻找与栈顶距离的 search 方法。

第一次创建栈，不包含任何元素。一个更完善，可靠性更强的 LIFO 栈操作由 Deque 接口和他的实现提供，应该优先使用这个类

```java
Deque<Integer> stack = new ArrayDeque<Integer>()
```

### AbstractSet 抽象类

继承体系和 AbstractList 基本相同，除了 AbstractList 实现了 List 接口，而 AbstractSet 实现了 Set 接口。

- AbstractSet 是 Set 接口的骨干实现，以求最大限度的减少实现此接口的工作量。
- 通过扩展此类来实现集合的过程与通过扩展 AbstractCollection 实现集合的过程相同。除了这个类的子类中的所有方法和构造函数必须遵守 Set 接口强加的附加约束（例如，add 方法不允许将对象的多个实例添加到集合中）
- 注意这个类没有重写任何 AbstractCollection 的实现，它仅仅增加了 equals 和 hashCode 实现。

#### HashSet 类

HashSet 类是 Set 接口的实现类，由哈希表支持(实际上 HashSet 是 HashMap 的一个实例)。它不能保证集合的迭代顺序；特别是，它不保证顺序会随着时间的推移保持不变(也就是无序的)。这个类允许 null 元素。

* 这个类为基本操作 (add,remove,contains,size) 提供恒定的时间和性能，假设 hash 功能在桶之间正确的分散元素迭代此 set 需要的时间与 HashSet 实例大小(元素的数量)加上 HashMap 实例的 capacity (桶的数量)是成正比。因此，如果迭代性能很重要的话，那么不要设置初始 capacity 太高(或者负载因子太低)是很重要的。
* 注意这个实现不是线程安全的。如果多线程并发访问 HashSet ，并且至少一个线程修改了 set ，必须进行外部加锁。这通常通过在自然封装集合的某个对象上进行同步来实现
* 如果没有这个对象存在，这个 set 应该使用 Collections.synchronizedSet 方法重写。最好在创建时这么做，以防止对集合的意外不同步访问
* 这个实现持有 fail-fast 机制。

#### TreeSet 类

TreeSet 类是一个基于 TreeMap 的 NavigableSet 实现。这些元素使用他们的自然排序或者在创建时提供的Comparator 进行排序，具体取决于使用的构造函数。

* 此实现为基本操作 add,remove 和 contains 提供了 log(n) 的时间成本。

* 注意这个实现不是线程安全的。如果多线程并发访问 TreeSet ，并且至少一个线程修改了 set ，必须进行外部加锁。这通常通过在自然封装集合的某个对象上进行同步来实现

* 如果没有这个对象存在，这个 set 应该使用

  ```java
  SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...))
  ```

  方法重写。最好在创建时这么做，以防止对集合的意外不同步访问

* 这个实现持有 fail-fast 机制。

#### LinkedHashSet 类

LinkedHashSet 类继承于 Set，先来看一下 LinkedHashSet 的继承体系：

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/collection/04.png)



LinkedHashSet 是 Set 接口的 Hash 表和 LinkedList 的实现，具有可预测的迭代次序。这个实现不同于 HashSet的是它维护着一个贯穿所有条目的双向链表。此链表定义了元素插入集合的顺序。注意：如果元素重新插入，则插入顺序不会受到影响。

* 此实现免受 HashSet 混乱的排序。但不会导致像 TreeSet 相关的开销增加。无论原始集的实现如何，它都可用于生成与原始集合具有相同顺序的集合的副本：

```java
void foo(Set s) {
 	Set copy = new LinkedHashSet(s); 
}
```

* 这个 class 提供所有可选择的set操作，并且允许 null 元素。像 HashSet，它为基础操作(add,contains,remove) 提供了固定的基础操作。
* LinkedHashSet 有两个影响其构成的参数： 初始容量和加载因子。它们的定义与 HashSet 完全相同。但请注意：对于此类，选择过高的初始容量值的开销要比  HashSet 小，因为此类的迭代次数不受容量影响。
* 注意这个实现也不是线程安全的，如果多线程同时访问 LinkedHashSet ，并且其中一个线程修改了 set ，必须进行外部加锁。这通常通过在自然封装集合的某个对象上进行同步来实现
* 如果没有这样的对象存在，这个 set 应该使用 Collections.synchronizedSet 方法最好在创建时就这样做，防止意外地不同步访问该集
* 该类也支持 fail-fast 机制

### AbstractQueue 抽象类

这个类是 Queue 接口的骨干实现，当实现不允许为 null 元素时，使用此类中的实现是比较合适的。add, remove, element 是基于 offer, poll 和 peek 方法的。扩展此类 Queue 实现必须最低开销的定义一个方法 Queue.offer，该方法不允许插入 null 元素，以及方法 peek，poll，size，iterator。通常，还会覆盖其他方法。如果无法满足这些要求，请考虑用继承替代。

#### PriorityQueue 类

PriorityQueue 是 AbstractQueue 的实现类，优先级队列的元素根据自然排序或者通过在构造函数时期提供Comparator 来排序，具体根据构造器判断。一个优先级队列不允许null元素依赖于自然排序的优先级队列也不允许插入不可比较的对象（这样做可能导致 ClassCastException ）。

* 队列的头在某种意义上是指定顺序的最后一个元素。队列检索操作 poll, remove, peek 和 element 访问队列头部元素。
* 优先级队列是无限制的，但具有内部 capacity，用于控制用于在队列中存储元素的数组大小。它总是至少像queue 的容量一样大。作为新添加进来的元素，它的容量会自动增长。
* 该类以及迭代器实现了 Collection、Iterator 接口的所有可选方法。这个迭代器提供了 iterator() 方法不能保证以任何特定顺序遍历优先级队列的元素。如果你需要有序遍历，考虑使用 Arrays.sort(pq.toArray())。
* 注意这个实现不是线程安全的，多线程不应该并发访问 PriorityQueue 实例如果有某个线程修改了队列的话，使用线程安全的类 PriorityBlockingQueue。

### AbstractSequentialList 抽象类

AbstractSequentialList 是 List 一系列子类接口的核心接口，以求最大限度的减少实现此接口的工作量，由顺序访问数据存储(例如链接链表)支持。对于随机访问的数据(像是数组)，应该优先使用 AbstractList。

* 在这个概念上，这个接口可以说是与 AbstractList 类相反的，它实现了随机访问方法，提供了 get(int index), set(int index,E element), add(int index,E element) 和 remove(int index) 方法在列表的迭代器上
* 要实现一个列表，程序员只需要扩展这个类并且提供 listIterator 和 size方法即可。对于不可修改的列表来说， 程序员需要实现列表迭代器的 hasNext(), next(), hasPrevious(), previous() 和 index() 方法
* 对于允许修改的 list，程序员应该额外实现 list 迭代器的set方法，对于可变大小的列表，程序员应该额外实现迭代器的r emove方法 和 add 方法

#### LinkedList 类

LinkedList 是一个双向链表，实现了所有 List 列表的可选择性操作，允许存储任何元素(包括 null 值)。它的主要特性如下：

* LinkedList 所有的操作都可以表现为双向性的，索引到链表的操作将遍历从头到尾，视哪个距离近为遍历顺序。

* 注意这个实现也不是线程安全的，如果多个线程并发访问链表，并且至少其中的一个线程修改了链表的结构，那么这个链表必须进行外部加锁。(结构化的操作指的是任何添加或者删除至少一个元素的操作，仅仅对已有元素的值进行修改不是结构化的操作)。

* 如果没有这样的链表存在，这个链表应该用 Collections.synchronizedList 方法进行包装，最好在创建时间就这样做，避免意外的非同步对链表的访问

* ```java
  List list = Collections.synchronizedList(new LinkedList(...))
  ```

* Iterator 和 listIterator 都会返回 iterators，并且都是支持 fail-fast 机制的。如果创建后再进行结构化修改的话，除了通过迭代器自己以外的任何方式，都会抛出 ConcurrentModificationException 。因此，面对并发修改的情况，iterator 能够快速触发 fail-fast 机制，而不是冒着潜在的风险。

  更多关于 LinkedList的用法，请参考

  [LinkedList 基本示例及源码解析](https://www.cnblogs.com/cxuanBlog/p/10952578.html)

## 03、Map 接口

Map 是一个支持 key-value 存储的对象，Map 不能包含重复的 key，每个键最多映射一个值。这个接口代替了Dictionary 类，Dictionary 是一个抽象类而不是接口。

- Map 接口提供了三个集合的片段，它允许将 map 的内容视为一组键，值的集合和一组 key-value 映射。map的顺序定义为 map 映射集合上的迭代器返回其元素的顺序。一些 map 实现，像是 TreeMap 类，保证了 map的次数；其他的实现，像是 HashMap，则没有保证。
- 一般用途的 map 实现类应该提供两个标准的构造器：一个空 map 创建一个 void (无参数)构造器。和一个只有单个 Map 类型参数的构造器。后者的构造器允许用户复制任意 map，生成所需类的有效映射。无法强制执行此建议(因为接口不能包含构造器)但 JDK 中的所有通用映射实现都符合要求。
- 一些map实现包含在内的对键和值有限制例如，一些实现禁止 null keys 和 values，一些实现对 keys 的类型有限制。尝试插入不合格的键或值会引发未经检查的异常，比如 NullPointerException 或者ClassCastException 尝试查询不合格的 key 或 value 也可能抛出异常，或者可能返回 false；一些实现允许前者的行为，一些实现允许后者的行为

### AbstractMap 抽象类

这个抽象类是 Map 接口的骨干实现，以求最大化的减少实现类的工作量。

- 为了实现不可修改的 map，程序员仅需要继承这个类并且提供 entrySet 方法的实现，它将会返回一组 map 映射的某一段。通常，返回的集合将在 AbstractSet 之上实现。这个 set 不应该支持 add 或者 remove 方法，并且它的迭代器也不支持 remove 方法。
- 为了实现可修改的 map，程序员必须额外重写这个类的 put 方法(否则就会抛出UnsupportedOperationException)，并且 entrySet.iterator() 返回的 iterator 必须实现 remove() 方法。
- 程序员应该提供一个无返回值(无参数)的 map 构造器，

#### HashMap 类

哈希表基于 Map 接口的实现，这个实现提供可选择的 map，并且允许空 value 值和空 key，可以认为 HashMap是 Hashtable 的增强版，可以认为 HashMap 不是可同步的(非线程安全)和允许空值外，可以认为 HashMap 顺序性，也就是说，它不能保证其内部的顺序一成不变

* 这个实现为基本操作 (get、set) 提供了固定开销的性能，假设哈希函数将元素均匀的分散在各个桶中，遍历结束这个 Collection 的视图需要恒定的时间，包括 HashMap 实例(桶的数量)加上( key-value 映射的数量)。因此，如果遍历元素是很重要的话，不要去把初始容量设置的太高(或者负载因子太低)。

* 一个 HashMap 实例有两个参数扮演着重要的角色，初始容量和负载因子，这个初始容量是 hash 表桶的数量，并且初始容量只是创建哈希表时的最初的容量，这个负载因子是一种衡量哈希表的填充程度，在其容量自动增加之前获取，当哈希表中存在足够数量的 entry，以至于超过了负载因子和当前容量，这个哈希表会进行重新哈希操作，内部的数据结构重新 rebuilt，这样的哈希表大约有两倍的桶数量

* 作为一般的规则，这个默认的负载因子0.75 提供了一个好的平衡点在时间和空间利用上面，这个值越高，就越会降低空间开销，但是会增加查找成本(反应在大部分 HashMap 的操作上面包括 get 、 put )，预期数量map 中的 entry 链和加载因子当设置初始容量的时候应该被考虑进去，以便降低最小数量的重新 hash 操作。如果初始容量要比(最大数量的 entry 链/负载因子)大则不用重新哈希，将一直能够操作。

* 注意这个实现类 (HashMap) 不是同步的(非线程安全的)，如果多个线程同时影响到了 hash map，并且至少一个线程修改了 map 的结构，那么必须借助外力的同步操作。( adds 或者 deletes 一个或多个映射是一个结构化的修改操作。仅仅改变 key 的 value 值不是一个结构化的修改)。如果不存在此类对象，这个 map 应该被Collections.synchronizedMap 封装，最好是在创建的时候就封装好，去阻止偶然的对 map 的非同步访问 

* ```java
  Map m = Collections.synchronizedMap(new HashMap());
  ```

* 支持 fail-fast 快速失败机制

#### TreeMap 类

一个基于 NavigableMap 实现的红黑树。这个 map 根据 key 自然排序存储，或者通过 Comparator 基于NavigableMap 实现的红黑树。这个 map 根据 key 自然排序存储，或者通过 Comparator 进行定制排序

* 这个实现为 containsKey, get, put 和 remove 方法提供了 log(n) 的操作。

* 注意这个实现不是线程安全的。如果多线程并发访问 TreeMap，并且至少一个线程修改了 map，必须进行外部加锁。这通常通过在自然封装集合的某个对象上进行同步来实现

* 如果没有这个对象存在，这个 set 应该使用

  ```java
  SortedMap m = Collections.synchronizedSortedMap(new TreeMap(...))
  ```

  方法重写。最好在创建时这么做，以防止对集合的意外不同步访问

* 这个实现持有 fail-fast 机制。

* 此类中的方法返回所有的 Map.Entry 对及其试图表示生成时映射的快照。他们不支持 Entry.setValue 方法

#### LinkedHashMap 类

LinkedHashMap 是 map 接口的哈希表和链表的实现，具有可预测的迭代顺序。这个实现与 HashMap 不同之处在于它维护了一个贯穿其所有条目的双向链表。这个链表定义了遍历顺序，通常是插入 map 中的顺序。注意重新插入并不会影响其插入的顺序。如果在 containsKey(k) 时调用 m.put(k,v)，则将 key 插入到 map 中会在调用之前返回 true。

* 此实现使其客户端免受 HashMap, Hashtable 未指定的，混乱的排序。而不会导致像 TreeMap 一样的性能开销，无论原始 map 的实现如何，它都可用于生成与原始 map 具有相同顺序的map副本。

  ```java
  void foo(Map m) {
    Map copy = new LinkedHashMap(m); 
  }
  ```

* 它提供一个特殊的 LinkedHashMap(int,float,boolean) 构造器来创建 LinkedHashMap，其遍历顺序是其最后一次访问的顺序。这种类型的map很适合构建LRU缓存[图解集合6：LinkedHashMap](https://www.cnblogs.com/xrq730/p/5052323.html)

* 可以重写 removeEldestEntry(Map.Entry) 方法，以便在将新映射添加到 map 时强制删除过期映射的策略。

* 这个类提供了所有可选择的 map 操作，并且允许null元素。像 HashMap，它为基础操作(add,contains,remove)提供了恒定的时间消耗，假定散列函数在桶之间正确分配了元素。由于维护链表的额外开销，性能可能会低于HashMap，有一条除外：遍历 LinkedHashMap 中的 collection-views 需要与map.size 成正比，无论其容量如何。HashMap 的迭代看起来开销更大，因为还要求时间与其容量成正比。

* LinkedHashMap 有两个因素影响了它的构成：初始容量和加载因子。他们也定义在 HashMap。注意，对于此类，选择过高的容量的初始值代价小于 HashMap，因为此类的迭代次数不受容量的影响。

* 注意这个实现不是线程安全的。如果多线程并发访问 LinkedHashMap，并且至少一个线程修改了 map，必须进行外部加锁。这通常通过在自然封装集合的某个对象上进行同步来实现

  如果没有这个对象存在，这个set应该使用

  ```java
  Map m = Collections.synchronizedMap(new LinkedHashMap(...))
  ```

  方法重写。最好在创建时这么做，以防止对集合的意外不同步访问

* 这个实现持有 fail-fast 机制。

#### Hashtable 类

Hashtable类实现了一个哈希表，能够将键映射到值。任何非空对象都可以用作键或值。

* 为了从哈希表中成功存储和检索对象，这个对象的 key 必须实现 hashCode 方法和 equals 方法。
* Hashtable 的实例有两个参数影响它的构成：初始容量和加载因子。容量就是哈希表中桶的数量，初始容量就是哈希表创建出来的容量。注意这个哈希表是开放的：为了避免哈希冲突，一个桶存储了多个 entries 必须按顺序搜索。负载因子是衡量哈希表在其容量自动增加之前可以变得多大。初始容量和负载系数参数仅仅是实现的提示。 关于何时以及是否调用rehash方法的确切细节是依赖于实现的
* 大致上来说，默认的负载因子0.75 提供了一个空间和时间开销的平衡。此值太高会减少空间开销但是会增加查找的时间成本，这会反应在get、put操作中。
* 初始容量控制了浪费空间与 rehash 操作需求之间的权衡，这是非常耗时的。如果 初始容量 > entries链条数目 / load factor，则 rehash 操作永远不会发生。然而，设置初始容量太高会浪费空间。
* 下面例子展示了一个基础用法

```java
// 放置元素
Hashtable<String, Integer> numbers = new Hashtable<String, Integer>();
numbers.put("one", 1);
numbers.put("two", 2);
numbers.put("three", 3);

// 取出元素
Integer n = numbers.get("two");
if(n != null){
  System.out.println("two = " + n);
}
```

* 此实现类支持 fail-fast 机制
* 与新的集合实现不同，Hashtable 是线程安全的。如果不需要线程安全的容器，推荐使用 HashMap，如果需要多线程高并发，推荐使用 ConcurrentHashMap。

#### IdentityHashMap 类

IdentityHashMap 是比较小众的Map实现了，它使用哈希表实现Map接口，在比较键和值时使用引用相等性替换对象相等性。换句话说，在IdentityHashMap中两个key，k1和k2 当且仅当 k1 == k2时被认为是相等的。(在正常的Map实现中(类似HashMap)，两个键k1 和 k2 当且仅当 k1 == null ? k2 == null : k1.equals(k2)时 被认为时相等的

* 这个类不是一个通用的Map实现！虽然这个类实现了Map接口，但它故意违反了Map的约定，该约定要求在比较对象时使用equals方法，此类仅适用于需要引用相等语义的极少数情况。
* 此类提供所有可选择的map操作，并且允许null值和null key。这个类不保证map的顺序，特别的，它不保证顺序会随着时间的推移保持不变。
* 同 HashMap，IdentityHashMap 也是无序的，并且该类不是线程安全的，如果要使之线程安全，可以调用Collections.synchronizedMap(new IdentityHashMap(...))方法来实现。
* 支持 fail-fast 机制

#### WeakHashMap 类

WeakHashMap 类基于哈希表的 Map 基础实现，带有弱键。WeakHashMap 中的 entry 当不再使用时还会自动移除。更准确的说，给定 key 的映射的存在将不会阻止 key 被垃圾收集器丢弃。

* 基于 map 接口，是一种弱键相连，WeakHashMap 里面的键会自动回收
* 支持 null值 和 null 键。和 HashMap 有些相似
* fast-fail 机制
* 不允许重复
* WeakHashMap 经常用作缓存关于 HashMap 的具体存储结构

相关更详细的 WeakHashMap 解析，见下面这篇文章 http://ifeve.com/weakhashmap/

### SortedMap 抽象类

SortedMap 是一个提供了在key上全部排序的map。这个 map 根据 key 的 Comparable 自然排序，提供了在创建时期的 map 排序。遍历有序映射的集合片段(由

entrySet、keySet 和 values 方法返回)时能够反应此顺序的影响。

* 所有插入到 sorted map 的 key 都必须实现 Comparable 接口(或者接受定制的 Comparator 接口)。进一步来讲，所有的 key 都必须相互比较形如 k1.compareTo(k2) 或者 comparator.compare(k1,k2) 对其中的任何 key都不能抛出 ClassCastException。试图违反此限制将导致违规方法或构造函数抛出 ClassCastException
* 所有一般情况下的 sortedmap 实现类提供四个标准的构造器。
  * 一个无返回值(无参数)的构造器，它根据 key 的自然排序创建类一个空 sorted map。
  * 一个有单个 Comparator 类型参数的构造函数，它根据指定比较器创建了一个空的 sorted map 排序。
  * 一个有单个 map 类型参数的构造函数，它创建了一个相同 key-value 映射参数的 map，根据 key 来自然排序。
  * 一个有单个 SortedMap 类型的构造器，它创建了一个新的有序映射，其具有相同的键 - 值映射和与输入有序映射相同的顺序。

## 04、Comparable && Comparator

具体描述请详见[Comparable 和 Comparator的理解](https://www.cnblogs.com/cxuanBlog/p/10927495.html)

## 05、Collections 类

Collections 不属于Java框架继承树上的内容，它属于单独的分支，Collections 是一个包装类，它的作用就是为集合框架提供某些功能实现，此类只包括静态方法操作或者返回 collections。

#### 同步包装

同步包装器将自动同步（线程安全性）添加到任意集合。 六个核心集合接口（ Collection，Set，List，Map，SortedSet 和 SortedMap）中的每一个都有一个静态工厂方法。

```java

public static  Collection synchronizedCollection(Collection c);
public static  Set synchronizedSet(Set s);
public static  List synchronizedList(List list);
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);
public static  SortedSet synchronizedSortedSet(SortedSet s);
public static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m);

```

#### 不可修改的包装

不可修改的包装器通过拦截修改集合的操作并抛出 UnsupportedOperationException，主要用在下面两个情景：

* 构建集合后使其不可变。在这种情况下，最好不要去获取返回 collection 的引用，这样有利于保证不变性
* 允许某些客户端以只读方式访问你的数据结构。 你保留对返回的 collection 的引用，但分发对包装器的引用。 通过这种方式，客户可以查看但不能修改，同时保持完全访问权限。

这些方法是：

```java

public static  Collection unmodifiableCollection(Collection<? extends T> c);
public static  Set unmodifiableSet(Set<? extends T> s);
public static  List unmodifiableList(List<? extends T> list);
public static <K,V> Map<K, V> unmodifiableMap(Map<? extends K, ? extends V> m);
public static  SortedSet unmodifiableSortedSet(SortedSet<? extends T> s);
public static <K,V> SortedMap<K, V> unmodifiableSortedMap(SortedMap<K, ? extends V> m);

```

#### 线程安全的Collections

Java1.5 并发包 (java.util.concurrent) 提供了线程安全的 collections 允许遍历的时候进行修改，通过设计 iterator为 fail-fast 并抛出 ConcurrentModificationException。一些实现类是`CopyOnWriteArrayList`，`ConcurrentHashMap`，`CopyOnWriteArraySet`

#### Collections 算法

此类包含用于集合框架算法的方法，例如二进制搜索，排序，重排，反向等。

有兴趣的读者可以阅读https://docs.oracle.com/javase/tutorial/collections/algorithms/ 本篇暂不探讨算法问题。

## 06、Arrays 类

这个类包含各种各样的操作数组的方法(例如排序和查找)。这个类也包含一个静态工厂把数组当作列表来对待。如果指定数组的引用为 null，这个类中的方法都会抛出 NullPointerException，除非有另外说明。

详细的用法请参考[JAVA基础——Arrays工具类十大常用方法](https://www.cnblogs.com/hysum/p/7094232.html)



## 07、集合实现类特征图

下图汇总了部分集合框架的主要实现类的特征图，让你能有清晰明了看出每个实现类之间的差异性

| 集合                 | 排序 | 随机访问 | key-value存储 | 重复元素 | 空元素 | 线程安全 |
| -------------------- | ---- | -------- | ------------- | -------- | ------ | -------- |
| ArrayList            | Y    | Y        | N             | Y        | Y      | N        |
| LinkedList           | Y    | N        | N             | Y        | Y      | N        |
| HashSet              | N    | N        | N             | N        | Y      | N        |
| TreeSet              | Y    | N        | N             | N        | N      | N        |
| HashMap              | N    | Y        | Y             | N        | Y      | N        |
| TreeMap              | Y    | Y        | Y             | N        | N      | N        |
| Vector               | Y    | Y        | N             | Y        | Y      | Y        |
| Hashtable            | N    | Y        | Y             | N        | N      | Y        |
| ConcurrentHashMap    | N    | Y        | Y             | N        | N      | Y        |
| Stack                | Y    | N        | N             | Y        | Y      | Y        |
| CopyOnWriteArrayList | Y    | Y        | N             | Y        | Y      | Y        |



## 08、相关集合的面试题

碍于篇幅的原因，这里就不贴出来具体的面试题和答案了，我提供了一些面试题的相关文章，请你参考

经典Java集合面试题 https://blog.csdn.net/u010775025/article/details/79315361

关于集合类的使用 https://github.com/hollischuang/toBeTopJavaer

Java集合必会14问（精选面试题整理）https://www.jianshu.com/p/939b8a672070 



相关阅读：

[Comparable 和 Comparator的理解](https://www.cnblogs.com/cxuanBlog/p/10927495.html)

[for 、foreach 、iterator 三种遍历方式的比较](https://www.cnblogs.com/cxuanBlog/p/10927538.html)

[ArrayList相关方法介绍及源码分析](https://www.cnblogs.com/cxuanBlog/p/10949552.html)

[LinkedList 基本示例及源码解析](https://www.cnblogs.com/cxuanBlog/p/10952578.html)





文章参考：

https://www.journaldev.com/1260/collections-in-java-tutorial#collections-class

https://blog.csdn.net/markzy/article/details/79789655

https://www.cnblogs.com/cxuanBlog/p/10927538.html

https://docs.oracle.com/javase/tutorial/collections/algorithms/ 