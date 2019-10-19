---
layout: post
title: 羞，Java 字符串拼接竟然有这么多姿势
tagline: by 沉默王二
categories: java
tags: 
  - java
---

就在昨天，一位叫小菜的读者微信我说：

>二哥，我今年大二，看你分享的《阿里巴巴 Java 开发手册》上有一段内容说：“循环体内，拼接字符串最好使用 StringBuilder 的 append 方法，而不是 + 号操作符。”到底为什么啊，我平常一直就用的‘+’号操作符啊！二哥有空的时候能否写一篇文章分析一下呢？

<!--more-->
我当时看到这条微信的第一感觉是：小菜你也太菜了吧，这都不知道为啥啊！我估计正在读这篇文章的你也会有同样的感觉。

但扪心自问，在我做程序员的前两年内，我也不知道为啥。遇到字符串拼接就上“+”号操作符，甭管是不是在循环体内。和小菜比起来，我当时可没他这么幸运，还有一位热心的“二哥”能够分享这份价值连城的开发手册。

既然我这么热心分享，不如好人做到底，对不对？我就认认真真地写一篇文章，给小菜解惑一下。

### 01、“+”号操作符

要说姿势，“+”号操作符必须是字符串拼接最常用的一种了，没有之一。

```java
String chenmo = "沉默";
String wanger = "王二";

System.out.println(chenmo + wanger);
```

我们把这段代码使用 JAD 反编译一下。

```java
String chenmo = "\u6C89\u9ED8"; // 沉默
String wanger = "\u738B\u4E8C"; // 王二
System.out.println((new StringBuilder(String.valueOf(chenmo))).append(wanger).toString());
```

我去，原来编译的时候把“+”号操作符替换成了 StringBuilder 的 append 方法。也就是说，“+”号操作符在拼接字符串的时候只是一种形式主义，让开发者使用起来比较简便，代码看起来比较简洁，读起来比较顺畅。算是 Java 的一种语法糖吧。

### 02、StringBuilder

除去“+”号操作符，StringBuilder 的 append 方法就是第二个常用的字符串拼接姿势了。

先来看一下 StringBuilder 类的 append 方法的源码：

```java
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
```

这 3 行代码没啥可看的，可看的是父类 AbstractStringBuilder 的 append 方法：

```java
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
```

1）判断拼接的字符串是不是 null，如果是，当做字符串“null”来处理。`appendNull` 方法的源码如下：

```java
private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
```

2）拼接后的字符数组长度是否超过当前值，如果超过，进行扩容并复制。`ensureCapacityInternal` 方法的源码如下：

```java
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}
```

3）将拼接的字符串 str 复制到目标数组 value 中。

```java
str.getChars(0, len, value, count)
```

### 03、StringBuffer

先有 StringBuffer 后有 StringBuilder，两者就像是孪生双胞胎，该有的都有，只不过大哥 StringBuffer 因为多呼吸两口新鲜空气，所以是线程安全的。

```java
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```

StringBuffer 类的 append 方法比 StringBuilder 多了一个关键字 synchronized，可暂时忽略 `toStringCache = null`。

synchronized 是 Java 中的一个非常容易脸熟的关键字，是一种同步锁。它修饰的方法被称为同步方法，是线程安全的。

### 04、String 类的 concat 方法

单就姿势上来看，String 类的 concat 方法就好像 StringBuilder 类的 append。

```java
String chenmo = "沉默";
String wanger = "王二";

System.out.println(chenmo.concat(wanger));
```

文章写到这的时候，我突然产生了一个奇妙的想法。假如有这样两行代码：

```java
chenmo += wanger
chenmo = chenmo.concat(wanger)
```

它们之间究竟有多大的差别呢？

之前我们已经了解到，`chenmo += wanger` 实际上相当于 `(new StringBuilder(String.valueOf(chenmo))).append(wanger).toString()`。

要探究“+”号操作符和 `concat` 之间的差别，实际上要看 append 方法和 concat 方法之间的差别。

append 方法的源码之前分析过了。我们就来看一下 concat 方法的源码吧。

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

1）如果拼接的字符串的长度为 0，那么返回拼接前的字符串。

```java
if (otherLen == 0) {
    return this;
}
```

2）将原字符串的字符数组 value 复制到变量 buf 数组中。

```java
char buf[] = Arrays.copyOf(value, len + otherLen);
```

3）把拼接的字符串 str 复制到字符数组 buf 中，并返回新的字符串对象。

```java
str.getChars(buf, len);
return new String(buf, true);
```

通过源码分析我们大致可以得出以下结论：

1）如果拼接的字符串是 null，concat 时候就会抛出 NullPointerException，“+”号操作符会当做是“null”字符串来处理。

2）如果拼接的字符串是一个空字符串（""），那么 concat 的效率要更高一点。毕竟不需要 `new  StringBuilder` 对象。

3）如果拼接的字符串非常多，concat 的效率就会下降，因为创建的字符串对象越多，开销就越大。

**注意了！！！**

弱弱地问一下啊，还有在用 JSP 的同学吗？EL 表达式中是不允许使用“+”操作符来拼接字符串的，这时候就只能用 `concat` 了。

```jsp
${chenmo.concat('-').concat(wanger)}
```

### 05、String 类的 join 方法

JDK 1.8 提供了一种新的字符串拼接姿势：String 类增加了一个静态方法 join。

```java
String chenmo = "沉默";
String wanger = "王二";
String cmower = String.join("", chenmo, wanger);
System.out.println(cmower);
```

第一个参数为字符串连接符，比如说：

```java
String message = String.join("-", "王二", "太特么", "有趣了");
```

输出结果为：王二-太特么-有趣了

我们来看一下 join 方法的源码：

```java
public static String join(CharSequence delimiter, CharSequence... elements) {
    Objects.requireNonNull(delimiter);
    Objects.requireNonNull(elements);
    // Number of elements not likely worth Arrays.stream overhead.
    StringJoiner joiner = new StringJoiner(delimiter);
    for (CharSequence cs: elements) {
        joiner.add(cs);
    }
    return joiner.toString();
}
```

发现了一个新类 StringJoiner，类名看起来很 6，读起来也很顺口。StringJoiner 是 `java.util` 包中的一个类，用于构造一个由分隔符重新连接的字符序列。限于篇幅，本文就不再做过多介绍了，感兴趣的同学可以去了解一下。

### 06、StringUtils.join

实战项目当中，我们处理字符串的时候，经常会用到这个类——`org.apache.commons.lang3.StringUtils`，该类的 join 方法是字符串拼接的一种新姿势。

```java
String chenmo = "沉默";
String wanger = "王二";

StringUtils.join(chenmo, wanger);
```

该方法更善于拼接数组中的字符串，并且不用担心 NullPointerException。

```java
StringUtils.join(null)            = null
StringUtils.join([])              = ""
StringUtils.join([null])          = ""
StringUtils.join(["a", "b", "c"]) = "abc"
StringUtils.join([null, "", "a"]) = "a"
```

通过查看源码我们可以发现，其内部使用的仍然是 StringBuilder。

```java
public static String join(final Object[] array, String separator, final int startIndex, final int endIndex) {
    if (array == null) {
        return null;
    }
    if (separator == null) {
        separator = EMPTY;
    }

    final StringBuilder buf = new StringBuilder(noOfItems * 16);

    for (int i = startIndex; i < endIndex; i++) {
        if (i > startIndex) {
            buf.append(separator);
        }
        if (array[i] != null) {
            buf.append(array[i]);
        }
    }
    return buf.toString();
}
```

大家读到这，不约而同会有这样一种感觉：我靠（音要拖长），没想到啊没想到，字符串拼接足足有 6 种姿势啊，晚上回到家一定要一一尝试下。

### 07、给小菜一个答复

我相信，小菜读到我这篇文章的时候，他一定会明白为什么阿里巴巴不建议在 for 循环中使用”+”号操作符进行字符串拼接了。

来看两段代码。

第一段，for 循环中使用”+”号操作符。

```java
String result = "";
for (int i = 0; i < 100000; i++) {
    result += "六六六";
}
```

第二段，for 循环中使用 append。

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100000; i++) {
    sb.append("六六六");
}
```

这两段代码分别会耗时多长时间呢？在我的 iMac 上测试出的结果是：

1）第一段代码执行完的时间为 6212 毫秒

2）第二段代码执行完的时间为 1 毫秒

差距也太特么大了吧！为什么呢？

我相信有不少同学已经有了自己的答案：第一段的 for 循环中创建了大量的 StringBuilder 对象，而第二段代码至始至终只有一个  StringBuilder 对象。

