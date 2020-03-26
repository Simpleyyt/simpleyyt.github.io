---
layout: post
categories: 集合系列
title: 集合知识全系列回顾
tags: 
  - 炸鸡可乐
---

实际开发中，经常用到的 ArrayList、LinkedList、HashMap、LinkedHashMap 等集合类，其实涵盖了很多数据结构和算法，每个类可以说都是精华，今天阿粉想和大家一起来梳理一下！

<!--more-->
### 一、摘要
在 Java 中，集合大致可以分为两大体系，一个是 **Collection**，另一个是 **Map**，都位于`java.util`包下。

* **Collection ：主要由 List、Set、Queue 接口组成，List 代表有序、重复的集合；其中 Set 代表无序、不可重复的集合；Java 5 又增加了 Queue 体系集合，代表队列集合。**
* **Map：则代表具有映射关系的键值对集合。**

很多书籍将 List、Set、Queue、Map 等能存放元素的接口体系，也归纳为**容器**，因为他们可以存放元素！

**集合**和**容器**，这两者只是在概念上定义不同，比如`ArrayList`是一个存放数组的对象，真正用起来并不会去关心这个东西到底是**集合**还是**容器**，把东西用好才是关键！

**java.util.Collection下的接口和继承类关系简易结构图：**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/66b999971340447386ff3fabc7f1deb2.jpg)

**java.util.Map下的接口和继承类关系简易结构图：**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/2c68c401bf694fb1a2275bf80c367a89.jpg)

**需要注意的是，网上有些集合架构图将 Map 画成继承自 Collection，实际上 Map 并没有继承 Collection，Map 是单独的一个集合体系，这点需要纠正一下**。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/3dfea6a29806476285f02e83f13e503c.jpg)

在 Java 集合框架中，**数据结构和算法**可以说在里面体现的淋淋尽致，这一点可以从我们之前对各个集合类的分析就可以看的出来，如动态数组、链表、红黑树、Set、Map、队列、栈、堆等，基本上只要出去面试，集合框架的话题一定不会少！

下面我们就一起来看看各大数据结构的具体实现！
### 二、List
> List 集合的特点：存取有序，可以存储重复的元素，可以用下标进行元素的操作。

**List 集合中，各个类继承结构图：**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/9f0d121b9fa84c97bc13417cfa23d339.jpg)

从图中可以看出，List 主要实现类有：ArrayList、LinkedList、Vector、Stack。

#### 2.1、ArrayList
ArrayList 是一个动态数组结构，支持随机存取，在指定的位置插入、删除效率低（因为要移动数组元素）；如果内部数组容量不足则自动扩容，**扩容系数为原来的1.5倍**，因此当数组很大时，效率较低。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/7c71ae0af79f4a6c94b2331fc73bef38.png)

当然，插入删除也不是效率非常低，在某些场景下，比如尾部插入、删除，因为不需要移动数组元素，所以效率也很高哦！

ArrayList 是一个非线程安全的类，在多线程环境下使用迭代器遍历元素时，**会报错，抛`ConcurrentModificationException`异常！**

因此，如果想要在多线程环境下使用 ArrayList，建议直接使用并发包中的`CopyOnWriteArrayList`！

#### 2.2、LinkedList
LinkedList 是一个双向链表结构，在任意位置插入、删除都很方便，但是不支持随机取值，每次都只能从一端开始遍历，直到找到查询的对象，然后返回；不过，它不像 ArrayList 那样需要进行内存拷贝，因此相对来说效率较高，但是因为存在额外的前驱和后继节点指针，因此占用的内存比 ArrayList 多一些。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/52e64a58694f48238702cdab0b66a1d7.png)

LinkedList 底层通过双向链表实现，通过`first`和`last`引用分别指向链表的第一个和最后一个元素，注意这里没有所谓的哑元（某个参数如果在子程序或函数中没有用到，那就被称为哑元），当链表为空的时候`first`和`last`都指向`null`。

#### 2.3、Vector
Vector 也是一个动态数组结构，一个元老级别的类，早在 jdk1.1 就引入进来了，之后在 jdk1.2 里引进 ArrayList，ArrayList 可以说是 Vector 的一个迷你版，ArrayList 大部分的方法和 Vector 比较相似！

两者不同的是，Vector 中的方法都加了`synchronized`，保证操作是线程安全的，但是效率低，而 ArrayList 所有的操作都是非线程安全的，执行效率高，但不安全！

对于 Vector，虽然可以在多线程环境下使用，但是在迭代遍历元素的时候依然会报错，抛`ConcurrentModificationException`异常！

在 JDK 中 Vector 已经属于过时的类，官方不建议在程序中采用，如果想要在多线程环境下使用 Vector，建议直接使用并发包中的`CopyOnWriteArrayList`！


#### 2.4、Stack
Stack 是 Vector 的一个子类，本质也是一个动态数组结构，不同的是，它的数据结构是先进后出，取名叫**栈**！

不过，关于 Java 中 Stack 类，有很多的质疑声，栈更适合用队列结构来实现，这使得 Stack 在设计上不严谨，因此，官方推荐使用 Deque 下的类来是实现栈！

#### 2.5、小结
List 接口各个实现类性能比较，如图：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/775948ebaf704723baecbc5361e2bea0.jpg)

* **ArrayList（动态数组结构），查询快（随意访问或顺序访问），增删慢，但在末尾插入删除，速度与LinkedList相差无几，但是是非线程安全的！**
* **LinkedList（双向链表结构），查询慢，增删快，也是非线程安全的！**
* **Vector（动态数组结构），因为方法加了同步锁，相比 ArrayList 执行都慢，基本不在使用，如果需要在多线程下使用，推荐使用并发容器中的`CopyOnWriteArrayList`来操作，效率高！**
* **Stack（栈结构）继承于Vector，数据是先进后出，基本不在使用，如果要实现栈，推荐使用 Deque 下的 ArrayDeque，效率比 Stack 高！**


### 三、Map
> Map是一个双列集合，其中保存的是键值对，键要求保持唯一性，值可以重复。

从上文中，我们了解到 Map 集合结构体系如下：
![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/50dbf2841194465b8cba35e3c32b8932.jpg)

从图中可以看出，Map 的实现类有 HashMap、LinkedHashMap、TreeMap、IdentityHashMap、WeakHashMap、Hashtable、Properties 等等。

#### 3.1、HashMap
> 关于 HashMap，相信大家都不陌生，是一个使用非常频繁的容器类，继承自 AbstractMap，它允许键值都放入 null 元素，但是 key 不可重复。

因为使用的是哈希表存储元素，所以输入的数据与输出的数据，顺序基本不一致，另外，HashMap 最多只允许一条记录的 key 为 null。


在 jdk1.7中，HashMap 主要是由 **数组+ 单向链表** 组成，当发生 hash 冲突的时候，就将冲突的元素放入链表中。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/75fe39153a69421abca43d484ac82153.jpg)

从 jdk1.8开始，HashMap 主要是由** 数组+单向链表+红黑树** 实现，相比 jdk1.7 而言，多了一个红黑树实现。

**当链表长度超过 8 的时候，就将链表变成红黑树**，如图所示。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/c5f44efd8cf944a89d70aa7caca835df.png)

HashMap 的实现原理，算是面试必问的一个话题，详细的实现过程，有兴趣的朋友可以阅读小编之前写的文章**《深入分析HashMap》**！

HashMap 虽然很强大，但是它是非线程安全的，也就是说，如果在多线程环境下使用，**可能因为程序自动扩容操作将单向链表转变成了循环链表，在查询遍历元素的时候，造成程序死循环！此时 CPU 直接会飙到 100%！**

如果我们想在多线程环境下使用 HashMap，其中一个推荐的解决办法就是使用 java 并发包下的 **ConcurrentHashMap** 类！

在 JDK1.7 中，**ConcurrentHashMap** 类采用了分段锁的思想，将 HashMap 进行切割，把 HashMap 中的哈希数组切分成小数组（Segment），每个小数组有 n 个 HashEntry 组成，其中 Segment 继承自`ReentrantLock（可重入锁）`，从而实现并发控制！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/8a6a46d644fd43bd98b6954e1f6b193b.jpg)

从 jdk1.8 开始，ConcurrentHashMap 类取消了 `Segment` 分段锁，采用 `CAS + synchronized`来保证并发安全，数据结构跟 jdk1.8 中 HashMap 结构保持一致，都是** 数组 + 链表**（当链表长度大于8时，链表结构转为红黑树）结构。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/535a73dadbbb4b70b3ca53278c0ae007.jpg)

 jdk1.8 中的 ConcurrentHashMap 中 `synchronized` 只锁定当前链表或红黑树的首节点，只要节点 hash 不冲突，就不会产生并发，相比 JDK1.7 的 ConcurrentHashMap 效率又提升了 N 倍！

详细的实现原理，可以参阅小编之前写的文章**《深入分析ConcurrentHashMap》**！


#### 3.2、LinkedHashMap
LinkedHashMap 继承自 HashMap，可以认为是 HashMap + LinkedList，它既使用 HashMap 操作数据结构，又使用 LinkedList 维护插入元素的先后顺序，内部采用双向链表（doubly-linked list）的形式将所有元素（ entry ）连接起来。

从名字上，就可以看出 LinkedHashMap 是一个 LinkedList 和 HashMap 的混合体，同时满足 HashMap 和 LinkedList 的某些特性，可将 LinkedHashMap 看作采用 Linkedlist 增强的HashMap。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/88563feeee954560a45fb43c331c2912.png)

双向链表头部插入的数据为链表的入口，迭代器遍历方向是从链表的头部开始到链表尾部结束，结构图如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/55969d54594d402ab84e3c0718552fd9.jpg)

这种结构除了可以保持迭代的顺序，还有一个好处：迭代 LinkedHashMap 时不需要像 HashMap 那样遍历整个 table，而只需要直接遍历 header 指向的双向链表即可，也就是说 LinkedHashMap 的迭代时间就只跟 entry 的个数相关，而跟 table 的大小无关。

LinkedHashMap 继承了 HashMap，所有大部分功能特性与 HashMap 基本相同，允许放入 key 为`null`的元素，也允许插入 value 为`null`的元素。

二者唯一的区别是 LinkedHashMap 在 HashMap 的基础上，采用双向链表（doubly-linked list）的形式将所有 entry 连接起来，这样是为保证元素的迭代顺序跟插入顺序相同。

#### 3.3、TreeMap
> TreeMap实现了 SortedMap 接口，也就是说会按照 key 的大小顺序对 Map 中的元素进行排序，key 大小的评判可以通过其本身的自然顺序（natural ordering），也可以通过构造时传入的比较器（Comparator）。

TreeMap 底层通过红黑树（Red-Black tree）实现，所以要了解 TreeMap 就必须对红黑树有一定的了解。

**如下为红黑树图：**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/ab6353257cb24e318341ac0df568af65.jpg)

红黑树是基于平衡二叉树实现的，基于平衡二叉树的实现还有 AVL 算法，也被称为 **AVL** 树，AVL 算法要求**任何节点的两棵子树的高度差不大于1**，为了保持树的高度差不大于1，主要有2中调整方式：**左旋转**、**右旋转**。

**绕某元素右旋转，图解如下：**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/8160ce5d3ef7469eba057d835a0493ed.jpg)

**绕某元素右旋转，图解如下：**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/0aeb1ffde9324b638a0808d7ef482b22.jpg)

虽然 AVL 算法保证了树高度平衡，查询效率极高，但是也有缺陷，在删除某个节点时，需要多次**左旋转或右旋转调整**，**红黑树的出现就是为了解决尽可能少的调整，提高平衡二叉树的整体性能！**

那么对于一棵有效的红黑树，主要有以下规则：
* **1、每个节点要么是红色，要么是黑色，但根节点永远是黑色的；**
* **2、每个红色节点的两个子节点一定都是黑色；**
* **3、红色节点不能连续（也即是，红色节点的孩子和父亲都不能是红色）；**
* **4、从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点；**
* **5、所有的叶节点都是是黑色的（注意这里说叶子节点其实是上图中的 NIL 节点）；**

红黑树为了保证树的基本平衡，调整也有类似 **AVL** 算法那样的**左旋转**、**右旋转**操作；同时，红黑树因为红色节点不能连续，因此还增加一个颜色调整操作：**节点颜色转换**。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/db79949d82ae4257a92cf5cdc6b2aa14.jpg)

**AVL**虽然可以保证平衡二叉树高度平衡，但是在删除某个节点的时候，最多需要 n 次调整，也就是左旋转、右旋转；而红黑树，虽然也是基于平衡二叉树实现，但是并不像 **AVL** 那样高度平衡，而是**基本平衡**，在删除某个节点的时候，至多 3 次调整，效率比 **AVL** 高！

红黑树，在插入的时候，与 **AVL** 一样，结点最多只需要2次旋转；在查询方面，因为不是高度平衡，红黑树在查询效率方面稍逊于 AVL，但是比二叉查找树强很多！

就整体来看，红黑树的性能要好于 **AVL**，只是实现稍微复杂了一些，有兴趣的朋友可以参阅小编之前写的文章**《3分钟看完关于树的故事》**！

#### 3.4、IdentityHashMap
> IdentityHashMap 从名字上看，感觉表示唯一的 HashMap，然后并不是，别被它的名称所欺骗哦。

IdentityHashMap 的数据结构很简单，底层实际就是一个 Object 数组，在存储上并没有使用链表来存储，而是将 K 和 V 都存放在 Object 数组上。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/d00c58b3d6044333b040f791d0aef783.jpg)

当添加元素的时候，会根据 Key 计算得到散列位置，如果发现该位置上已经有改元素，直接进行新值替换；如果没有，直接进行存放。当元素个数达到一定阈值时，Object 数组会自动进行扩容处理。

IdentityHashMap 的实现也不同于 HashMap，虽然也是数组，不过 IdentityHashMap 中没有用到链表，解决冲突的方式是计算下一个有效索引，并且将数据 `key` 和 `value` 紧挨着存入 `map` 中，即`table[i]=key`、`table[i+1]=value`；

IdentityHashMap 允许`key`、`value`都为`null`，当`key`为`null`的时候，默认会初始化一个`Object`对象作为`key`；

#### 3.5、WeakHashMap
> WeakHashMap 是 Map 体系中一个很特殊的成员，它的特殊之处在于  WeakHashMap 里的元素可能会被 GC 自动删除，即使程序员没有显示调用 remove() 或者 clear() 方法。

换言之，当向 WeakHashMap 中添加元素的时候，再次遍历获取元素，可能发现它已经不见了，我们来看看下面这个例子。
```java
public static void main(String[] args) {
        Map weakHashMap = new WeakHashMap();
        //向weakHashMap中添加3个元素
		weakHashMap.put("key-1", "value-1");
		weakHashMap.put("key-2", "value-2");
		weakHashMap.put("key-3", "value-3");
        //输出添加的元素
        System.out.println("数组长度："+weakHashMap.size() + "，输出结果：" + weakHashMap);
        //主动触发一次GC
        System.gc();
        //再输出添加的元素
        System.out.println("数组长度："+weakHashMap.size() + "，输出结果：" + weakHashMap);
}
```
输出结果：
```
数组长度：3，输出结果：{key-2=value-2, key-1=value-1, key-0=value-0}
数组长度：3，输出结果：{}
```
当主动调用 GC 回收器的时候，再次查询 WeakHashMap 里面的数据的时候，数据已经被 GC 收集了，内容为空。

这是因为 WeakHashMap 的 `key` 使用了**弱引用类型**，在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

WeakHashMap 跟普通的 HashMap 不同，在存储数据时，key 被设置为弱引用类型，而弱引用类型在 java 中，可能随时被 jvm 的 gc 回收，所以再次通过获取对象时，可能得到空值，而 value 是在访问数组内容的时候，进行清除。

可能很多人觉得这样做很奇葩，其实不然，WeekHashMap 的这个特点特别适用于需要缓存的场景。

在缓存场景下，由于系统内存是有限的，不能缓存所有对象，可以使用 WeekHashMap 进行缓存对象，即使缓存丢失，也可以通过重新计算得到，不会造成系统错误。

比较典型的例子，Tomcat 中的 ConcurrentCache 类就使用了 WeekHashMap 来实现数据缓存。

#### 3.6、Hashtable
> Hashtable 一个元老级的集合类，早在 JDK 1.0 就诞生了，而 HashMap 诞生于 JDK 1.2。

在实现上，HashMap 可以看作是 Hashtable 的一个迷你版，虽然二者的底层数据结构都是 **数组 + 链表** 结构，具有查询、插入、删除快的特点，但是二者又有很多的不同。

* **虽然 HashMap 和 Hashtable 都实现了 Map 接口，但 Hashtable 继承于 Dictionary 类，而 HashMap 是继承于 AbstractMap；**
* **HashMap 可以允许存在一个为 null 的 key 和任意个为 null 的 value，但是 HashTable 中的 key 和 value 都不允许为 null；**
* **Hashtable 的方法是同步的，因为在方法上加了 synchronized 同步锁，而 HashMap 是非线程安全的；**

虽然 Hashtable 是 HashMap 线程安全的一个版本，但是官方已经不推荐使用 HashTable了，因为历史遗留原因，并没有移除此类，如果需要在线程安全的环境下使用 HashMap，那么推荐使用 ConcurrentHashMap。在上文中已经有所提到！

#### 3.7、Properties
> 在 Java 中，其实还有一个非常重要的类 Properties，它继承自 Hashtable，主要用于读取配置文件。

Properties 类是 java 工具包中非常重要的一个类，比如在实际开发中，有些变量，我们可以直接硬写入到自定义的 java 枚举类中。

但是有些变量，在测试环境、预生产环境、生产环境，变量所需要取的值都不一样，这个时候，我们可以通过使用 properties 文件来加载程序需要的配置信息，以达到一行代码，多处环境都可以运行的效果！

比如，最常见的 JDBC 数据源配置文件，`properties`文件以`.properties`作为后缀，文件内容以`键=值`格式书写，左边是变量名称，右边是变量值，用`#`做注释，新建一个`jdbc.properties`文件，内容如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/ca2d5e3cb12b45718cde92d2f6f62997.jpg)

Properties 继承自 Hashtable，大部分方法都复用于 Hashtable，**与 Hashtable 不同的是， Properties 中的 key 和 value 都是字符串**。

实际开发中，Properties 主要用于读取配置文件，尤其是在不同的环境下，变量值需要不一样的情况，可以通过读取配置文件来避免将变量值写死在 java 的枚举类中，以达到一行代码，多处运行的目的！

### 四、Set
> Set集合的特点：元素不重复，存取无序，无下标。

Set 集合，在元素存储方面，注重**独立无二**的特性，如果某个元素在集合中已经存在，不会存储重复的元素，同时，集合存储的是元素，不像 Map 集合那样存储的是键值对。

**Set 集合中，各个类继承结构图：**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/7b7f433efd8a4598847cc72868df3f33.jpg)

从图中可以看出，Set 集合主要实现类有： HashSet、LinkedHashSet 、TreeSet 、EnumSet（ RegularEnumSet、JumboEnumSet ）等等

#### 4.1、HashSet
HashSet 是一个输入输出无序的集合，底层基于 **HashMap** 来实现，HashSet 利用 **HashMap** 中的 key 元素来存放元素，这一点我们可以从源码上看出来，源码定义如下：
```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable{
    // HashMap 变量
    private transient HashMap<E,Object> map;
    /**HashSet 初始化*/
    public HashSet() {
        //默认实例化一个 HashMap
        map = new HashMap<>();
    }
}
```

因为 HashMap 允许 key 为空为`null`，所以 HashSet 也允许添加为`null`的元素。

#### 4.2、LinkedHashSet
LinkedHashSet 是一个输入输出有序的集合，继承自 HashSet，但是底层基于 LinkedHashMap 来实现。

LinkedHashSet 的源码，类定义如下：
```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    public LinkedHashSet() {
        //调用 HashSet 的方法
        super(16, .75f, true);
    }
}
```
查询源码，`super`调用的方法，源码如下：
```java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        //初始化一个 LinkedHashMap
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```
LinkedHashSet 与 HashSet 相比，LinkedHashSet 保证了元素输入输出有序！

#### 4.3、TreeSet
TreeSet 是一个排序的集合，实现了`NavigableSet`、`SortedSet`、`Set`接口，底层基于 `TreeMap` 来实现。

TreeSet 利用 TreeMap 中的 key 元素来存放元素，这一点我们也可以从源码上看出来，阅读源码，类定义如下：

```java
public class TreeSet<E> extends AbstractSet<E>
implements NavigableSet<E>, Cloneable, java.io.Serializable {
    //TreeSet 使用NavigableMap接口作为变量
    private transient NavigableMap<E,Object> m;
    /**对象初始化*/
    public TreeSet() {
        //默认实例化一个 TreeMap 对象
        this(new TreeMap<E,Object>());
    }
    //对象初始化调用的方法
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }
}
```
`new TreeSet<>()`对象实例化的时候，表达的意思，可以简化为如下：
```java
NavigableMap<E,Object> m = new TreeMap<E,Object>();
```
因为`TreeMap`实现了`NavigableMap`接口，所以没啥问题。
```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable{
    ......
}
```
TreeSet 是一个排序的 Set 集合，元素不可重复，底层基于 TreeMap 的 key 来实现，元素不可以为空，默认按照自然排序来存放元素，也可以使用 `Comparable`和 `Comparator` 接口来比较大小，实现自定义排序。

#### 4.4、EnumSet
> EnumSet 是一个与枚举类型搭配使用的专用 Set 集合，在 jdk1.5 中加入。

与 HashSet、LinkedHashSet 、TreeSet 不同的是，EnumSet 元素必须是Enum的类型，并且所有元素都必须来自同一个枚举类型。

新建一个EnumEntity的枚举类型，定义2个参数。
```java
public enum EnumEntity {
    WOMAN,MAN;
}
```
创建一个 EnumSet，并将枚举类型的元素全部添加进去！
```
//创建一个 EnumSet，将EnumEntity 元素内容添加到EnumSet中
EnumSet<EnumEntity> allSet = EnumSet.allOf(EnumEntity.class);
System.out.println(allSet);
```
输出结果：
```java
[WOMAN, MAN]
```

在 Java 中，EnumSet 是一个虚类，有2个实现类 `RegularEnumSet`、`JumboEnumSet`，**不能显式的实例化改类，EnumSet 会动态决定使用哪一个实现类，当元素个数小于等于64的时候，使用 RegularEnumSet；大于 64的时候，使用JumboEnumSet类**。

**EnumSet 其内部使用位向量实现，拥有极高的时间和空间性能，如果元素是枚举类型，推荐使用 EnumSet。**

#### 4.5、小结
![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/ac2bcfd92f0748fa94d0eac00d0492ce.jpg)

### 五、Queue
> 在 jdk1.5 中，新增了 Queue 接口，代表一种队列集合的实现。

Queue 接口是由大名鼎鼎的 Doug Lea 创建，中文名为道格·利，翻开 JDK1.8 源代码，可以将 Queue 接口旗下的实现类抽象成如下结构图：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/39afc338bdef4f148c2262c0c2862740.jpg)

从上图可以看出，Queue 接口主要实现类有：ArrayDeque、LinkedList、PriorityQueue。

#### 5.1、ArrayDeque
ArrayQueue 是一个**基于数组实现的双端队列**，可以想象，在队列中存在两个指针，一个指向头部，一个指向尾部，因此它具有`FIFO队列`及`LIFO栈`的方法特性。

**其中队列（FIFO）表示先进先出，比如水管，先进去的水先出来；栈（LIFO）表示先进后出，比如，手枪弹夹，最后进去的子弹，最先出来。**

**如下为 ArrayDeque 数据结构图，head 表示头部指针，tail表示尾部指针：**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/d063111552104d97a1e7aff16fae18ca.jpg)

在上文中，我们也说到 Stack 也可以作为**栈**使用，但是 ArrayDeque 的效率要高于 Stack 类，并且功能也比 Stack 类丰富的多，当需要使用栈时，Java 已不推荐使用 Stack，而是推荐使用更高效的 ArrayDeque。

#### 5.2、LinkedList
LinkedList 是一个**基于链表实现的双端队列**，在上文中我们也说到 LinkedList 实现自 List，作为一个双向链表时，增加或删除元素的效率较高，如果查找的话需要遍历整个链表，效率较低！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/c8aa4748a5224095b682ab4347ec2ee9.png)

于此同时，LinkedList 也实现了 Deque 接口，既可以作队列使用也可以作为栈使用。

从上图中可以得出，ArrayDeque 和 LinkedList 都是 Deque 接口的实现类，都具备既可以作为队列，又可以作为栈来使用的特性，两者主要区别在于底层数据结构的不同。

**ArrayDeque 底层数据结构是以循环数组为基础，而 LinkedList 底层数据结构是以循环链表为基础。**

理论上，链表在添加、删除方面性能高于数组结构，在查询方面数组结构性能高于链表结构，但是对于数组结构，如果不进行数组移动，在添加方面效率也很高。

下面，分别以10万条数据为基础，通过添加、删除，来测试两者作为队列、栈使用时所消耗的时间，测试结果如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/efb4c5cf7f154612974847777f9ecea3.jpg)

从数据上可以看出，在 10 万条数据下，两者性能都差不多，当达到 100 万条、1000 万条数据的时候，两者的差别就比较明显了，ArrayDeque 无论是作为队列还是作为栈使用，性能均高于 LinkedList 。

为什么 ArrayDeque 性能，在大数据量的时候，明显高于 LinkedList？

个人分析，LinkedList 底层是以循环链表来实现的，每一个节点都有一个前驱、后继的变量，也就是说，每个节点上都存放有它上一个节点的指针和它下一个节点的指针，同时还包括它自己的元素，每次插入或删除节点时需要修改前驱、后继节点变量，在同等的数据量情况下，链表的内存开销要明显大于数组，同时因为 ArrayDeque 底层是数组结构，天然在查询方面占优势，在插入、删除方面，只需要移动一下头部或者尾部变量，时间复杂度是 O（1）。

所以，在大数据量的时候，LinkedList 的内存开销明显大于 ArrayDeque，在插入、删除方面，都要频发修改节点的前驱、后继变量；而 ArrayDeque 在插入、删除方面依然保存高性能。

如果对于小数据量，ArrayDeque 和 LinkedList 在效率方面相差不大，但是对于大数据量，推荐使用 ArrayDeque。

#### 5.3、PriorityQueue
> PriorityQueue 并没有直接实现 Queue接口，而是通过继承 AbstractQueue 类来实现 Queue 接口一些方法，在 Java 定义中，PriorityQueue 是一个基于优先级的无界优先队列。

通俗的说，添加到 PriorityQueue 队列里面的元素都经过了排序处理，默认按照自然顺序，也可以通过 Comparator 接口进行自定义排序。

优先队列的作用是保证每次取出的元素都是队列中权值最小的。

PriorityQueue 排序实现与上文中说到的 TreeMap 类似。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-17/33fc68440ddc4e9aa8151e488c1a21f1.jpg)

在 Java 中 **PriorityQueue** 是一个使用数组结构来存储元素的优先队列，虽然它也实现了`Queue`接口，但是元素存取并不是先进先出，而是通过一个二叉小顶堆实现的，默认底层使用自然排序规则给插入的元素进行排序，也可以使用自定义比较器来实现排序，每次取出的元素都是队列中权值最小的。

同时需要注意的是，PriorityQueue 不能插入`null`，否则报空指针异常！

### 六、总结
以上主要是对 java 集合往期文章内容做了一个简单的总结，有兴趣的可以参阅各篇详细内容，如果有理解不当之处，欢迎指正！
