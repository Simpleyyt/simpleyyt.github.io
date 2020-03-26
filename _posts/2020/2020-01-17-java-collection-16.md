---
layout: post
categories: 集合系列
title: 并发神器之 CopyOnWriteArrayList
tags: 
  - 炸鸡可乐
---

相信大家对 ConcurrentHashMap 这个线程安全类非常熟悉，但是如果我想在多线程环境下使用 ArrayList，该怎么处理呢？阿粉今天来给你揭晓答案！

<!--more-->
### 一、摘要
在介绍 CopyOnWriteArrayList 之前，我们一起先来看看如下方法执行结果，代码内容如下：

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<String>();
    list.add("1");
    list.add("2");
    list.add("1");
    System.out.println("原始list元素："+ list.toString());
    //通过对象移除等于内容为1的元素
    for (String item : list) {
        if("1".equals(item)) {
            list.remove(item);
        }
    }
    System.out.println("通过对象移除后的list元素："+ list.toString());
}
```
执行结果内容如下：
```java
原始list元素：[1, 2, 1]
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at com.example.container.a.TestList.main(TestList.java:16)
```

**很遗憾，结果并没有达到我们想要的预期效果，执行之后直接报错！抛ConcurrentModificationException异常！**

**为啥会抛这个异常呢？**

我们一起来看看，`foreach `写法实际上是对`List.iterator()` 迭代器的一种简写，因此我们可以从分析`List.iterator()` 迭代器进行入手，看看为啥会抛这个异常。

`ArrayList`类中的`Iterator`迭代器实现，源码内容：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/128dfb5c3565447e9d45c777623bff9c.jpg)

通过代码我们发现 `Itr` 是 `ArrayList` 中定义的一个私有内部类，每次调用`next`、`remove`方法时，都会调用`checkForComodification`方法，源码如下：

```java
/**修改次数检查*/
final void checkForComodification() {
	//检查List中的修改次数是否与迭代器类中的修改次数相等
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```
`checkForComodification`方法，实际上是用来检查`List`中的修改次数`modCount`是否与迭代器类中的修改次数`expectedModCount`相等，如果不相等，就会抛出`ConcurrentModificationException`异常！

那么问题基本上已经清晰了，上面的运行结果之所以会抛出这个异常，就是因为`List`中的修改次数`modCount`与迭代器类中的修改次数`expectedModCount`不相同造成的！

**阅读过集合源码的朋友，可能想起`Vector`这个类，它不是 JDK 中 ArrayList 线程安全的一个版本么？**

好的，为了眼见为实，我们把`ArrayList`换成`Vector`来测试一下，代码如下：
```java
public static void main(String[] args) {
    Vector<String> list = new Vector<String>();
    //模拟10个线程向list中添加内容，并且读取内容
    for (int i = 0; i < 5; i++) {
        final int j = i;
        new Thread(new Runnable() {
            @Override
            public void run() {
                //添加内容
                list.add(j + "-j");

                //读取内容
                for (String str : list) {
                    System.out.println("内容：" + str);
                }
            }
        }).start();
    }
}
```
执行程序，运行结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/3b984a4f43204e9c99e97d5ca64af305.jpg)

**还是一样的结果，抛异常了**，`Vector`虽然线程安全，只不过是加了`synchronized`关键字，但是迭代问题完全没有解决！

继续回到本文要介绍的 CopyOnWriteArrayList 类，我们把上面的例子，换成`CopyOnWriteArrayList`类来试试，源码内容如下：

```java
public static void main(String[] args) {
    //将ArrayList换成CopyOnWriteArrayList
    CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
    list.add("1");
    list.add("2");
    list.add("1");
    System.out.println("原始list元素："+ list.toString());

    //通过对象移除等于11的元素
    for (String item : list) {
        if("1".equals(item)) {
            list.remove(item);
        }
    }
    System.out.println("通过对象移除后的list元素："+ list.toString());
}
```
执行结果如下：
```java
原始list元素：[1, 2, 1]
通过对象移除后的list元素：[2]
```
**呃呵，执行成功了，没有报错！是不是很神奇～～**

当然，类似上面这样的例子有很多，比如写10个线程向`list`中添加元素读取内容，也会抛出上面那个异常，操作如下：
```java
public static void main(String[] args) {
    final List<String> list = new ArrayList<>();
    //模拟10个线程向list中添加内容，并且读取内容
    for (int i = 0; i < 10; i++) {
        final int j = i;
        new Thread(new Runnable() {
            @Override
            public void run() {
                //添加内容
                list.add(j + "-j");

                //读取内容
                for (String str : list) {
                    System.out.println("内容：" + str);
                }
            }
        }).start();
    }
}
```
类似的操作例子就非常多了，这里就不一一举例了。

**CopyOnWriteArrayList 实际上是 ArrayList 一个线程安全的操作类！**

从它的名字可以看出，**`CopyOnWrite` 是在写入的时候，不修改原内容，而是将原来的内容复制一份到新的数组，然后向新数组写完数据之后，再移动内存指针，将目标指向最新的位置。**

### 二、简介
从 JDK1.5 开始 Java 并发包里提供了两个使用`CopyOnWrite `机制实现的并发容器，分别是`CopyOnWriteArrayList`和`CopyOnWriteArraySet `。

从名字上看，`CopyOnWriteArrayList`主要针对动态数组，一个线程安全版本的 ArrayList ！

而`CopyOnWriteArraySet`主要针对集，`CopyOnWriteArraySet`可以理解为`HashSet`线程安全的操作类，我们都知道`HashSet`基于散列表`HashMap`实现，**但是`CopyOnWriteArraySet`并不是基于散列表实现，而是基于`CopyOnWriteArrayList`动态数组实现！**

关于这一点，我们可以从它的源码中得出结论，部分源码内容：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/f9fa317653734e7e96d3a23b9b2196aa.jpg)

从源码上可以看出，`CopyOnWriteArraySet`默认初始化的时候，实例化了`CopyOnWriteArrayList`类，`CopyOnWriteArraySet`的大部分方法，例如`add`、`remove`等方法都基于`CopyOnWriteArraySet`实现！

两者最大的不同点是，`CopyOnWriteArrayList`可以允许元素重复，而`CopyOnWriteArraySet`不允许有重复的元素！

**好了，继续来 BB 本文要介绍的`CopyOnWriteArrayList`类～～**

打开`CopyOnWriteArrayList`类的源码，内容如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/efbfdb7541f34155a276952a18c599cf.jpg)

可以看到 `CopyOnWriteArrayList` 的存储元素的数组`array`变量，使用了`volatile`关键字保证的多线程下数据可见行；同时，使用了`ReentrantLock`可重入锁对象，保证线程操作安全。

在初始化阶段，`CopyOnWriteArrayList`默认给数组初始化了一个对象，当然，初始化方法还有很多，比如如下我们经常会用到的一个初始化方法，源码内容如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/1d9b35afcd814a06a575ac5ab9a66b7d.jpg)

这个方法，表示如果我们传入的是一个 `ArrayList`数组对象，会将对象内容复制一份到新的数组中，然后初始化进去，操作如下：
```java
List<String> list = new ArrayList<>();
...
//CopyOnWriteArrayList将list内容复制出来，并创建一个新的数组
CopyOnWriteArrayList<String> copyList = new CopyOnWriteArrayList<>(list);
```

`CopyOnWriteArrayList`是对原数组内容进行复制再写入，那么是不是也存在多线程下操作也会发生冲突呢？

下面我们再一起来看看它的方法实现！
### 三、常用方法
#### 3.1、添加元素
`add()`方法是`CopyOnWriteArrayList`的添加元素的入口！

`CopyOnWriteArrayList`之所以能保证多线程下安全操作， `add()`方法功不可没，源码如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/2281132eb35f4691b4bf47078d137892.jpg)

操作步骤如下：

* 1、获得对象锁；
* 2、获取数组内容；
* 3、将原数组内容复制到新数组；
* 4、写入数据；
* 5、将array数组变量地址指向新数组；
* 6、释放对象锁；

在 Java 中，独占锁方面，有2种方式可以保证线程操作安全，一种是使用虚拟机提供的`synchronized `来保证并发安全，另一种是使用`JUC`包下的`ReentrantLock`可重入锁来保证线程操作安全。

`CopyOnWriteArrayList`使用了`ReentrantLock`这种可重入锁，保证了线程操作安全，同时数组变量`array`使用`volatile`保证多线程下数据的可见行！

其他的，还有指定下标进行添加的方法，如`add(int index, E element)`，操作类似，先找到需要添加的位置，如果是中间位置，则以添加位置为分界点，分两次进行复制，最后写入数据！

#### 3.2、移除元素
`remove()`方法是`CopyOnWriteArrayList`的移除元素的入口！

源码如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/f8b45810641d4e02a24e55fbb568405b.jpg)

操作类似添加方法，步骤如下：

* 1、获得对象锁；
* 2、获取数组内容；
* 3、判断移除的元素是否为数组最后的元素，如果是最后的元素，直接将旧元素内容复制到新数组，并重新设置`array`值；
* 4、如果是中间元素，以`index`为分界点，分两节复制；
* 5、将array数组变量地址指向新数组；
* 6、释放对象锁；

当然，移除的方法还有基于对象的`remove(Object o)`，原理也是一样的，先找到元素的下标，然后执行移除操作。

#### 3.3、查询元素
`get()`方法是`CopyOnWriteArrayList`的查询元素的入口！

源码如下：

```java
public E get(int index) {
    //获取数组内容，通过下标直接获取
    return get(getArray(), index);
}
```
查询因为不涉及到数据操作，所以无需使用锁进行处理！

#### 3.4、遍历元素
上文中我们介绍到，基本都是在遍历元素的时候因为修改次数与迭代器中的修改次数不一致，导致检查的时候抛异常，我们一起来看看`CopyOnWriteArrayList`迭代器实现。

打开源码，可以得出`CopyOnWriteArrayList`返回的迭代器是`COWIterator`，源码如下：
```java
public Iterator<E> iterator() {
    return new COWIterator<E>(getArray(), 0);
}
```

打开`COWIterator`类，其实它是`CopyOnWriteArrayList`的一个静态内部类，源码如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/7d1b7f7b4dca4fa2b0219e2f1b8d7239.jpg)

可以看出，在使用迭代器的时候，遍历的元素都来自于上面的`getArray()`方法传入的对象数组，也就是传递进来的 array 数组！

**由此可见，CopyOnWriteArrayList 在使用迭代器遍历的时候，操作的都是原数组，没有像上面那样进行修改次数判断，所以不会抛异常！**

当然，从源码上也可以得出，**使用`CopyOnWriteArrayList`的迭代器进行遍历元素的时候，不能调用`remove()`方法移除元素，因为不支持此操作！**

**如果想要移除元素，只能使用`CopyOnWriteArrayList`提供的`remove()`方法，而不是迭代器的`remove()`方法，这个需要注意一下！**

### 四、总结
`CopyOnWriteArrayList`是一个典型的读写分离的动态数组操作类！

在写入数据的时候，将旧数组内容复制一份出来，然后向新的数组写入数据，最后将新的数组内存地址返回给数组变量；移除操作也类似，只是方式是移除元素而不是添加元素；而查询方法，因为不涉及线程操作，所以并没有加锁出来！

因为`CopyOnWriteArrayList`读取内容没有加锁，在写入数据的时候同时也可以进行读取数据操作，因此性能得到很大的提升，但是也有缺陷，**对于边读边写的情况，不一定能实时的读到最新的数据**，比如如下操作：
```java
public static void main(String[] args) throws InterruptedException {
    final CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
    list.add("a");
    list.add("b");
    for (int i = 0; i < 5; i++) {
        final int j =i;
        new Thread(new Runnable() {
            @Override
            public void run() {
                //写入数据
                list.add("i-" + j);
                //读取数据
                for (String str : list) {
                    System.out.println("线程-" + Thread.currentThread().getName() + "，读取内容：" + str);
                }
            }
        }).start();
    }
}
```
新建5个线程向`list`中添加元素，执行结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-collection-16/14237779b33b44378305a4d16eff578d.jpg)

**可以看到，5个线程的读取内容有差异！**

因此`CopyOnWriteArrayList`很适合读多写少的应用场景！

### 五、参考
1、JDK1.7&JDK1.8 源码

2、[掘金 - 拥抱心中的梦想 - 说一说Java中的CopyOnWriteArrayList ](https://juejin.im/post/5aaa2ba8f265da239530b69e)
