---
layout: post
categories: 数据结构与算法
title: 阿粉教你这样解锁单链表环的操作
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---

临近假期，很多人都放松了学习，阿粉一阵激动，这可是超越别人的好机会啊，赶紧去补一补数据结构和算法方面的内容。

今天阿粉教你这样解锁单链表环的操作，让你面试手写代码再也不怕！

说到单链表，肯定会想到 5 种经典操作，饭要一口一口吃，算法要一个一个拿下。今天先来讲讲在单链表中环的操作。

# 判断链表中是否有环

先来看一张图：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/linklist.jpg)

我们能够清楚的看到，在这个单链表中，是有环的。那么使用代码，该如何判断它是否有环呢？

先别着急看代码，先和阿粉一起来分析一下，有了思路，代码实现相对就比较容易。

> 判断链表中是否有环，可以从头结点开始，依次遍历单链表中的每一个节点。
每遍历一个节点，就和前面的所有节点作比较，如果发现新节点和之前的某个节点相同，则说明此节点被遍历过两次，说明链表有环，反之就是没有。

但是仔细看一下这种方法，你会发现这种方法很耗时耗力，因为每遍历一个节点，都要把它和前面所有的节点都比较一遍。
别着急，阿粉还有一个很巧妙的方法，就是使用两个指针。这样思路就可以这样想：

> 使用两个指针，一个快指针，一个慢指针。
快指针每次走 2 步，慢指针每次走 1 步。
如果链表中没有环，则快指针会先指向 null
如果链表中有环，则快慢指针一定会相遇

思路打通，咱们就可以使用代码来进行实现了：
```java
/**
 * 判断链表是否有环
 */
public class IsHasLoop {
    public static class Node{
        private int data;
        private Node next;
        public Node(int data,Node next){
            this.data=data;
            this.next=next;
        }
        public int getData(){
            return data;
        }
    }
    public static void main(String[] args){
        // 初始化单链表
        Node node5=new Node(5,null);
        Node node4=new Node(4,node5);
        Node node3=new Node(3,node4);
        Node node2=new Node(2,node3);
        Node node1=new Node(1,node2);
        // 让 node5 的指针指向 node1 形成一个环
        node5.next=node1;

        boolean flag=isHasLoop(node1);
        System.out.println(flag);

    }
    public static boolean isHasLoop(Node list){
        if (list == null){
            return false;
        }

        Node slow=list;
        Node fast=list;

        while (fast.next != null && fast.next.next != null){
            // 慢指针走一步,快指针走两步
            slow=slow.next;
            fast=fast.next.next;
            // 如果快慢指针相遇,则说明链表中有环
            if (slow==fast){
                return true;
            }
        }
		// 反之链表中没有环
        return false;
    }
}
```

# 求环长

现在判断链表中是否有环这个问题已经解决了，阿粉觉得不能到此为止，思路就向外扩散了一下，既然有环了，如果想要求环长该怎么办呢？

既然快慢指针相遇了，阿粉记录下此时的位置，接下来再让满指针继续向前走，每次走 1 步，这样当慢指针再次走到相遇时的位置时，慢指针走过的长度不就是环长嘛

```java
public static int getLength(Node list){
        // 定义环长初始值为 0
        int loopLength=0;
        Node slow=list;
        Node fast=list;

        while (fast != null && fast.next != null) {
            // 慢指针走一步,快指针走两步
            slow=slow.next;
            fast=fast.next.next;

            // 第一次相遇时跳出循环
            if (slow == fast) break;
        }
        // 如果 fast next 指针首先指向 null 指针,说明该链表没有环,则环长为 0
        if(fast.next == null || fast.next.next == null){
            return 0;
        }
        // 如果有环,使用临时变量保存当前的链表
        Node temp = slow;
        // 让慢指针一直走,直到走到原来位置
        do{
            slow = slow.next;
            loopLength++;
        } while(slow != temp);

        return loopLength;
}
```

# 求入环点

既然有环了，也求出了环长，那么入环点应该也知道了吧？
阿粉对于求入环点这个问题有点儿懵，就画了一张图出来：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/DataStructures&Algorithms/chain.jpg)

如上图，我们假设：

> 入环点距离头结点距离为 D
入环点与首次相遇点较短的距离为 S1
入环点与首次相遇点较长的距离为 S2
当两个指针首次相遇时，慢指针一次只走 1 步，则它所走的距离为： D+S1
快指针每次走 2 步，多走了 n(n>=1) 圈，则它所走的距离为： D+S1+n(S1+S2)
快指针速度为慢指针的 2 倍，则： 2(D+S1)=D+S1+n(S1+S2)
上面等式，整理可得： D=(n-1)(S1+S2)+S2

如果让 (n-1)(S1+S2) 为 0 ，是不是 D 和 S2 就相等了？也就是说，当两个指针第一次相遇时，只要把其中一个指针放回到头结点位置，另外一个指针保持在首次相遇点，接下来两个指针每次都向前走 1 步，接下来这两个指针相遇时，就是要求的入环点。

有点儿像做数学题的感觉，还好阿粉的数学功底还是有那么一丢丢的。

基于上面的思路，代码就很容易实现了：

```java
public static Node entryNodeOfLoop(Node list){
        Node slow=list;
        Node fast=list;
        while(fast.next != null && fast.next.next != null){
            // 慢指针走一步,快指针走两步
            slow=slow.next;
            fast=fast.next.next;

            // 第一次相遇时跳出循环
            if (slow == fast) break;
        }
        // 如果 fast next 指针首先指向 null 指针,说明该链表没有环,则入环点为 null
        if (fast.next == null || fast.next.next == null){
            return  null;
        }
        // 第一次相遇之后,让一个指针指向头结点,另外一个指针在相遇位置
        // 两个指针每次走 1 步,相遇为止,此时相遇节点即为入环点
        Node head=list; // 头结点
        Node entryNode=slow;    // 相遇节点
        while (entryNode != head){
            entryNode=entryNode.next;
            head=head.next;
        }
        return entryNode;
}
```

关于单链表环的操作，阿粉掌握的就是这些了。

最后总结一下：不管是求环长，还是找到入环点，最关键的就是找到第一次相遇时所在的位置，找到了这个位置，那么接下来的问题就比较容易解决了。

参考：
- 漫画算法：小灰的算法之旅