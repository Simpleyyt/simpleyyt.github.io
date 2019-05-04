---
layout: post
title: Java：优雅地处理异常真是一门学问啊！
tagline: by 沉默王二
categories: java
tag:
    - Java 异常
---

![](https://upload-images.jianshu.io/upload_images/1179389-a83c3620131690fa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 01、

你有没有这样的印象，当你想要更新一款 APP 的时候，它的更新日志里总有这么一两句描述：

*   修复若干 bug
*   杀了某程序员祭天，并成功解决掉他遗留的 bug

<!--more-->

作为一名负责任的程序员，我们当然希望程序不会出现 bug，因为 bug 出现的越多，间接地证明了我们的编程能力越差，至少领导是这么看的。

事实上，领导是不会拿自己的脑袋宣言的：“我们的程序绝不存在任何一个 bug。”但当程序出现 bug 的时候，领导会毫不犹豫地选择让程序员背锅。

为了让自己少背锅，我们可以这样做：

*   在编码阶段合理使用异常处理机制，并记录日志以备后续分析
*   在测试阶段进行大量有效的测试，在用户发现错误之前发现错误

还有一点需要做的是，在敲代码之前，学习必要的编程常识，做到兵马未动，粮草先行。

### 02、

在 Java 中，异常（Throwable）的层次结构大致如下。

![](https://upload-images.jianshu.io/upload_images/1179389-0f8a932f9faee710.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Error 类异常描述了 Java 运行时系统的内部错误，比如最常见的 `OutOfMemoryError` 和 `NoClassDefFoundError`。

导致 `OutOfMemoryError` 的常见原因有以下几种：

- 内存中加载的数据量过于庞大，如一次从数据库取出过多数据；
- 集合中的对象引用在使用完后未清空，使得 JVM 不能回收；
- 代码中存在死循环或循环产生过多重复的对象；
- 启动参数中内存的设定值过小；

`OutOfMemoryError` 的解决办法需要视情况而定，但问题的根源在于程序的设计不够合理，需要通过一些性能检测才能找得出引发问题的根源。

导致 `NoClassDefFoundError` 的原因只有一个，Java 虚拟机在编译时能找到类，而在运行时却找不到。

![](https://upload-images.jianshu.io/upload_images/1179389-0c3098c62ca6a9e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`NoClassDefFoundError` 的解决办法，我截了一张图，如上所示。当一个项目引用了另外一个项目时，切记这一步！

Exception（例外）通常可分为两类，一类是写代码的人造成的，比如访问空指针（`NullPointerException`）。应当在敲代码的时候进行检查，以杜绝这类异常的发生。

```java
if (str == null || "".eqauls(str)) {
}
```

另外一类异常不是写代码的人造成的，要么需要抛出，要么需要捕获，比如说常见的 `IOException`。

抛出的示例。

```java
public static void main(String[] args) throws IOException {
	InputStream is = new FileInputStream("沉默王二.txt");
	int b;
	while ((b = is.read()) != -1) {

	}
}
```

捕获的示例。

```java
public static void main(String[] args) {
	try {
		InputStream is = new FileInputStream("沉默王二.txt");
		int b;
		while((b = is.read()) != -1) {
			
		}
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

### 03、

当抛出异常的时候，剩余的代码就会终止执行，这时候一些资源就需要主动回收。Java 的解决方案就是 `finally` 子句——不管异常有没有被捕获，`finally` 子句里的代码都会执行。

在下面的示例当中，输入流将会被关闭，以释放资源。

```java
public static void main(String[] args) {
	InputStream is = null;
	try {
		is = new FileInputStream("沉默王二.txt");
		int b;
		while ((b = is.read()) != -1) {}
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		is.close();
	}
}
```

但我总觉得这样的设计有点问题，因为 `close()` 方法同样会抛出 `IOException`：

```java
    public void close() throws IOException {}
```

也就是说，调用 `close()` 的 main 方法要么需要抛出 `IOException`，要么需要在 `finally` 子句里重新捕获 `IOException`。

选择前一种就会让 `try catch` 略显尴尬，就像下面这样。

```java
public static void main(String[] args) throws IOException {
	InputStream is = null;
	try {
		is = new FileInputStream("沉默王二.txt");
		int b;
		while ((b = is.read()) != -1) {}
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		is.close();
	}
}
```

选择后一种会让代码看起来很臃肿，就像下面这样。

```java
public static void main(String[] args) {
	InputStream is = null;
	try {
		is = new FileInputStream("沉默王二.txt");
		int b;
		while ((b = is.read()) != -1) {}
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		try {
			is.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```

总之，我们需要另外一种更优雅的解决方案。JDK7 新增了 `Try-With-Resource` 语法：如果一个类（比如 `InputStream`）实现了 `AutoCloseable` 接口，那么就可以将该类的对象创建在 `try` 关键字后面的括号中，当 `try-catch` 代码块执行完毕后，Java 会确保该对象的 `close`方法被调用。示例如下。

```java
public static void main(String[] args) {
	try (InputStream is = new FileInputStream("沉默王二.txt")) {
		int b;
		while ((b = is.read()) != -1) {
		}
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

### 04、

关于异常处理机制的使用，我这里总结了一些非常实用的建议，希望你能够采纳。

**1）尽量捕获原始的异常**。

实际应该捕获 `FileNotFoundException`，却捕获了泛化的 `Exception`。示例如下。

```java
InputStream is = null;
try {
	is = new FileInputStream("沉默王二.txt");
} catch (Exception e) {
	e.printStackTrace();
}
```

这样做的坏处显而易见：假如你喊“王二”，那么我就敢答应；假如你喊“老王”，那么我还真不敢答应，万一你喊的我妹妹“王三”呢？

很多初学者误以为捕获泛化的 `Exception` 更省事，但也更容易让人“丈二和尚摸不着头脑”。相反，捕获原始的异常能够让协作者更轻松地辨识异常类型，更容易找出问题的根源。

**2）尽量不要打印堆栈后再抛出异常**

当异常发生时打印它，然后重新抛出它，以便调用者能够适当地处理它。就像下面这段代码一样。

```java
public static void main(String[] args) throws IOException {
	try (InputStream is = new FileInputStream("沉默王二.txt")) {
	}catch (IOException e) {
		e.printStackTrace();
		throw e;
	} 
}
```

这似乎考虑得很周全，但是这样做的坏处是调用者可能也打印了异常，重复的打印信息会增添排查问题的难度。

```java
java.io.FileNotFoundException: 沉默王二.txt (系统找不到指定的文件。)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at java.io.FileInputStream.<init>(FileInputStream.java:93)
	at learning.Test.main(Test.java:10)
Exception in thread "main" java.io.FileNotFoundException: 沉默王二.txt (系统找不到指定的文件。)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at java.io.FileInputStream.<init>(FileInputStream.java:93)
	at learning.Test.main(Test.java:10)
```

**3）千万不要用异常处理机制代替判断**

我曾见过类似下面这样奇葩的代码，本来应该判 `null` 的，结果使用了异常处理机制来代替。

```java
public static void main(String[] args) {
	try {
		String str = null;
		String[] strs = str.split(",");
	} catch (NullPointerException e) {
		e.printStackTrace();
	}
}
```

捕获异常相对判断花费的时间要多得多！我们可以模拟两个代码片段来对比一下。

代码片段 A：

```java
long a = System.currentTimeMillis();
for (int i = 0; i < 100000; i++) {
	try {
		String str = null;
		String[] strs = str.split(",");
	} catch (NullPointerException e) {
	}
}
long b = System.currentTimeMillis();
System.out.println(b - a);
```

代码片段 B：

```java
long a = System.currentTimeMillis();
for (int i = 0; i < 100000; i++) {
	String str = null;
	if (str != null) {
		String[] strs = str.split(",");
	}
}
long b = System.currentTimeMillis();
System.out.println(b - a);
```

100000 万次的循环，代码片段 A（异常处理机制）执行的时间大概需要 1983 毫秒；代码片段 B（正常判断）执行的时间大概只需要 1 毫秒。这样的比较虽然不够精确，但足以说明问题。

**4）不要盲目地过早捕获异常**

如果盲目地过早捕获异常的话，通常会导致更严重的错误和其他异常。请看下面的例子。

```java
InputStream is = null;
try {
	is = new FileInputStream("沉默王二.txt");

} catch (FileNotFoundException e) {
	e.printStackTrace();
}

int b;
try {
	while ((b = is.read()) != -1) {
	}
} catch (IOException e) {
	e.printStackTrace();
}

finally {
	try {
		is.close();
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

假如文件没有找到的话，`InputStream` 的对象引用 is 就为 `null`，新的 `NullPointerException` 就会出现。

```java
java.io.FileNotFoundException: 沉默王二.txt (系统找不到指定的文件。)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at java.io.FileInputStream.<init>(FileInputStream.java:93)
	at learning.Test.main(Test.java:12)
Exception in thread "main" java.lang.NullPointerException
	at learning.Test.main(Test.java:28)
```

`NullPointerException` 并不是程序出现问题的本因，但实际上它出现了，无形当中干扰了我们的视线。正确的做法是延迟捕获异常，让程序在第一个异常捕获后就终止执行。

### 05、

好了，关于异常我们就说到这。异常处理是程序开发中必不可少的操作之一，但如何正确优雅地对异常进行处理却是一门学问，好的异常处理机制可以确保程序的健壮性，提高系统的可用率。
