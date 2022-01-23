---
layout: post
categories: 数据结构与算法
title: 一篇文章教你搞定递归单链表反转
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---

关于单链表反转，阿粉以前写过一篇文章，是用迭代法实现的，还有一种方法是使用递归来实现的，阿粉一直没敢写，因为害怕讲不清楚。但是不能因为害怕讲不清楚就不写了，对不对。

所以这篇文章来使用递归来实现一下，并且尝试将里面的细节一一剖出来，不废话。

<!--more-->

首先，咱们要先明确，什么是递归。递归就是自己调用自己对吧。比如:有一个函数为 ` f(n) = f(n-1) * n ` ，(注意，我这里是举例子，这个函数没有给出递归的结束条件)给 n 赋值为 5 ， 则:

```java
--> f(5)
--> 5 * f(4)
--> 5 * ( 4 * f(3))
--> 5 * ( 4 * (3 * f(2)))
--> 5 * ( 4 * ( 3 * ( 2 * f (1))))
--> 5 * ( 4 * ( 3 * ( 2 * 1)))
--> 5 * ( 4 * ( 3 * 2))
--> 5 * ( 4 * 6 )
--> 5 * 24
--> 120
```

在看完例子之后，咱们接下来不 BB ，直接 show code：

```java
/**
 * 单链表反转---递归实现
 */
public class ReverseSingleList {
    public static class Node{
        private int data;
        private Node next;

        public Node( int data , Node next){
            this.data = data;
            this.next = next;
        }

        public int getData(){return  data;}
    }
    public static void main(String[] args){
        // 初始化单链表
        Node node5 = new Node(5,null);
        Node node4 = new Node(4,node5);
        Node node3 = new Node(3,node4);
        Node node2 = new Node(2,node3);
        Node node1 = new Node(1,node2);

        // 调用反转方法
        Node recursiveList = recursiveList(node1);
        System.out.println(recursiveList);
    }
    /**
     *递归实现单链表反转
     * @param list 为传入的单链表
     */
    public static Node recursiveList(Node list){
        // 如果链表为空 或者 链表中只有一个节点,直接返回
        // 也是递归结束的条件
        if (list == null || list.next == null) return list;
        Node recursive = recursiveList(list.next);
        // 将 list.next.next 指针指向当前链表 list
        list.next.next = list ;
        // 将 list.next 指针指向 null
        list.next = null;
        // 返回反转之后的链表 recursive
        return recursive;
    }
}

```

经过上面的代码，应该能够看到核心代码就是，递归实现单链表反转部分的那 5 行代码，别小看了这 5 行代码，想要真正弄清楚还真的挺不容易的。

我把这 5 行代码贴在这里，咱们一行行分析，争取看完这篇博客就能懂~(注释我就去掉了，咱们专心看这几行核心代码)

```java
if (list == null || list.next == null) return list;
Node recursive = recursiveList(list.next);
list.next.next = list ;
list.next = null;
return recursive;
```

第一行就是一个判断，条件不满足，那就往下走，第二行是自己调用自己，程序又回到第一行，不满足条件程序向下执行，自己调用自己

就这样循环到符合条件为止，那么什么时候符合条件呢?也就是 ` list == null ` 或者 ` list.next == null ` 时，看一下自己定义的链表是 ` 1->2->3->4->5->null ` ，所以符合条件时，此时的链表为 `5->null ` ，符合条件之后，程序继续向下执行，在执行完 ` Node recursive = recursiveList(list.next); ` 这行代码之后，咱们来看一下此时的程序执行结果:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveOne.jpg)


我把上面这个给画出来(阿粉的画工不是不好，不要在乎它的美丑~)

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveTwo.jpg)

接下来程序该执行 ` list.next.next = list ` 执行结束之后，链表大概就是这个样子:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveThree.jpg)

那是图，下面是程序断点调试程序的结果，发现和上面的图是一样的:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveFour.jpg)

程序继续向下走 ` list.next = null ` ，也就是说，将 list 的 next 指针指向 null :

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveFive.jpg)

从图中看到， list 为 ` 4->null ` ， recursive 为 `5->4->null ` ，咱们来看看程序的结果，是不是和图相符:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveSix.jpg)

完全一样有没有！

OK ，还记得咱们刚开始的递归函数例子嘛?现在执行完毕，开始执行下一次，咱们继续来看，此时的链表是这个样子的:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveSeven.jpg)

接下来程序执行的代码就是四行了:

```java
Node recursive = recursiveList(list.next);
list.next.next = list ;
list.next = null;
return recursive;
```

继续执行程序，咱们来看结果，将 ` list.next.next = list ` 运行结束时，此时链表为:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveEight.jpg)

从图中能够看到，链表 list 为 ` 3->4->3->4 ` 循环中， recursive 为 ` 5->4->3->4->3` 循环，咱们看一下程序是不是也是如此(在这里我截了两个循环作为示例):

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveNine.jpg)

接下来程序执行 ` list.next = null ` ，执行完毕之后，就是将 list 的 next 指针指向 null :

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveTen.jpg)
 
从图中能够看出来， list 为 ` 3->null ` ， recursive 为 ` 5->4->3->null ` ，上图看看实际结果和分析的是否一致:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/recursiveEleven.jpg)
  
说明什么？！说明咱们上面的分析是正确的。接下来的程序分析，读者们就自行研究吧，相信接下来的分析就难不倒咱们聪明的读者啦~
 
# 反转单链表的前 N 个节点

OK ，咱们趁热打铁一下，刚刚是通过递归实现了整个单链表反转，那如果我只是想反转前 N 个节点呢?

比如单链表为 `1->2->3->4->5->null ` ，现在我只想反转前三个节点，变为 ` 3->2->1->4->5->null `

有没有想法?

咱们进行整个单链表反转时，可以理解为传递了一个参数 n ，这个 n 就是单链表的长度，然后递归程序不断调用自己，然后实现了整个单链表反转。
那么，如果我想要反转前 N 个节点，是不是传递一个参数 n 来解决就好了?

咱们就直接上代码了:

```java
    /**
     *反转单链表前 n 个节点
     * @param list 为传入的单链表 , n 为要反转的前 n 个节点
     */
    public static Node next;
    public static Node reverseListN(Node list,int n){
        if (n == 1) {
            // 要进行反转链表时,先将 list 后的节点数据保存到 next 中
            next = list.next;
            return  list;
        }

        Node reverse = reverseListN(list.next , n-1);
        list.next.next = list;
        // 将 list.next 的指针指向没有进行反转的链表
        list.next = next ;
        return reverse;
    }

```

# 反转单链表的一部分

既然反转整个单链表实现了，反转前 N 个节点实现了，那么如果有个需求是反转其中的一部分数据呢?大概就是这样，原来的链表为 ` 1->2->3->4->5->null ` ，反转其中的一部分，使反转后的链表为 ` 1->4->3->2->5->null `

借用反转前 N 个节点的思路，是不是我传两个参数进来，一个是开始反转的节点，一个是结束反转的节点，然后递归操作就可以了?

瞅瞅代码是怎么写的:

```java
    /**
     *反转部分单链表
     * @param list 为传入的单链表, m 为开始反转的节点, n 为结束的反转节点
     */
    public static Node reverseBetween(Node list , int m , int n){
        if (m == 1){
            return reverseListN(list,n);
        }
        list.next = reverseBetween(list.next,m-1,n-1);
        return list;
    }
```

终于给弄清楚了
最后两个例子，读者们可以自行研究，我这里因为篇幅的问题就不进行解析了，如果第一个例子自己能够剖析清楚，下面两个也没啥大问题~

其中实现的思路借鉴了网上，真是太巧妙了，分享给大家。

最后，看到阿粉又是调试程序又是画图来帮助大家理解的份上，点点在看支持一下？
