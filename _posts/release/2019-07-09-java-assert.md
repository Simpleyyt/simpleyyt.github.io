---
layout: post
title: Java断言，一个被遗忘的关键字！
tagline: by 炸鸡可乐
categories: Java
tags: 
  - Java
---

在实际的开发过程中，几乎很少接触到java的assert，它是个啥呢，今天小编带大家一起来了解一下！

<!--more-->

### 01、assert是个啥？
> **断言是为了方便调试程序，并不是发布程序的组成部分。理解这一点是很关键的。**

在C和C++语言中都有assert关键字，表示断言。

java也不例外，在Java SE 1.4版本以后也增加了断言的特性。

默认情况下，JVM是关闭断言的。因此如果想使用断言调试程序，需要手动打开断言功能。

在命令行模式下运行Java程序时可增加参数-enableassertions或者-ea打开断言。

也可通过-disableassertions或者-da关闭断言(默认情况,可有可无)。
### 02、断言使用
> 断言是通过关键字assert来定义的，一般的，它有两种形式。

* 2.1、 `assert <boolean表达式>`

> 如果boolean表达式为true，则程序继续执行。如果为false，则程序抛出AssertionError，并终止执行。

例如：
```
public class AssertTest {
 
	public static void main(String[] args) {
		boolean isOk = false;
		assert isOk;
		System.out.println("断言通过!");
	}
}
```
直接运行，是直接通过的，因为JVM是关闭断言的！
但是，我们可以通过命令模式运行，带参数`-ea`！
```
java -ea AssertTest
```
比如Eclipse，可这样设置: Run as -> Run Configurations -> Arguments -> VM arguments：敲入-ea即可。
![](http://www.justdojava.com/assets/images/2019/java/image-jay/ef945e1f09144c6b9a617777b898991e.jpg)

运行结果：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/d297ca8ce35244769da95c8564cd232a.jpg)
* 2.2、 `assert <boolean表达式> : <错误信息表达式>`

> 如果boolean表达式为true，则程序继续执行。如果为false，则程序抛出java.lang.AssertionError，并输入错误信息表达式。

例如：
```
public class AssertTest2 {
	 
	public static void main(String[] args) {
		boolean isOk = false;
		assert isOk : "不通过！";
		System.out.println("断言通过!");
	}
}
```
同样，我们可以通过命令模式运行，带参数`-ea`！
在`eclipse`里面配置好参数，运行结果：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/07ccc4916aa447509aa690545995c46a.jpg)

### 03、陷阱
有的同学，可能觉得`assert`类似`if`判断，所以呢，就可以在代码中使用！

比如考虑下面这个简单的例子：
```
public class AssertTest2 {
	 
	public static void main(String[] args) {
		int[] is = {1};
		assert(is.length > 0);
		System.out.println(is[1]);
	}
}
```
该句`assert(is.length > 0)`和`if(is.length >0)`意思相近，jvm一般线上都不会开启断言，如果在发布程序的时候，该句会被忽视，因此会导致以下错误，数组越界：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/d13fc3aeec5c43bfa51596f656207306.jpg)
### 04、总结
**断言只是为了用来调试程序，切勿将断言写入业务逻辑中！**

**如果需要测试，更好的工具，可以用`junit`来实现！**

