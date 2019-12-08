---
layout: post  
title: Java Grammar：几道面试题学习String
tagline: by Vi的技术博客
categories: 面试  
tags: 
    - Vi的技术博客

---

几道面试题学习String
<!--more-->
### 字符串介绍



`String`类是`java.lang`包中的一个类，是我们日常中使用的非常多的一个类，它不是基础数据类型，底层实现是字符数组来实现的：

```java
/** The value is used for character storage. */
    private final char value[];
```

`String`类是由`final`修饰的，所以是无法被继承的，一旦创建了`String`对象，我们就无法改变它的值。因此，**它是线程安全的**，可以安全地用于多线程环境中。

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence
```



下面我们通过几道面试题来学习`String`类



### 如何创建一个字符串

一般来说有三种：

- 通过`new`关键字通过构造方法去创建
- 通过双引号`“”`
- 通过字符串连接符`+`和其余字符串进行拼接创建



### 说说这几种的区别



1. 当通过`new`关键字调用无参构造时，仅仅在JVM的堆内存中创建了一个对象

1. 通过`""`创建对象的时候，如果字符串常量池存在该字符串，直接返回该字符串对象在字符串常量池的地址，否则创建一个新的字符串对象并存储在字符串常量池。



### String s = new String("a") 创建了几个对象

当通过`new`关键字传入双引号字符串参数时，会先去把该双引号的字符串放入字符串常量池，然后遇到new以后会在堆中再次创建一个字符串对象，这里是创建了两个对象。



![](http://www.justdojava.com/assets/images/2019/java/image_vi/08_19/2019-08-17-081144.png)



### + 的实现原理

```java
String s1 = null;
String s2 = "abc";
System.out.println(s1 + s2);
```

借这道面试题来聊一下+的原理，这道题的答案是”nullabc“，也许会有些奇怪，但是当你了解了`+`的原理后也许就不会感到奇怪了，我们使用`javap`命令去看一下编译器那里把`+`编译成了什么？

![](http://www.justdojava.com/assets/images/2019/java/image_vi/08_19/2019-08-17-085354.png)

我们在图中被红色框柱的部分可以看出，`+`的执行的过程其实就是先把 `String`转换成了`StringBuilder`后调用`append`方法完成拼接后再调用`toString`方法完成字符串的拼接。所以上面的代码也可以转换为

```java
StringBuilder s1 = new StringBuilder(String.valueOf(null));
StringBuilder s2 = new StringBuilder("abc");
s1.append(s2).toString();
```



### 关于StringBuilder和StringBuffer



`StringBuffer` 和 `StringBuilder` 二者都继承了 `AbstractStringBuilder` ，底层都是利用可修改的char数组(JDK 9 以后是 byte数组)。两者的区别是`StringBuilder`是线程不安全的，而`StringBuffer`是线程安全的。性能上来说，`StringBuilder`要高于`StringBuffer`。

在单线程情况下，如有大量的字符串操作情况，不能使用`String`来拼接而是使用，避免产生大量无用的中间对象，耗费空间且执行效率低下（新建对象、回收对象花费大量时间）。这时就需要用到我们的`StringBuilder`。

而在多线程情况下，应当使用`StringBuffer`来保证线程的安全~



### 判空



在日常的开发中，我们经常会遇到判断字符串是否为空的需求，这里安利几个工具类中的写法：



```java
// 来自apache下的lang3包中的StringUtils
import org.apache.commons.lang3.StringUtils
....
  
  
  //这里是判断是否为null或为空
  String s;
  StringUtils.isNotEmpty(s);

	//这里是用于判断是否为null或为空，或空格，Tab这样的占用符
	StringUtils.isNotBlank(s);
```



### 是否相等

关于两个字符串是否相等，我用的最多的是`java.util`包下的`Objects`类中的方法 ，实现方法如下：
```java
public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
}
```

用法也很简单:
```java
Objects.equals(a,b);
```



### 后面

对于新手可能会对本片中涉及到JVM的部分有些疑问，欢迎关注本人正在连载更新的**每日五分钟，玩转JVM系列**文章~


