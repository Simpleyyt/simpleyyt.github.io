---
layout: post
categories: 集合
title: 你确定 LinkedList 在新增/删除元素时，效率比 ArrayList 高？
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
在面试的时候都会被问到集合相关的问题，比如：你能讲讲 ArrayList 和 LinkedList 的区别吗？
<!-- more -->

那么我相信你肯定能够答上来： ArrayList 是基于数组实现的， LinkedList 是基于链表实现的

接下来面试官就会连环问了，那你能讲讲，它们都用在什么场景下吗？

阿粉知道这种程度肯定难不倒咱们读者的：因为 ArrayList 是基于数组实现的，所以在遍历的时候， ArrayList 的效率是要比 LinkedList 高的， LinkedList 是基于链表实现的，所以在进行新增/删除元素的时候， LinkedList 的效率是要比 ArrayList 高的

面试官：哦哦，好的，我大概了解了，我这边没有什么想问的了，您回去等消息可以吗

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/21-问号脸.gif)

？？？发生了什么？

哈哈，上面模拟了一个面试场景，是想引出来这篇文章的主题： LinkedList 在新增/删除元素时，效率比 ArrayList 高，这是真的吗？

我相信你也知道套路，一般这么一问，那肯定就不是真的了

放一张图片，这是经过我测试之后的真实结果

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/22-ArrayList-LinkedList.jpg)

> 因为微信不能放外链的缘故，可以在公众号后台发送 “20200821” 获取测试代码

# ArrayList 与 LinkedList 新增元素比较

从图中可以看出来， LinkedList 在新增元素时，它的效率不一定比 ArrayList 高，这是要分情况的

如果是从集合头部位置新增元素的话，那确实是 LinkedList 的效率要比 ArrayList 高

但是如果是从集合中间位置或者是尾部位置新增元素， ArrayList 效率反而要比 LinkedList 效率要高

Excuse me ？竟然和我以前学的不一样？阿粉我学的浅，你别骗我

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/23-别骗我.gif)

哈哈哈，为什么会这样呢

这是因为 ArrayList 是基于数组实现的嘛，而数组是一块连续的内存空间，所以在添加元素到数组头部时，需要对头部后面的数据进行复制重排，所以效率是蛮低的

但是 LinkedList 是基于链表实现的，在添加元素的时候，首先会通过循环查找到添加元素的位置，如果要添加的位置处于 List 前半段，那就从前向后找;如果位置在后半段，那就从后往前找，所以 LinkedList 添加元素到头部是非常高效的（小声 BB ，这我知道

哦，这你知道？看来基础蛮不错的嘛~

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/24-小伙子很可以.gif)

所以当 ArrayList 在添加元素到数组中间时，有一部分数据需要复制重排，效率就不是很高，那为啥 LinkedList 比它还要低呢？这是因为 LinkedList 把元素添加到中间位置的时候，需要在添加之前先遍历查找，这个查找的时间比较耗时

添加元素到尾部操作中， ArrayList 的效率要比 LinkedList 的还要高，这是为啥嘞

因为 ArrayList 在添加的时候不需要什么操作，直接插入就好了，所以效率蛮高的

但是 LinkedList 就不一样了，对于 LinkedList 来说，也不需要查找啥的，直接插入就可以了，但是需要 new 对象，还有变换指针指向对象呀，这些过程耗时加起来可就比 ArrayList 长了

<strong>它是有前提的，那就是 ArrayList 初始化容量是足够的情况下，才有上述的特点，如果 ArrayList 涉及到动态扩容，那它的效率肯定会降低</strong>

# ArrayList 与 LinkedList 删除元素比较

删除元素和新增元素的原理是一样的，所以删除元素的操作和新增元素的操作耗时也是很相近

这里就不再赘述

# ArrayList 与 LinkedList 遍历元素比较

测试结果非常明显，对于 LinkedList 来说，如果使用 for 循环的话，效率特别低，但是 ArrayList 使用 for 循环去遍历的话，就比较高

为啥呢？

emmm ，得从源码说起

先来看 ArrayList 的源码吧，这个比较简单


```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

能够看到， ArrayList 实现了 List ， RandomAccess ， Cloneable 还有 Serializable 接口

你是不是对 RandomAccess 这个接口挺陌生的？这是个啥？

但是通过查阅源码能够发现它也只是个空的接口罢了，那 ArrayList 为啥还要去实现它嘞

因为 RandomAccess 接口是一个标志接口，它标识着“只要实现该接口的 list 类，都可以实现快速随机访问”

实现快速随机访问？你能想到什么？这不就是数组的特性嘛！可以直接通过 index 来快速定位 & 读取

那你是不是就能想到， ArrayList 是数组实现的，所以实现了 RandomAccess 接口， LinkedList 是用链表实现的，所以它没有用 RandomAccess 接口实现吧？

beautiful ~就是这样

咱瞅瞅 LinkedList 源码

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

果然，跟咱们设想的一样，没有实现 RandomAccess 接口

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/25-机智就是我了.gif)

那为啥 LinkedList 接口使用 for 循环去遍历的时候，慢的不行呢？

咱们瞅瞅 LinkedList 在 get 元素时，都干了点儿啥

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

在 get 方法中，主要调用了 node() 方法，因为 LinkedList 是双向链表，所以 if (index < (size >> 1)) 在判断 i 是在前半段还是后半段，如果是前半段就正序遍历，如果是在后半段那就倒序遍历，那么为什么使用 for 循环遍历 LinkedList 时，会这么慢？（好像离真相越来越近了

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/26-好像知道了什么.gif)

原因就在两个 for 循环之中，以第一个 for 循环为例

- get(0) ：直接拿到了node0 地址，然后拿到 node0 的数据

- get(1) ：先拿到 node0 地址，然后 i < index ，开始拿 node1 的地址，符合条件，然后去拿 node1 的数据

- get(2) ：先拿到 node0 的地址，然后 i < index ，拿到 node1 的地址， i < index ，继续向下走，拿到 node2 的地址，符合条件，获取 node2 的数据

发现问题了嘛？我就是想要 2 的数据， LinkedList 在遍历时，将 0 和 1 也给遍历了，如果数据量非常大的话，那效率可不就唰唰的下来了嘛

那到现在，咱们也就非常明确了，如果是要遍历 ArrayList 的话，最好是用 for 循环去做，如果要遍历 LinkedList 的话，最好是用迭代器去做

我猜你一定会说，阿粉啊，那如果对方就给我传过来了一个 list ，我不知道它是 ArrayList 还是 LinkedList 呀？我该怎么办呢

还记得 ArrayList 和 LinkedList 有什么不同吗？是不是 ArrayList 实现了 RandomAccess 接口，但是 LinkedList 没有实现，所以可以从这点去着手解决

暖暖的阿粉在这里给个简单的小 demo ，可以参考下:

```java
public class ListTest {
    public static void main(String[] args) {
        List<String> arrayList = new ArrayList<String>();
        arrayList.add("aaa");
        arrayList.add("bbb");
        isUseIterator(arrayList);

        List<String> linkedList = new LinkedList<String>();
        linkedList.add("ccc");
        linkedList.add("ddd");
        isUseIterator(linkedList);
    }

    public static void isUseIterator(List list){
        if (list instanceof RandomAccess){
            System.out.println("实现了 RandomAccess 接口,使用 for 循环遍历");

            for (int i = 0 ; i < list.size(); i++ ){
                System.out.println(list.get(i));
            }
        }else{
            System.out.println("没有实现 RandomAccess 接口,使用迭代器遍历");

            Iterator it = list.iterator();
            while (it.hasNext()){
                System.out.println(it.next());
            }
        }
    }
}
```

> 本篇文章用到的所有代码，都上传到了 github 上，因为微信不能放外链的缘故，可以在公众号后台发送 “20200821” 获取本文完整代码

所以，乖，下次面试官再问你 LinkedList 在新增/删除元素时，效率比 ArrayList 高吗，不要再傻傻的回答是了，拿阿粉这篇文章和他扯皮，保证没问题

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/27-乖听话.gif)

参考:

极客时间 — 《Java 性能调优实战》