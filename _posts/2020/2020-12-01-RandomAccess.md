---
layout: post
categories: Java
title: RandomAccess 明明是个空接口，能有什么用呢？
tagline: by 子悠
tags: 
  - 子悠

---

Hello，大家好，我是阿粉，Java 语言中有很多有意思的设计，之前二哥的一篇文章中介绍了`Serializable` 空接口，今天阿粉给大家介绍一个另一个 Java 中的空接口 `RandomAccess`。

<!--more-->

## 背景

打开 JDK 源码，搜索 `RandomAccess` 我们可以看到，如下代码块，就是一个申明，里面空空如也，啥也没有。要是放在以前阿粉肯定会觉得这种写法肯定没什么用，但是毕竟看过二哥之前的文章，阿粉已经学到了，不会再那么年轻了。

```
public interface RandomAccess {
}
```

在介绍`RandomAccess` 的作用之前我们先看下集合 `List` 的两个基本的实现`ArrayList` 和`LinkedList` ，这两个的区别主要有下面几个

1. 底层数据结构不同，`Arraylist` 底层使用的是 `Object` 数组；`LinkedList` 底层使用的是双向链表数据结构；
2. 添加删除的时间复杂度不同，上面提到因为`ArrayList` 底层采用的是数组结构存储数据的，所以在插入和删除元素的时候，时间复杂度受元素位置的影响。 在执行添加方法的时候， `ArrayList` 会默认将元素添加到数组的最后，这种情况时间复杂度就是 `O(1)`。但是如果要在指定位置 `i` 插入和删除元素的话时间复杂度就为 `O(n-i)` ，因为底层的数组需要进行移动操作。 而 `LinkedList` 底层采用链表存储，对于添加元素和删除元素的时间复杂度就不会受元素位置的影响，近似 `O(1)`，如果是要在指定位置 `i` 插入和删除元素的话时间复杂度近似为 `o(n))` 因为需要先遍历移动到指定位置再插入。
3. **快速随机访问机制不同，`LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持，快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index)方法**。

## 揭秘

**正是由于 `ArrayList` 和`LinkedList` 的随机访问性不同，才有了`RandomAccess` 空接口的存在！**

看下面代码，我们创建 `ArrayList` 和` LinkedList` 两个集合，然后采用二分查找，找到指定的内容。我们顺藤摸瓜，可以发现`RandomAccess` 接口在`Collections` 类中`binarySearch` 方法中有使用，而且在给定的集合签名不同的时候会执行不同的查询方式，源码如下：

```java
public class ListTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("string1");
        list.add("string2");
        list.add("string3");
        int string2 = Collections.binarySearch(list, "string2");
        System.out.println(string2);

        LinkedList<String> linkedList = new LinkedList<>();
        linkedList.add("string4");
        linkedList.add("string5");
        linkedList.add("string6");
        int string5 = Collections.binarySearch(linkedList, "string6");
        System.out.println(string5);
    }
}

 public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key);
    else
        return Collections.iteratorBinarySearch(list, key);
}
```

可以明显的看到在这个二分查找的方法里面根据 `List` 是否实现了 `RandomAccess` 接口来决定采用哪个搜索方案，如果实现了`RandomAccess` 接口表示这个`List` 支持随机访问，所以采用的是常规的二分查找；如果没有实现`RandomAccess`接口，则采用的迭代器的二分查找，因为迭代器的二分查找的实现效率相对较低，源码如下：

```java
private static <T>
    int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
        int low = 0;
        int high = list.size()-1;

        while (low <= high) {
            int mid = (low + high) >>> 1;
            Comparable<? super T> midVal = list.get(mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }

    private static <T>
    int iteratorBinarySearch(List<? extends Comparable<? super T>> list, T key)
    {
        int low = 0;
        int high = list.size()-1;
        ListIterator<? extends Comparable<? super T>> i = list.listIterator();

        while (low <= high) {
            int mid = (low + high) >>> 1;
            Comparable<? super T> midVal = get(i, mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }

    private static <T> T get(ListIterator<? extends T> i, int index) {
        T obj = null;
        int pos = i.nextIndex();
        if (pos <= index) {
            do {
                obj = i.next();
            } while (pos++ < index);
        } else {
            do {
                obj = i.previous();
            } while (--pos > index);
        }
        return obj;
    }
```

从上面的源码我们可以看到，都是采用二分查找，但是不同的地方是，`indexedBinarySearch` 方法中使用的是`list.get(index))` 来获取中值的，而在`iteratorBinarySearch` 方法中是使用`get(i, mid);` 来获取中值的，`get(i, mid);` 中是通过`while` 遍历来查找中值的，因此效率会相对较低。

## 总结

通过上面的代码我们发现，其实`RandomAccess` 这个空接口就是用来作为标识的，标识某个集合是否支持随机访问，支持随机访问的话在进行搜索的时候我们可以采用更高效的搜索方案。

## 写在最后

最后邀请你加入我们的知识星球，这里有 1800+ 优秀的人与你一起进步，如果你是小白那你是稳赚了，很多业内经验和干货分享给你；如果你是大佬，那可以进来我们一起交流分享你的经验，说不定日后我们还可以有合作，给你的人生多一个可能。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)

