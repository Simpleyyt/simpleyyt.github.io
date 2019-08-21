---
layout: post
title: 集合系列 - 初探java集合框架图
tagline: by 炸鸡可乐
categories: Java
tags: 
  - Java
---

实际开发中，经常用到java的集合框架，比如ArrayList、LinkedList、HashMap、LinkedHashMap，几乎经常接触到，虽然用的多，但是对集合的整体框架，基础知识还是不够系统，今天想和大家一起来梳理一下！

<!--more-->

### 01、集合类简介
>  Java集合就像一种容器，可以把多个对象（实际上是对象的引用，但习惯上都称对象）“丢进”该容器中。从Java 5 增加了泛型以后，Java集合可以记住容器中对象的数据类型，使得编码更加简洁、健壮。

Java集合大致可以分为两大体系，一个是Collection，另一个是Map
* Collection ：主要由List、Set、Queue接口组成，List代表有序、重复的集合；其中Set代表无序、不可重复的集合；Java 5 又增加了Queue体系集合，代表一种队列集合实现。
* Map：则代表具有映射关系的键值对集合。

**java.util.Collection下的接口和继承类关系简易结构图：**
![](http://www.justdojava.com/assets/images/2019/java/image-jay/e5f968010ae841a090e44dee1a4ee1c9.png)

**java.util.Map下的接口和继承类关系简易结构图：**
![](http://www.justdojava.com/assets/images/2019/java/image-jay/1d13a00bfcd54c3aac0d37124c849c21.png)

其中，Java 集合框架中主要封装的是典型的数据结构和算法，如动态数组、双向链表、队列、栈、Set、Map 等。

将集合框架挖掘处理，可以分为以下几个部分
**1) 数据结构**
`List`列表、`Queue`队列、`Deque`双端队列、`Set`集合、`Map`映射
**2) 算法**
`Collections`常用算法类、`Arrays`静态数组的排序、查找算法
**3) 迭代器**
`Iterator`通用迭代器、`ListIterator`针对 `List` 特化的迭代器
**4) 比较器**
`Comparator`比较器

**以下内容，是各个数据结构的简单理论介绍！**
### 02、有序列表（List）
> List集合的特点就是存取有序，可以存储重复的元素，可以用下标进行元素的操作

List主要实现类：ArrayList、LinkedList、Vector、Stack。
#### 2.1、ArrayList
ArrayList是一个动态数组结构，支持随机存取，尾部插入删除方便，内部插入删除效率低（因为要移动数组元素）；如果内部数组容量不足则自动扩容，因此当数组很大时，效率较低。
#### 2.2、LinkedList
LinkedList是一个双向链表结构，在任意位置插入删除都很方便，但是不支持随机取值，每次都只能从一端开始遍历，直到找到查询的对象，然后返回；不过，它不像 ArrayList 那样需要进行内存拷贝，因此相对来说效率较高，但是因为存在额外的前驱和后继节点指针，因此占用的内存比 ArrayList 多一些。
#### 2.3、Vector
Vector也是一个动态数组结构，一个元老级别的类，早在jdk1.1就引入进来类，之后在jdk1.2里引进ArrayList，ArrayList大部分的方法和Vector比较相似，两者是不同的，Vector是允许同步访问的，Vector中的操作是线程安全的，但是效率低，而ArrayList所有的操作都是异步的，执行效率高，但不安全！

关于`Vector`，现在用的很少了，因为里面的`get`、`set`、`add`等方法都加了`synchronized`，所以，执行效率会比较低，如果需要在多线程中使用，可以采用下面语句创建ArrayList对象
```
List<Object> list =Collections.synchronizedList(new ArrayList<Object>());
```
也可以考虑使用复制容器 `java.util.concurrent.CopyOnWriteArrayList`进行操作，例如：
```
final CopyOnWriteArrayList<Object> cowList = new CopyOnWriteArrayList<String>(Object);
```
#### 2.4、Stack
Stack是Vector的一个子类，本质也是一个动态数组结构，不同的是，它的数据结构是先进后出，取名叫栈！

关于`Stack`，现在用的也很少，因为有个`ArrayDeque`双端队列，可以替代`Stack`所有的功能，并且执行效率比它高！
### 03、集(Set)
> Set集合的特点：元素不重复，存取无序，无下标；

Set主要实现类：HashSet、LinkedHashSet和TreeSet。
#### 3.1、HashSet
HashSet底层是基于 HashMap 的`k`实现的，元素不可重复，特性同 HashMap。
#### 3.2、LinkedHashSet
LinkedHashSet底层也是基于 LinkedHashMap 的`k`实现的，一样元素不可重复，特性同 LinkedHashMap。
#### 3.3、TreeSet
同样的，TreeSet也是基于 TreeMap 的`k`实现的，同样元素不可重复，特性同 TreeMap；

**Set集合的实现，基本都是基于Map中的键做文章，使用Map中键不能重复、无序的特性；所以，我们只需要重点关注Map的实现即可！**
### 04、队列(Queue)
> Queue是一个队列集合，队列通常是指“先进先出”（FIFO）的容器。新元素插入（offer）到队列的尾部，访问元素（poll）操作会返回队列头部的元素。通常，队列不允许随机访问队列中的元素。

Queue主要实现类：ArrayDeque、LinkedList、PriorityQueue。
#### 4.1、ArrayDeque
ArrayQueue是一个基于数组实现的双端队列，可以想象，在队列中存在两个指针，一个指向头部，一个指向尾部，因此它具有“FIFO队列”及“栈”的方法特性。

既然是双端队列，那么既可以先进先出，也可以先进后出，以下是测试例子！

**先进先出**
```
public static void main(String[] args) {
                ArrayDeque<String> queue = new ArrayDeque<>();
        //入队
        queue.offer("AAA");
        queue.offer("BBB");
        queue.offer("CCC");
        System.out.println(queue);
        //获取但不出队
        System.out.println(queue.peek());
        System.out.println(queue);
        //出队
        System.out.println(queue.poll());
        System.out.println(queue);
}
```
输出结果：
```
[AAA, BBB, CCC]
AAA
[AAA, BBB, CCC]
AAA
[BBB, CCC]
```

**先进后出**
```
public static void main(String[] args) {
                ArrayDeque<String> stack = new ArrayDeque<>();
        //压栈,此时AAA在最下,CCC在最外
        stack.push("AAA");
        stack.push("BBB");
        stack.push("CCC");
        System.out.println(stack);
        //获取最后添加的元素,但不删除
        System.out.println(stack.peek());
        System.out.println(stack);
        //弹出最后添加的元素
        System.out.println(stack.pop());
        System.out.println(stack);
}
```
输出结果：
```
[CCC, BBB, AAA]
CCC
[CCC, BBB, AAA]
CCC
[BBB, AAA]
```
#### 4.2、LinkedList
LinkedList是List接口的实现类，也是Deque的实现类，底层是一种双向链表的数据结构，在上面咱们也有所介绍，LinkedList可以根据索引来获取元素，增加或删除元素的效率较高，如果查找的话需要遍历整合集合，效率较低，LinkedList同时实现了stack、Queue、PriorityQueue的所有功能。

**例子**
```
public static void main(String[] args) {
                LinkedList<String> ll = new LinkedList<>();
        //入队
        ll.offer("AAA");
        //压栈
        ll.push("BBB");
        //双端的另一端入队
        ll.addFirst("NNN");
        ll.forEach(str -> System.out.println("遍历中:" + str));
        //获取队头
        System.out.println(ll.peekFirst());
        //获取队尾
        System.out.println(ll.peekLast());
        //弹栈
        System.out.println(ll.pop());
        System.out.println(ll);
        //双端的后端出列
        System.out.println(ll.pollLast());
        System.out.println(ll);
}
```
输出结果：
```
遍历中:NNN
遍历中:BBB
遍历中:AAA
NNN
AAA
NNN
[BBB, AAA]
AAA
[BBB]
```
#### 4.3、PriorityQueue
PriorityQueue也是一个队列的实现类，此实现类中存储的元素排列并不是按照元素添加的顺序进行排列，而是内部会按元素的大小顺序进行排列，是一种能够自动排序的队列。

**例子**
```
public static void main(String[] args) {
        PriorityQueue<Integer> queue1 = new PriorityQueue<>(10);

        System.out.println("处理前的数据");
        Random rand = new Random();
        for (int i = 0; i < 10; i++) {
                Integer num = rand.nextInt(90) + 10;
                System.out.print(num + ", ");
            queue1.offer(num); // 随机两位数
        }

        System.out.println("\n处理后的数据");
        for (int i = 0; i < 10; i++) { // 默认是自然排序 [升序]
            System.out.print(queue1.poll() + ", ");
        }
}
```
输出结果：
```
处理前的数据
36, 23, 24, 11, 12, 26, 79, 96, 14, 73, 
处理后的数据
11, 12, 14, 23, 24, 26, 36, 73, 79, 96, 
```
### 05、映射表(Map)
> Map是一个双列集合，其中保存的是键值对，键要求保持唯一性，值可以重复。

Map 主要实现类：HashMap、LinkedHashMap、TreeMap、Hashtable。
#### 5.1、HashMap
关于HashMap，相信大家都不陌生，key 不可重复，因为使用的是哈希表存储元素，所以输入的数据与输出的数据，顺序基本不一致，另外，HashMap最多只允许一条记录的 key 为 null。

* **存储数据的过程**

HashMap的数据结构是有数组+链表实现的，当进行put操作的时候，先通过K获得K的 hashCode ，然后经过扰动函数处理得到 hash 值，再通过 `(n - 1) & hash` 计算当前元素存放的数组下标位置（这里的 n 指的是数组的长度）；接着，对K进行hashCode和equals比较，如果判断是true，那么就是同一个对象，对值进行覆盖处理，如果是false，就依次存储到链表中！

在链表的存储方面，jdk1.8与jdk1.7有些区别，jdk1.8使用了红黑树结构，也就是平衡二叉树，目的是在有hash冲突的链表结构中，查询的时候，效率会更高！

* **获取数据的过程**

当通过k调用get方法来获取参数值时，原理也是一样，先计算k的hashCode值,经过扰动函数处理得到 hash 值，进一步计算获取数组下标，再通过hashCode和equals方法，依次在链表中进行搜寻，获取对应的参数值！

#### 5.2、LinkedHashMap
HashMap 的子类，内部使用链表数据结构来记录插入的顺序，使得输入的记录顺序和输出的记录顺序是相同的。这使得**LinkedHashMap与HashMap最大的不同处在于，LinkedHashMap输入的记录和输出的记录顺序是相同的！**

#### 5.3、TreeMap
能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用 Iterator 遍历时，得到的记录是排过序的；如需使用排序的映射，建议使用 TreeMap。TreeMap实际使用的比较少！
#### 5.4、Hashtable
Hashtable，允许方法同步访问，早在jdk1.1中就引入了，同时也实现了Map中的方法，也是一个元老级的类。

HashMap 是 HashTable 的轻量级实现，他们都完成了Map 接口，主要区别在于 HashMap 允许K和V为空，由于非线程安全，效率上可能高于 Hashtable。

而Hashtable，key不能为空，因为`Hashtable`把所有方法都加上`synchronized`关键字来实现线程安全，所以，效率比较低！

如果需要在多线程环境下使用HashMap，可以使用如下的同步器来实现或者使用并发工具包中的`ConcurrentHashMap`类
```
Map<String, Object> map =Collections.synchronizedMap(new HashMap<>());
```

最后，本文主要是对java集合框架理论知识，做了一个简单的介绍和梳理，如果有理解不到位的地方，欢迎直接给我们留言！
