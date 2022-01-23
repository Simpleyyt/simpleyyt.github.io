---
layout: post
categories: Dubbo
title: 还在使用集合类完成这些功能？不妨来看看 Guava 集合类!!!
tagline: by 小黑
tags: 
  - 小黑
---

日常开发中，阿粉经常需要用到 Java 提供集合类完成各种需求。Java 集合类虽然非常强大实用，但是提供功能还是有点薄弱。

<!--more-->

举个例子，阿粉最近接到一个需求，从输入一个文档中，统计一个关键词出现的次数。代码如下:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/1fxfb.jpg)

虽然这个需求使用 `Map` 可以轻松搞定，但是阿粉还是觉得这种写法有点笨拙，如果没有判空，将会导致 NPE 异常。

如果很多地方需要功能，我们就可以抽象出来，将其封装成工具类。

不过上面的功能大家就不需要自己封装，一款来自 **Google** 开源工具类-**Guava**，可以轻松的解决上面的统计问题。

## Guava 介绍

Guava 是一款 Google 开源工具类，包含许多 Google 内部 `Java` 项目依赖的核心类。Guava 扩展 Java 基础类工程，比如集合，并发等，也增加一些其他强大功能，比如缓存，限流等功能。

另外 Guava 推出一些类，如 `Optional`，甚至被 Java 开发者学习，后续增加到 JDK 中。

目前 [Guava Github 仓库](https://github.com/google/guava)已有 **36k star**，可以见到 Guava 受欢迎程度。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/wm5ah.jpg)

Guava 核心功能包括多个模块，今天阿粉主要带大家玩转 Guava 集合类。

## 扩展集合类

Guava 创造很多 JDK 没有，但是我们日常却明显有用的新集合类型。这些新类型使用 JDK 集合接口规范，所以使用方法与 JDK 集合框架差不多，并没有增加很多使用难度。

### Multiset

阿粉第一次见到 `Multiset` 这个类，还以为是 `Set` 接口子类。实际上此『Set』，仅仅只是数学上集合概念。

`Multiset` 继承 JDK `Collection` 接口，我们可以多次增加相同的元素，另外 `Multiset` 最大特定将会为元素计数，我们可以将它类似等同为 `Map<E, Integer>` 。

使用 `Multiset`可以轻松解决开头的问题。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/up32k.jpg)

使用 `Multiset` 简化了代码，并且再也不用担心新 **NPE** 的问题。

跟 JDK 集合类一样，`Multiset`也有许多子类。

![来源于 Github](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/28d0l.jpg)

这里阿粉提醒一下大家，虽然上面说过我们可以将 `Multiset<E>` 看做 `Map<E, Integer>`,但是 `Multiset` 可不是 `Map` 的子类，它可是 血统纯正的 `Collection` 子类。

### Multimap

阿粉有时会在业务需求中使用 `Map<String,List<Integer> `实现下面的需求。

```jav
a->[1,2,3] b->4,c->[6,5]
```

使用 `Map` + `List` 这种结构比较笨拙，并且代码实现也比较繁琐。`Multimap` 正式 Guava 中解决这种问题的新出的一个雷。

使用 `Multimap` 实现代码如下:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/fspza.jpg)

这里阿粉使用 `Multimap` 子类 `HashMultimap`，其行为类似为 `Map<K,Set<V>>`，也就是说 `Value` 对应的集合内部元素不能重复。如果需要保存的重复的元素我们可以使用 `ArrayListMultimap`。

 `Multimap`还有其他子类，如图所示：

![来源于 Github](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/fwq7j.jpg)

### BiMap

`BiMap` 可以用来实现键值对的双向映射需求，这样我们就可以通过 `Key` 查找对对应的 `Value`，也可以使用 `Value` 查找对应的 `Key`。

这个需求如果使用 `Map` 实现，我们就不得不使用两个 `Map`，维护双向关系，并且任何改动还要保持同步。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/k6chf.jpg)

使用 `BiMap` 修改上面的代码：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/5lkh3.jpg)

这里需要注意，`BiMap#put`方法不能加入重复元素， 若加入，将会抛错。如果若特定值一定要替换，可以使用 `BiMap#forcePut`代替。

敲黑板，这个知识点记下来。阿粉使用过程中，就踩过这个坑。

同样的 `BiMap` 也有各种实现类：

![来源于 Github](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/thh61.jpg)

### 其他扩展集合类

Guava 另外还提供其他集合类，不过这些类使用起来有点复杂，阿粉还未在业务代码中使用过，这里简单提下，感兴趣同学可以深入了解一下。

- Table 
- ClassToInstanceMap
- RangeSet
- RangeMap

## 集合工具类

除了上面提到的新集合类以外，Guava 提供通用的工具类：

![来源于 Github](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/pq7y1.jpg)

这些工具类需对使用的方法，我们可以快速创建集合，分割集合，转化集合等。

### 快速创建集合实例

使用工具类，我们可以快速创建集合。例如：

```java
List<String> list=Lists.newArrayList();
Set<String> set=Sets.newHashSet();
Map<String,String> map=Maps.newHashMap();
```

相比于 `new` 集合方法，Guava 方法创建方式更加简单。

```java
List<String> list=new ArrayList<String>();
Set<String> set=new HashSet<String>();
Map<String,String> map=new HashMap<String, String>();
```

Guava 工具类智能推导 `List` 泛型，再也不用两侧都重复写泛型了。

另外还可以指定集合类的初始化大小。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/4y0i7.jpg)

### Lists.transform

`Lists#transform`方法可以替代繁琐 `for` 循环，将元素转化，创建一个新集合类。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/tewi4.jpg)

不过使用这个方法我们要**注意**一点。

`Lists#transform` 内部使用懒加载的机制，只有在调用获取的元素的时候，如 `result.get` 才会真正使用 `Function` 从源 `List` 获取元素，做相应的转化。**每次获取元素**都将会使用 `function` 进行转化。

所以使用 `Lists#transform` 得到 `List` 仅仅只是源 `List` 一个视图，任何对源 `List` 的元素修改，都将会被反应到创建之后的 `List` 。任何对创建之后 List 中的元素进行修改，都不会生效。下次再次读取元素时，将会发现相应修改的丢失了。。。

阿粉之前就踩过这个坑，如果你有这种需求，可以使用以下方式创建一个新集合：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/uy0ub.jpg)

> JDK8 之前版本，阿粉经常使用该方法转化 `List` 中的元素。不过你如果使用 JDK8，阿粉还是推荐使用 Stream 流式编程。

### 交集并集差集

`Sets` 提供几个方法，可以快速求出两个 `Set` 集合的交集，并集以及差集。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/qs39i.png)

## 不可变集合

不可变（**Immutable**）集合，顾名思义集合不可以被修改。初始创建不可变集合时吗，需要传入数据源，创建完成之后，集合就再也不能修改，增加，删除元素，否则将会报错。

这是一种防御性策略，防止集合在后续操作中被修改，从而引发问题。

不可变集合优点在于：

- 由于不可变集合仅仅只能读，多线程并发天然安全
- 由于不可变集合固定不变，可以将其当做常量安全，不用单线其他人修改
- 不可变集合占用更少内存空间
- 不可变集合不可以被修改，所以不用担心其他程序任意修改集合

Guava 不可变集合支持 JDK 所有集合接口：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/6bdna.jpg) 

 我们可以使用如下几种方式创建不可变集合，以 `ImmutableList` 为例：

![ImmutableList](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/jb59w.jpg)

```java
List<String> fromList=Lists.newArrayList("点赞","关注");
// 从一个集合拷贝元素
ImmutableList.copyOf(fromList);
ImmutableList.of("关注","Java极客技术");
ImmutableList.builder().add("关注").addAll(fromList).build();
```



## 总结

这篇文章阿粉带大家学习开源工具 Guava 集合的相关类使用方法，日常开发中我们善于使用这些工具类，不要自己重复造轮子。

本篇文章仅仅只是介绍 Guava 一小部分功能，还有很对功能，阿粉也觉得很好用在。这里推荐大家去查看 Guava 官方 wiki，查看具体使用方法。

如果大家还想知道其他开源工具类，给阿粉**点个赞**，下次给大家带来十分好用开源工具类~

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200308/qref4.jpg)





