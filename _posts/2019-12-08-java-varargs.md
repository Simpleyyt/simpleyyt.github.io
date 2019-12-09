---
layout: post
title:  灵魂拷问：Java的可变参数究竟是怎么一回事？
categories: java
tags:
	- 沉默王二

---

在逛 programcreek 的时候，我发现了一些专注基础但不容忽视的主题。比如说：Java 的可变参数究竟是怎么一回事？像这类灵魂拷问的主题，非常值得深入地研究一下。

<!--more-->

[我](https://mp.weixin.qq.com/s/feoOINGSyivBO8Z1gaQVOA)以前很不重视基础，觉得不就那么回事嘛，会用就行了。就比如说今天这个主题，管它可变不可变呢，不就是个参数嘛，还能有多大学问——抱着这种态度，我一直横行江湖近十载（苦笑）。可等到读者找我提一些基础的问题时，我几乎回答不上来，感觉知识是散的，或者是浮于表面的。幸好最近一段时间，我开始幡然醒悟，开始不放过任何一个细节，渐渐地，有点“知识储备”了。

PS：大家有什么问题，可以在《Java极客技术》星球上向我提问。

好了，牛逼吹完，让我们来步入正题。Java 的可变参数究竟是怎么一回事？

可变参数是 Java 1.5 的时候引入的功能，它允许方法使用任意多个、类型相同（`is-a`）的值作为参数。就像下面这样。

```java
public static void main(String[] args) {
    print("沉");
    print("沉", "默");
    print("沉", "默", "王");
    print("沉", "默", "王", "二");
}

public static void print(String... strs) {
    for (String s : strs)
        System.out.print(s);
    System.out.println();
}
```

静态方法 `print()` 就使用了可变参数，所以 `print("沉")` 可以，`print("沉", "默")` 也可以，甚至 3 个、 4 个或者更多个字符串都可以作为参数传递给 `print()` 方法。

说到可变参数，我想起来阿里巴巴开发手册上有这样一条规约。

![](http://www.itwanger.com/assets/images/2019/12/java-varargs-1.png)


意思就是尽量不要使用可变参数，如果要用的话，可变参数必须要在参数列表的最后一位。既然坑位有限，只能在最后，那么可变参数就只能有一个（悠着点，悠着点）。如果可变参数不在最后一位，IDE 就会提示对应的错误，如下图所示。

![](http://www.itwanger.com/assets/images/2019/12/java-varargs-2.png)



那可变参数是怎么工作的呢？

原理也很简单。**当使用可变参数的时候，实际上是先创建了一个数组，该数组的大小就是可变参数的个数，然后将参数放入数组当中，再将数组传递给被调用的方法**。

这就是为什么可以使用数组作为参数来调用带有可变参数的方法的根本原因。代码如下所示。

```java
public static void main(String[] args) {
    print(new String[]{"沉"});
    print(new String[]{"沉", "默"});
    print(new String[]{"沉", "默", "王"});
    print(new String[]{"沉", "默", "王", "二"});
}

public static void print(String... strs) {
    for (String s : strs)
        System.out.print(s);
    System.out.println();
}
```

那如果方法的参数是一个数组，然后像使用可变参数那样去调用方法的时候，能行得通吗？大家感兴趣的话，不妨试一试（行不通，嘘）。



那一般什么时候使用可变参数呢？

可变参数，可变参数，顾名思义，当一个方法需要处理任意多个相同类型的对象时，就可以定义可变参数。Java 中有一个很好的例子，就是 String 类的 `format()` 方法，就像下面这样。

```java
System.out.println(String.format("年纪是: %d", 18));
System.out.println(String.format("年纪是: %d 名字是: %s", 18, "沉默王二"));
```

PS：`%d` 表示将整数格式化为 10 进制整数，`%s` 表示输出字符串。

如果不使用可变参数，那需要格式化的参数就必须使用“+”号操作符拼接起来了。麻烦也就惹祸上身了。

在实际的项目代码中，开源包 slf4j.jar 的日志输出就经常要用到可变参数（log4j 就没法使用可变参数，日志中需要记录多个参数时就痛苦不堪了）。就像下面这样。

```java
protected Logger logger = LoggerFactory.getLogger(getClass());
logger.debug("名字是{}", mem.getName());
logger.debug("名字是{}，年纪是{}", mem.getName(), mem.getAge());
```

查看源码就可以发现，`debug()` 方法使用的可变参数。

```java
public void debug(String format, Object... arguments);
```

那在使用可变参数的时候有什么注意事项吗？

有的，有的。我们要避免重载带有可变参数的方法——这样很容易让编译器陷入自我怀疑中。

```java
public static void main(String[] args) {
    print(null);
}

public static void print(String... strs) {
    for (String a : strs)
        System.out.print(a);
    System.out.println();
}

public static void print(Integer... ints) {
    for (Integer i : ints)
        System.out.print(i);
    System.out.println();
}
```

这时候，编译器完全不知道该调用哪个 `print()` 方法，`print(String... strs)` 还是 `print(Integer... ints)`，傻傻分不清。

![](http://www.itwanger.com/assets/images/2019/12/java-varargs-3.png)

假如真的需要重载带有可变参数的方法，就必须在调用方法的时候给出明确的指示，不要让编译器去猜。

```java
public static void main(String[] args) {
    String [] strs = null;
    print(strs);

    Integer [] ints = null;
    print(ints);
}

public static void print(String... strs) {
}

public static void print(Integer... ints) {
}
```

上面这段代码是可以编译通过的。因为编译器知道[实参](http://www.itwanger.com/java/2019/11/26/java-yinyong-value.html)是 String 类型还是 Integer 类型，只不过为了运行时不抛出 `NullPointerException`，两个 `print()` 方法的内部要做好[判空](https://mp.weixin.qq.com/s/PBqR_uj6dd4xKEX8SUWIYQ)的操作。

------

好了各位读者朋友们，以上就是本文的全部内容了。**能看到这里的都是人才，二哥必须要为你点个赞**👍。如果觉得不过瘾，还想看到更多，我再给大家推荐几篇。

[灵魂拷问：Java 如何获取数组和字符串的长度？length 还是 length()？](http://www.itwanger.com/java/2019/12/08/java-array-string-length.html)
[灵魂拷问：Java 的 substring() 是如何工作的？](https://mp.weixin.qq.com/s/rLakWBPuWqYG8QT6ACetGQ)
[灵魂拷问：为什么 Java 字符串是不可变的？](https://mp.weixin.qq.com/s/CRQrm5zGpqWxYL_ztk-b2Q)
[灵魂拷问：创建 Java 字符串，用""还是构造函数](http://www.itwanger.com/java/2019/11/28/java-string-shuangyinhao-gouzaohanshu.html)



如果你有什么问题需要我的帮助，或者想喷我了，欢迎留言哟。