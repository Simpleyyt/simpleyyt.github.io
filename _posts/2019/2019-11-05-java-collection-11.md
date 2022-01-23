---
layout: post
title: 【集合系列】- 深入浅出的分析 Set集合
tagline: by 炸鸡可乐
categories: Java
tags: 
  - 炸鸡可乐
---

前几篇文章中，咱们聊到 List、Map 接口相关的实现类，今天咱们来聊聊集合中的 Set 接口！

<!--more-->
### 01、摘要
> 关于 Set 接口，在实际开发中，其实很少用到，但是如果你出去面试，它可能依然是一个绕不开的话题。

言归正传，废话咱们也不多说了，相信使用过 Set 集合类的朋友都知道，Set集合的特点主要有：**元素不重复、存储无序**的特点。

啥意思呢？你可以理解为，向一个瓶子里面扔东西，这些东西没有记号是第几个放进去的，但是有一点就是这个瓶子里面不会有重样的东西。

细细思考，你会发现， Set 集合的这些特性正处于 List 集合和 Map 集合之间，为什么这么说呢？之前的集合文章中，咱们了解到，List 集合的特点就是存取有序，**本质是一个有序数组，每个元素依次按照顺序存储**；Map 集合主要用于存放**键值对**，虽然底层也是用数组存放，但是元素在数组中的下标是通过哈希算法计算出来的，**数组下标无序**。

而 Set 集合，在元素存储方面，注重**独立无二**的特性，如果某个元素在集合中已经存在，不会存储重复的元素，同时，集合存储的是元素，不像 Map 集合那样存储的是键值对。

具体的分析，咱们慢慢道来，打开 Set 集合，主要实现类有 HashSet、LinkedHashSet 、TreeSet 、EnumSet（ RegularEnumSet、JumboEnumSet ）等等，总结 Set 接口实现类，图如下：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/8a80c04994df49d3ae889db17a57bbbb.jpg)

由图中的继承关系，可以知道，Set 接口主要实现类有 AbstractSet、HashSet、LinkedHashSet 、TreeSet 、EnumSet（ RegularEnumSet、JumboEnumSet ），其中 AbstractSet、EnumSet 属于抽象类，EnumSet 是在 jdk1.5 中新增的，不同的是 EnumSet 集合元素必须是枚举类型。
* HashSet 是一个输入输出无序的集合，集合中的元素基于 HashMap 的 key 实现，元素不可重复；
* LinkedHashSet 是一个输入输出有序的集合，集合中的元素基于 LinkedHashMap  的 key 实现，元素也不可重复；
* TreeSet 是一个排序的集合，集合中的元素基于 TreeMap 的 key 实现，同样元素不可重复；
* EnumSet 是一个与枚举类型一起使用的专用 Set 集合，其中 RegularEnumSet 和 JumboEnumSet 不能单独实例化，只能由 EnumSet 来生成，同样元素不可重复；

下面咱们来对各个主要实现类进行一一分析！

### 02、HashSet
HashSet 是一个输入输出无序的集合，底层基于 HashMap 来实现，**HashSet 利用 HashMap 中的`key`元素来存放元素**，这一点我们可以从源码上看出来，阅读源码如下：
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
#### 2.1、add方法
打开`HashSet`的`add()`方法，源码如下：
```java
public boolean add(E e) {
    //向 HashMap 中添加元素
    return map.put(e, PRESENT)==null;
}
```
其中变量`PRESENT`，是一个非空对象，源码部分如下：
```java
private static final Object PRESENT = new Object();
```
可以分析出，当进行`add()`的时候，等价于
```java
HashMap map = new HashMap<>();
map.put(e, new Object());//e 表示要添加的元素
```
在之前的集合文章中，咱们了解到 HashMap 在添加元素的时候 ，通过`equals()`和`hashCode()`方法来判断传入的`key`是否相同，如果相同，那么 HashMap 认为添加的是同一个元素，反之，则不是。

从源码分析上可以看出，HashSet 正是使用了 HashMap 的这一特性，实现存储元素下标无序、元素不会重复的特点。

#### 2.2、remove方法
HashSet 的删除方法，同样如此，也是基于 HashMap 的底层实现，源码如下：
```java
public boolean remove(Object o) {
    //调用HashMap 的remove方法，移除元素
    return map.remove(o)==PRESENT;
}
```
#### 2.3、查询方法
HashSet 没有像 List、Map 那样提供 get 方法，而是使用迭代器或者 for 循环来遍历元素，方法如下：
```java
public static void main(String[] args) {
    Set<String> hashSet = new HashSet<String>();
    System.out.println("HashSet初始容量大小："+hashSet.size());
    hashSet.add("1");
    hashSet.add("2");
    hashSet.add("3");
    hashSet.add("3");
    hashSet.add("2");
    hashSet.add(null);

    //相同元素会自动覆盖
    System.out.println("HashSet容量大小："+hashSet.size());
    //迭代器遍历
    Iterator<String> iterator = hashSet.iterator();
    while (iterator.hasNext()){
        String str = iterator.next();
        System.out.print(str + ",");
    }

    System.out.println("\n===========");
    //增强for循环
    for (String str : hashSet) {
        System.out.print(str + ",");
    }
}
```
输出结果：
```java
HashSet初始容量大小：0
HashSet容量大小：4
null,1,2,3,
===========
null,1,2,3,
```
需要注意的是，HashSet 允许添加为`null`的元素。
### 03、LinkedHashSet
LinkedHashSet 是一个输入输出有序的集合，继承自 HashSet，但是底层基于 LinkedHashMap 来实现。

如果你之前了解过 LinkedHashMap，那么你一定知道，它也继承自 HashMap，唯一有区别的是，**LinkedHashMap 底层数据结构基于循环链表实现，并且数组指定了头部和尾部，虽然数组的下标存储无序，但是却可以通过数组的头部和尾部，加上循环链表，依次可以查询到元素存储的过程，从而做到输入输出有序的特点**。

如果还不了解 LinkedHashMap 的实现过程，可以参阅集合系列中关于 LinkedHashMap 的实现过程文章。

阅读 LinkedHashSet 的源码，类定义如下：
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
#### 3.1、add方法
`LinkedHashSet`没有重写`add`方法，而是直接调用`HashSet`的`add()`方法，因为`map`的实现类是`LinkedHashMap`，所以此处是向`LinkedHashMap`中添加元素，当进行`add()`的时候，等价于
```java
HashMap map = new LinkedHashMap<>();
map.put(e, new Object());//e 表示要添加的元素
```
#### 3.2、remove方法
`LinkedHashSet`也没有重写`remove`方法，而是直接调用`HashSet`的删除方法，因为`LinkedHashMap`没有重写`remove`方法，所以调用的也是`HashMap`的`remove`方法，源码如下：
```java
public boolean remove(Object o) {
    //调用HashMap 的remove方法，移除元素
    return map.remove(o)==PRESENT;
}
```
#### 3.3、查询方法
同样的，LinkedHashSet 没有提供 get 方法，使用迭代器或者 for 循环来遍历元素，方法如下：
```java
public static void main(String[] args) {
    Set<String> linkedHashSet = new LinkedHashSet<String>();
    System.out.println("linkedHashSet初始容量大小："+linkedHashSet.size());
    linkedHashSet.add("1");
    linkedHashSet.add("2");
    linkedHashSet.add("3");
    linkedHashSet.add("3");
    linkedHashSet.add("2");
    linkedHashSet.add(null);
    linkedHashSet.add(null);

    System.out.println("linkedHashSet容量大小："+linkedHashSet.size());
    //迭代器遍历
    Iterator<String> iterator = linkedHashSet.iterator();
    while (iterator.hasNext()){
        String str = iterator.next();
        System.out.print(str + ",");
    }

    System.out.println("\n===========");
    //增强for循环
    for (String str : linkedHashSet) {
        System.out.print(str + ",");
    }
}
```
输出结果：
```java
linkedHashSet初始容量大小：0
linkedHashSet容量大小：4
1,2,3,null,
===========
1,2,3,null,
```
可见，LinkedHashSet 与 HashSet 相比，**LinkedHashSet 输入输出有序。**
### 04、TreeSet
TreeSet 是一个排序的集合，实现了`NavigableSet`、`SortedSet`、`Set`接口，底层基于 TreeMap 来实现。**TreeSet 利用 TreeMap 中的`key`元素来存放元素**，这一点我们也可以从源码上看出来，阅读源码，类定义如下：
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
`new TreeSet<>() `对象实例化的时候，表达的意思，可以简化为如下：
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
#### 4.1、add方法
打开`TreeSet`的`add()`方法，源码如下：
```java
public boolean add(E e) {
    //向 TreeMap 中添加元素
    return m.put(e, PRESENT)==null;
}
```
其中变量`PRESENT`，也是是一个非空对象，源码部分如下：
```java
private static final Object PRESENT = new Object();
```
可以分析出，当进行`add()`的时候，等价于
```java
TreeMap map = new TreeMap<>();
map.put(e, new Object());//e 表示要添加的元素
```
TreeMap 类主要功能在于，给添加的集合元素，按照一个的规则进行了排序，默认以自然顺序进行排序，当然也可以自定义排序，比如测试方法如下：
```java
public static void main(String[] args) {
    Map initMap = new TreeMap();
    initMap.put("4", "d");
    initMap.put("3", "c");
    initMap.put("1", "a");
    initMap.put("2", "b");
    //默认自然排序，key为升序
    System.out.println("默认 排序结果:" + initMap.toString());
    //自定义排序，在TreeMap初始化阶段传入Comparator 内部对象
    Map comparatorMap = new TreeMap<String, String>(new Comparator<String>() {
        @Override
        public int compare(String o1, String o2){
            //根据key比较大小，采用倒叙，以大到小排序
            return o2.compareTo(o1);
        }
    });
    comparatorMap.put("4", "d");
    comparatorMap.put("3", "c");
    comparatorMap.put("1", "a");
    comparatorMap.put("2", "b");
    System.out.println("自定义 排序结果:" + comparatorMap.toString());
}
```
输出结果：
```java
默认 排序结果:{1=a, 2=b, 3=c, 4=d}
自定义 排序结果:{4=d, 3=c, 2=b, 1=a}
```
相信使用过`TreeMap`的朋友，一定知道`TreeMap`会自动将`key`按照一定规则进行排序，`TreeSet`正是使用了`TreeMap`这种特性，来实现添加的元素集合，在输出的时候，其结果是已经排序好的。

如果您没看过源码`TreeMap`的实现过程，可以参阅集合系列文章中`TreeMap`的实现过程介绍，或者阅读 jdk 源码。
#### 4.2、remove方法
TreeSet 的删除方法，同样如此，也是基于 TreeMap 的底层实现，源码如下：
```java
public boolean remove(Object o) {
        //调用TreeMap 的remove方法，移除元素
        return m.remove(o)==PRESENT;
}
```
#### 4.3、查询方法
TreeSet 没有重写 get 方法，而是使用迭代器或者 for 循环来遍历元素，方法如下：
```java
public static void main(String[] args) {
    Set<String> treeSet = new TreeSet<>();
    System.out.println("treeSet初始容量大小："+treeSet.size());
    treeSet.add("1");
    treeSet.add("4");
    treeSet.add("3");
    treeSet.add("8");
    treeSet.add("5");

    System.out.println("treeSet容量大小："+treeSet.size());
    //迭代器遍历
    Iterator<String> iterator = treeSet.iterator();
    while (iterator.hasNext()){
        String str = iterator.next();
        System.out.print(str + ",");
    }

    System.out.println("\n===========");
    //增强for循环
    for (String str : treeSet) {
        System.out.print(str + ",");
    }
}
```
输出结果：
```java
treeSet初始容量大小：0
treeSet容量大小：5
1,3,4,5,8,
===========
1,3,4,5,8,
```
#### 4.4、自定义排序
使用自定义排序，有 2 种方法，第一种在需要添加的元素类，实现`Comparable `接口，重写`compareTo`方法来实现对元素进行比较，实现自定义排序。
##### 方法一
```java
/**
  * 创建实体类Person实现Comparable接口
  */
public class Person implements Comparable<Person>{
    private int age;
    private String name;
    public Person(String name, int age){
        this.name = name;
        this.age = age;
    }
    @Override
    public int compareTo(Person o){
        //重写 compareTo 方法，自定义排序算法
        return this.age-o.age;
    }
    @Override
    public String toString(){
        return name+":"+age;
    }
}
```
创建一个`Person`实体类，实现`Comparable`接口，重写`compareTo`方法，通过变量`age`实现自定义排序
测试方法如下：
```java
public static void main(String[] args) {
    Set<Person> treeSet = new TreeSet<>();
    System.out.println("treeSet初始容量大小："+treeSet.size());
    treeSet.add(new Person("李一",18));
    treeSet.add(new Person("李二",17));
    treeSet.add(new Person("李三",19));
    treeSet.add(new Person("李四",21));
    treeSet.add(new Person("李五",20));

    System.out.println("treeSet容量大小："+treeSet.size());
    System.out.println("按照年龄从小到大，自定义排序结果：");
    //迭代器遍历
    Iterator<Person> iterator = treeSet.iterator();
    while (iterator.hasNext()){
        Person person = iterator.next();
        System.out.print(person.toString() + ",");
    }
}
```
输出结果：
```java
treeSet初始容量大小：0
treeSet容量大小：5
按照年龄从小到大，自定义排序结果：
李二:17,李一:18,李三:19,李五:20,李四:21,
```
##### 方法二
第二种方法是在`TreeSet`初始化阶段，`Person`不用实现`Comparable`接口，将`Comparator `接口以内部类的形式作为参数，初始化进去，方法如下：
```java
public static void main(String[] args) {
    //自定义排序
    Set<Person> treeSet = new TreeSet<>(new Comparator<Person>(){
        @Override
        public int compare(Person o1, Person o2) {
            if(o1 == null || o2 == null){
                //不用比较
                return 0;
            }
            //从小到大进行排序
            return o1.getAge() - o2.getAge();
        }
    });
    System.out.println("treeSet初始容量大小："+treeSet.size());
    treeSet.add(new Person("李一",18));
    treeSet.add(new Person("李二",17));
    treeSet.add(new Person("李三",19));
    treeSet.add(new Person("李四",21));
    treeSet.add(new Person("李五",20));

    System.out.println("treeSet容量大小："+treeSet.size());
    System.out.println("按照年龄从小到大，自定义排序结果：");
    //迭代器遍历
    Iterator<Person> iterator = treeSet.iterator();
    while (iterator.hasNext()){
        Person person = iterator.next();
        System.out.print(person.toString() + ",");
    }
}
```
输出结果：
```java
treeSet初始容量大小：0
treeSet容量大小：5
按照年龄从小到大，自定义排序结果：
李二:17,李一:18,李三:19,李五:20,李四:21,
```
需要注意的是，**`TreeSet`不能添加为空的元素，否则会报空指针错误！**
### 05、EnumSet
EnumSet 是一个与枚举类型一起使用的专用 Set 集合，继承自`AbstractSet`抽象类。与 HashSet、LinkedHashSet 、TreeSet 不同的是，EnumSet 元素必须是`Enum`的类型，并且所有元素都必须来自同一个枚举类型，EnumSet 定义源码如下：
```java
public abstract class EnumSet<E extends Enum<E>> extends AbstractSet<E>
    implements Cloneable, java.io.Serializable {
    ......
}
```
`EnumSet`是一个虚类，不能直接通过实例化来获取对象，只能通过它提供的静态方法来返回`EnumSet`实现类的实例。

`EnumSet`的实现类有两个，分别是`RegularEnumSet`、`JumboEnumSet`两个类，两个实现类都继承自`EnumSet`。

`EnumSet`会根据枚举类型中元素的个数，来决定是返回哪一个实现类，当 `EnumSet`元素中的元素个数小于或者等于`64`，就会返回`RegularEnumSet`实例；当`EnumSet`元素个数大于`64`，就会返回`JumboEnumSet`实例。

这一点，我们可以从源码中看出，源码如下：
```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");
    //当元素个数小于或者等于 64 的时候，返回 RegularEnumSet
    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        //大于64，返回 JumboEnumSet
        return new JumboEnumSet<>(elementType, universe);
}
```
`noneOf`是`EnumSet`中一个静态方法，用于判断是返回哪一个实现类。

我们来看看当元素个数小于等于64的时候，使用`RegularEnumSet`的类，源码如下：
```java
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {

    /**元素为long型*/
    private long elements = 0L;

    /**添加元素*/
    public boolean add(E e) {
        typeCheck(e);

        long oldElements = elements;
        //二进制运算，获取元素
        elements |= (1L << ((Enum<?>)e).ordinal());
        return elements != oldElements;
    }
}
```
RegularEnumSet 通过二进制运算得到结果，直接使用`long`来存放元素。

我们再来看看当元素个数大于64的时候，使用`JumboEnumSet`的类，源码如下：
```java
class JumboEnumSet<E extends Enum<E>> extends EnumSet<E> {

    /**元素为long型*/
    private long elements = 0L;

    /**添加元素*/
    public boolean add(E e) {
        typeCheck(e);

        int eOrdinal = e.ordinal();
        int eWordNum = eOrdinal >>> 6;

        long oldElements = elements[eWordNum];
        //二进制运算
        elements[eWordNum] |= (1L << eOrdinal);
        //使用数组来操作元素
        boolean result = (elements[eWordNum] != oldElements);
        if (result)
            size++;
        return result;
    }
}
```
JumboEnumSet 也是通过二进制运算得到结果，使用`long`来存放元素，但是它是使用数组来存放元素。

二者相比，RegularEnumSet 效率比 JumboEnumSet 高些，因为操作步骤少，大多数情况下返回的是 RegularEnumSet，只有当枚举元素个数超过 64 的时候，会使用 JumboEnumSet。
#### 5.1、添加元素
新建一个`EnumEntity`的枚举类型，定义2个参数。
```java
public enum EnumEntity {
    WOMAN,MAN;
}
```
创建一个空的 EnumSet！
```java
//创建一个 EnumSet，内容为空
EnumSet<EnumEntity> noneSet = EnumSet.noneOf(EnumEntity.class);
System.out.println(noneSet);
```
输出结果：
```java
[]
```
创建一个 EnumSet，并将枚举类型的元素全部添加进去！
```java
//创建一个 EnumSet，将EnumEntity 元素内容添加到EnumSet中
EnumSet<EnumEntity> allSet = EnumSet.allOf(EnumEntity.class);
System.out.println(allSet);
```
输出结果：
```java
[WOMAN, MAN]
```
创建一个 EnumSet，添加指定的枚举元素！
```java
//创建一个 EnumSet，添加 WOMAN 到 EnumSet 中
EnumSet<EnumEntity> customSet = EnumSet.of(EnumEntity.WOMAN);
System.out.println(customSet);
```
#### 5.2、查询元素
`EnumSet`与`HashSet`、`LinkedHashSet`、`TreeSet`一样，通过迭代器或者 for 循环来遍历元素，方法如下：
```java
EnumSet<EnumEntity> allSet = EnumSet.allOf(EnumEntity.class);
for (EnumEntity enumEntity : allSet) {
    System.out.print(enumEntity + ",");
}
```
输出结果：
```java
WOMAN,MAN,
```
### 06、总结
![](http://www.justdojava.com/assets/images/2019/java/image-jay/de24435cce124364a9cc2220978dbd1f.jpg)

* **HashSet 是一个输入输出无序的 Set 集合，元素不重复，底层基于 HashMap 的 key 来实现，元素可以为空，如果添加的元素为对象，对象需要重写 equals() 和 hashCode() 方法来约束是否为相同的元素。**

* **LinkedHashSet 是一个输入输出有序的 Set 集合，继承自 HashSet，元素不重复，底层基于 LinkedHashMap 的 key来实现，元素也可以为空，LinkedHashMap 使用循环链表结构来保证输入输出有序。**

* **TreeSet 是一个排序的  Set 集合，元素不可重复，底层基于 TreeMap 的 key来实现，元素不可以为空，默认按照自然排序来存放元素，也可以使用 Comparable 和 Comparator 接口来比较大小，实现自定义排序。**

* **EnumSet 是一个与枚举类型搭配使用的专用 Set 集合，在 jdk1.5 中加入。EnumSet 是一个虚类，有2个实现类 RegularEnumSet、JumboEnumSet，不能显式的实例化改类，EnumSet 会动态决定使用哪一个实现类，当元素个数小于等于64的时候，使用 RegularEnumSet；大于 64的时候，使用JumboEnumSet类，EnumSet 其内部使用位向量实现，拥有极高的时间和空间性能，如果元素是枚举类型，推荐使用 EnumSet。**

### 07、参考
1、JDK1.7&JDK1.8 源码

2、[程序园 - java集合－EnumMap与EnumSet ](http://www.voidcn.com/article/p-hoochjxy-ng.html)