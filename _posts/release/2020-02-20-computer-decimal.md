---
layout: post
title: 计算机基础之浮点数问题与如何精确计算小数
categories: java
tags:
	- 炭烧生蚝
---

> 下面这篇文章探讨的是关于浮点数与精确小数计算的理解。

<!--more-->

大家好啊，这次我鸭血粉丝将和大家探讨一个计算机基础问题：关于浮点数带来的问题，以及如何精确地计算小数。

为什么会想到和大家讨论这个问题呢？

这还得从我公司的项目讲起。由于我是中途加入公司项目的，没有从零开始参与项目的构建。但在开发的过程中我发现在记录存款和消费金额等与钱有关的数据时项目中用到的都是BigDecimal类。

对于BigDecimal这个类我的印象中只知道它是一个小数工具类，可以完成很多与小数相关的计算。可是基本数据类型中明明已经有了float和double两个浮点数类型，为什么还要用BigDecimal呢？

带着疑惑，我一边探索，一边写下了这篇文章。

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/decimal/1.jpg)

# 浮点数带来的问题

假如你在面试，面试官看到你有和金融，银行相关的项目的经历，可能会问你记录存款和消费金额等与钱有关的数据时使用什么数据类型？

如果你能答上来BigDecimal等小数工具类的话，他可能会继续问你：为什么放着现成的float和double不用，要使用小数工具类呢？

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/decimal/2.jpg)

这里就涉及到浮点数在计算机系统中存储的问题了。

我们可以通过自己熟悉的语言快速地重新浮点数计算不精确的现象，本文以Java为例进行展示：

```java
public class Main {
    public static void main(String[] args){
        System.out.println(0.01 + 0.09);
        System.out.println(1 - 0.32);
        System.out.println(1.015 * 100);
    }
}

//输出结果：
0.09999999999999999
0.6799999999999999
101.49999999999999
```

那么究竟为什么会出现这样的不精确现象呢？

众所周知计算机的世界是一个二进制的世界，计算机系统内部的一切数据终究会落实到0和1这两个数字上。但我们人类世界中基本都是使用十进制，要想把人类的问题交给计算机解决，就必须有一个把十进制数转换为二进制数的过程，浮点数的问题就出在这个转换的过程中。

想要把一个十进制的整数转换为二进制，简单！数学上只要对目标整数不断除以二，直到除尽或余数为1即可，这样的转换是精确的。其背后的原理是所有的整数都能通过2^n之和加上正负号来表示（n取所有正整数和0）

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/decimal/3.png)

但是想要把一个十进制的小数转换为二进制，就不那么简单了。根本原因在于2^n之和加上正负号（n取所有负整数）不能表示所有的小数！

从数学上把一个小数从十进制转换为二进制，你需要对小数不断乘以二，直到得到1才算转换完成。事实上能完成这一转换的小数不多，大多数小数才乘以二的过程中都会进入一个循环的序列，导致无法精确地把一个小数从十进制转换为二进制。

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/decimal/4.png)

# 解决浮点数问题，精确计算小数

既然我们已经知道了计算机能准确计算整数，但不能准确计算小数。那我们可以这样想：如果把小数转换为整数，让计算机计算整数，在得出精确的计算结果后再转换为小数，这样计算结果不就精确了吗？

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/decimal/5.jpg)

按照这个思路，人们封装了精确计算小数的工具类，在Java中这个工具类叫做BigDecimal，我们可以通过一次Debug验证一下是否真的是这么干的：

```java
public class Main {
    public static void main(String[] args){
        BigDecimal bd1 = new BigDecimal("123.456");
        BigDecimal bd2 = new BigDecimal("654.321");
        BigDecimal result = bd1.add(bd2);
        System.out.println(result.toString());
    }
}

//输出结果：
777.777
```

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/decimal/6.png)


参考文章：

https://zhuanlan.zhihu.com/p/71796835
https://mp.weixin.qq.com/s/Cd4uRslnek8r_a6chjwnYQ
https://www.jianshu.com/p/61044808321f
