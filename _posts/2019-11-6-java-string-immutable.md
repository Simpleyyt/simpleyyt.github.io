---
layout: post
category: java
title: 灵魂拷问：为什么 Java 字符串是不可变的？
tagline: by 沉默王二
tag: java
---

在逛 programcreek 的时候，发现了一些精妙绝伦的主题。比如说：为什么 Java 字符串是不可变的？像这类灵魂拷问的主题，非常值得深思。


<!--more-->

对于绝大多数的初级程序员来说，往往停留在“知其然不知其所以然”的层面上——会用，但要说底层的原理，可就只能挠挠头双手一摊一张问号脸了。

很长一段时间内，我也一直处于这种层面上。导致的局面就是，我在挖一些高深点的技术方案时，往往束手无策；在读一些高深点的技术文章时，往往理解不了作者在说什么。

借此机会，我就和大家一起，对“为什么 Java 字符串是不可变的”进行一次深入地研究。注意了，准备打怪升级了！

### 01、图文分析

来看下面这行代码。

```java
String alita = "阿丽塔";
```

这行代码在字符串常量池中创建了一个内容为“阿丽塔”的对象，并将其赋值给了字符串变量 alita（存储的是字符串对象"阿丽塔"的引用）。如下图所示。

![](https://upload-images.jianshu.io/upload_images/1179389-bc9e0af38549ff9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再来看下面这行代码。

```java
String wanger = alita;
```

这行代码将字符串变量 alita 赋值给了字符串变量 wanger。这时候，wanger 和 alita 存储的是同一个字符串对象的引用。如下图所示。

![](https://upload-images.jianshu.io/upload_images/1179389-345342d059af0cea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再来看下面这行代码。

```java
alita = "战斗天使".concat(alita);
```

这行代码将字符串“战斗天使”拼接在字符串变量 alita 的前面，并重新赋值给 alita。这个过程就比之前的复杂了。我们需要先来看看 `concat()` 方法做了什么，源码如下所示。

```java
public String concat(String str) {
int otherLen = str.length();
if (otherLen == 0) {
return this;
}
int len = value.length;
char buf[] = Arrays.copyOf(value, len + otherLen);
str.getChars(buf, len);
return new String(buf, true);
}
```

可以看得出，`"战斗天使".concat(alita)` 这行代码会先在字符串常量池中创建一个新的字符串对象，内容为“战斗天使”，然后 `concat()` 方法会将其对应的字符数组和“阿丽塔”对应的字符数组复制到一个新的字符数组 buf 中，最后，再通过 new 关键字创建了一个新的字符串对象，并返回。如下图所示。

![](https://upload-images.jianshu.io/upload_images/1179389-74f4971aba320561.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图中可以得出结论，alita 此时引用的是在堆中新创建的字符串对象。

### 02、对象和对象引用

可能有些读者看完上面的图文分析没有理解反而更疑惑了：alita 不是变了吗？从“阿丽塔”变为“战斗天使阿丽塔”？怎么还说字符串是不可变的呢？

这里需要给大家解释一下，[什么是对象](http://www.itwanger.com/java/2019/11/05/java-eat-human-words.html)，什么是对象引用。

在 Java 中，由于不能直接操作对象本身，所以就有了对象引用这个概念，对象引用存储的是对象在内存中的地址。

PS：Java 虚拟机在执行程序的过程中会把内存区域划分为若干个不同的数据区域，如下图所示。

![](https://upload-images.jianshu.io/upload_images/1179389-64e7eeadb485b82e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对象存储在堆（heap）中，而对象的引用存储在栈（stack）中。

我们通常所说的“字符串是不可变的”是指“字符串对象是不可变的”。alita 是字符串对象“阿丽塔”或者“战斗天使阿丽塔”的引用。这下应该明白了吧？

### 03、源码分析

我们来看一下 String 类的部分源码。

```java
public final class String
implements java.io.Serializable, Comparable<String>, CharSequence {
/** The value is used for character storage. */
private final char value[];
}
```

可以看得出， String 类其实是通过操作字符数组 value 实现的。而 value 是 private 的，也没有提供 `serValue()` 这样的方法进行修改；况且 value 还是 final 的，意味着 value 一旦被初始化，就无法进行改变。

另外呢，String 类提供的方法，比如说 `substring()`：

```java
public String substring(int beginIndex) {
int subLen = value.length - beginIndex;
return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

`toLowerCase()`：

```java
public String toLowerCase(Locale locale) {
return new String(result, 0, len + resultOffset);
}
```

还有之前提到的 `concat()`，看似都能改变字符串的内容，但其实都是在方法内部使用 new 关键字重新创建的新字符串对象。


### 04、为什么要不可变

String 类的源码中还有一个重要的字段 hash，用来保存字符串对象的 hashCode。

```java
public final class String
implements java.io.Serializable, Comparable<String>, CharSequence {

/** Cache the hash code for the string */
private int hash; // Default to 0

public int hashCode() {
int h = hash;
if (h == 0 && value.length > 0) {
char val[] = value;

for (int i = 0; i < value.length; i++) {
h = 31 * h + val[i];
}
hash = h;
}
return h;
}
}
```

因为字符串是不可变的，所以一旦被创建，它的 hash 值就不会再改变了。由此字符串非常适合作为 HashMap 的 key 值，这样可以极大地提高效率。

另外呢，不可变对象天生是线程安全的，因此字符串可以在多个线程之间共享。

举个反面的例子，假如字符串是可变的，那么数据库的用户名和密码（字符串形式获得数据库连接）将不再安全，一些高手可以随意篡改，从而导致严重的安全问题。

### 05、最后

总结一下，字符串一旦在内存中被创建，就无法被更改。String 类的所有方法都不会改变字符串本身，而是返回一个新的字符串对象。如果需要一个可修改的字符序列，建议使用 StringBuffer 或 StringBuilder 类代替 String 类，否则每次创建的字新符串对象会导致 Java 虚拟机花费大量的时间进行垃圾回收。



参考链接：https://www.programcreek.com/2009/02/diagram-to-show-java-strings-immutability


谢谢大家的阅读，喜欢就点个赞，这将是我最强的写作动力。如果你觉得文章对你有所帮助，也蛮有趣的，就微信搜索“**沉默王二**”关注一下我的公众号。嘘，回复关键字「Java」更有好礼相送。
