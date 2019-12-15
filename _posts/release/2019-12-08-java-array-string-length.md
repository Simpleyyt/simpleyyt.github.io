---
layout: post
title:  灵魂拷问：Java 如何获取数组和字符串的长度？length 还是 length()？
categories: java
tags:
	- 沉默王二

---

限时 1 秒钟给出答案，来来来，听我口令：“Java 如何获取数组和字符串的长度？length 还是 length()？”

<!--more-->

在逛 programcreek 的时候，我发现了上面这个主题。说实话，我当时脑海中浮现出了这样一副惊心动魄的画面：

面试官老马坐在我的对面，地中海式的发型令我敬佩有加。尽管略显疲惫，但他仍然自信地向我抛出了上面这个问题。稍稍迟疑了一下，我回答说：“数组用 length，字符串用 length 跟上小括号”。老马不愧是面试中的高手，一瞬间就从我的回答中捕获到了不自信。我能感受得出来，因为我看到老马的嘴角微微地动了一下，似乎想要咂咂嘴。但出于对于我的礼貌，他克制住了。

到底该用 length 还是 length()，说真的，我当时真有点吃不准，怀念 IDE 的代码自动提醒功能啊！

```java
int[] arr = new int[4];
System.out.println(arr.length);// 获取数组的长度

String str = "沉默王二";
System.out.println(str.length());// 获取字符串的长度
```

按理说，[数组](http://www.itwanger.com/java/2019/11/08/java-array.html)和[字符串](http://www.itwanger.com/java/2019/11/08/java-string-join.html)都是对象，访问长度都用 `length()` 方法就好了。为什么数组偏偏剑走偏锋用的 `length` 字段呢？

首先呢，我们必须要明白：数组是一个容器，当它被创建后，不仅元素的类型是确定的，元素的个数也是确定的。换句话说，数组的长度是确定的，不可能再变长或者变短。因此，数组可以使用一个字段（length）来表示长度。

创建数组的方法有两种，这个应该大家都知道了。一种是通过 `new` 关键字创建指定长度后再赋值，另外一种是通过 `{}` 直接进行初始化。

```java
// new
int[] arr = new int[4];
arr[0] = 0;
arr[1] = 1;
arr[2] = 2;
arr[3] = 3;

// {}
int [] arr1 = {0, 1, 2, 3};
```

但不管用哪种方法，数组的长度是可以明确知道的。并且不会再变长或者变短（学不了孙悟空的金箍棒）。

由于数组也是对象，所以以下代码是合法的。

```java
Object arr2 = new int[4];
```

这就意味着数组继承了超类 `java.lang.Object` 的所有成员方法和字段。事实上，的确如此，我们可以通过以下代码来获取数组的类型信息 Class。

```java
Object arr2 = new int[4];
System.out.println(arr2.getClass());

Object arr3 = new String[4];
System.out.println(arr3.getClass());
```

输出的结果会是什么呢？

```
class [I
class [Ljava.lang.String;
```

`class [I` 表示一个“int 类型数组”在运行时的对象类型信息；`class [Ljava.lang.String;` 表示一个“字符串类型数组”在运行时的对象类型信息。

那为什么数组不单独定义一个类来表示呢？就像字符串 String 类那样呢？

一个合理的解释是 Java 将其隐藏了。假如真的存在一个 Array.java，我们也可以假想它真实的样子，它必须要定义一个容器来存放数组的元素，就像 String 类那样。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

但这样做真的有必要吗？为数组单独定义一个类，是不是有点画蛇添足的意味。那既然数组没必要定义成一个类，也就没有必要再定义一个 `length()` 方法来获取数组的长度了，直接用 `length` 这个字段就可以了，不是吗？

那为什么字符串 String 类会有 `length()` 方法呢？来看一下源码就明白了。

```java
    /**
     * Returns the length of this string.
     * The length is equal to the number of Unicode
     * code units in the string.
     */
    public int length() {
        return value.length;
    }
```

`length()` 方法返回的正是字符数组 value 的长度（length），value 本身是 private 的，因此很有必要为 String 类提供一个 public 级别的方法来供外部访问字符的长度。

总结一下，Java 获取数组长度的时候用 `length`，获取字符串长度的时候用的是 `length()`，他们之间的区别我相信大家已经搞清楚了。

最后提醒一点：万丈高楼平地起。一栋楼能盖多高，一座大桥能造多长，重要的是它们的地基。同样对于我们技术人员来说，基础知识越扎实，走得就会越远。