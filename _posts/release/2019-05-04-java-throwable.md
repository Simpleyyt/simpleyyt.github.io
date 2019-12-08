---
layout: post
title: Java：优雅地处理异常真是一门学问啊！
tagline: by 沉默王二
categories: java基础
tags:
    - 沉默王二
---

再来聊聊继承，以及超类 Object。

<!--more-->

### 01、先有继承，后有多态

利用继承，我们可以基于已存在的类构造一个新类。继承的好处在于，子类可以复用父类的非 `private` 的方法和非 `private` 成员变量。

`is-a` 是继承的一个明显特征，就是说子类的对象引用类型可以是一个父类。我们可以将通用的方法和成员变量放在父类中，达到代码复用的目的；然后将特殊的方法和成员变量放在子类中，除此之外，子类还可以覆盖父类的方法。这样，子类也就焕发出了新的生命力。

一个对象变量可以引用多种类型的现象被称为多态。多态发生的前提条件就是继承。也就是说，先有继承，后有多态。

```java
class Wanger {

	public void write() {
		System.out.println("我为自己活着");
	}
	
}

class Wangxiaoer extends Wanger {
	public void write() {
		System.out.println("我也为自己活着");
	}
}

class Test {
	public static void main(String [] args) {
		Wanger wanger;
		wanger = new Wanger();
		wanger = new Wangxiaoer();

		Wangxiaoer wangxiaoer;
		//wangxiaoer = new Wanger(); // 不可以
		wangxiaoer = new Wangxiaoer(); // 只能这样
	}
}
```

`wanger` 这个对象变量既可以引用 `Wanger` 对象，也可以引用 `Wangxiaoer `对象。但 `wangxiaoer` 就只能引用 `Wangxiaoer` 对象，不能引用 `Wanger` 对象。根本的原因在于 `Wangxiaoer` 是 `Wanger` 的继承者。

当使用 `wanger` 调用 `write()` 方法时，程序会在运行时自动识别其引用的对象类型，然后选择调用哪个方法——这种现象称为动态绑定。

动态绑定有一个非常重要的特性：无需对现有的代码进行修改，就能对程序进行扩展。假如 `Wangdaer` 也继承了 `Wanger`，并且 `wanger` 引用了`Wangdaer` 的对象，那么 `wanger.write()` 仍然可以正常运行。

当然了，有些类不愿意被继承，也没法被继承。谁不愿意被继承呢？比如武则天，亲手弄死自己的亲儿子。谁没法被继承呢，每朝每代最后的那位倒霉皇帝。

类怎么做到不被继承呢？可以使用 `final` 关键字。`final` 关键字修饰的类不能被继承，`final` 修饰的方法不能被覆盖。

```java
final class Wanger {

	public final void write() {
		System.out.println("你们谁都别想继承我");
	}
	
}
```

**继承**是面向对象编程当中举足轻重的一个概念，与多态、封装共为[面向对象](http://www.itwanger.com/java/2019/11/01/oop.html)的三个基本特征。 继承可以使得子类具有父类的成员变量和方法，还可以重新定义、追加成员变量和方法等。

在设计继承的时候，可以将通用的方法和成员变量放在父类中。但不建议随心所欲地将成员变量以 `protected` 的形式放在父类当中；尽管允许这样做，并且子类可以在需要的时候直接访问，但这样做会破坏类的封装性（封装要求成员变量以 `private` 的形式出现，并且提供对应 `getter / setter` 用来访问）。

Java 是不允许多[继承](http://www.itwanger.com/java/2019/11/01/java-extends.html)的，为什么呢？

如果有两个类共同继承一个有特定方法的父类，那么该方法会被两个子类重写。然后，如果你决定同时继承这两个子类，那么在你调用该重写方法时，编译器不能识别你要调用哪个子类的方法。

这也正是著名的菱形问题，见下图。ClassC 同时继承了 ClassA 和 ClassB，ClassC 的对象在调用 ClassA 和 ClassB 中重载的方法时，就不知道该调用 ClassA 的方法，还是 ClassB 的方法。

![](http://www.itwanger.com/assets/images/2019/11/java-extends-object-1.png)

### 02、超类 Object

在 Java 中，所有类都由 Object 类继承而来。Object 这个单词的英文意思是对象，是不是突然感觉顿悟了——万物皆对象？没错，Java 的设计者真是良苦用心了啊！现在，你一定明白了为什么 Java 是面向对象编程语言的原因。

你可能会疑惑地反问道：“我的类明明没有继承 Object 类啊？”如果一个类没用显式地继承某一个类，那么它就会隐式地继承 Object 类。换句话说，不管是鸡生了蛋，还是蛋孵出了鸡，总有一只 Object 鸡或者一个 Object 蛋。

在面试的时候，你可能会被问到这么一个问题：“Object 类包含了哪些方法呢？”

1）`protected Object clone() throws CloneNotSupportedException` 创建并返回此对象的副本。

不过，《阿里巴巴 Java 开发手册》上建议：慎用 Object 的 clone 方法来拷贝对象。因为 Object 的 clone 方法默认是浅拷贝，如果想实现深拷贝需要重写 clone 方法实现属性对象的拷贝。

什么是浅拷贝，什么是深拷贝呢？

浅拷贝是指在拷贝对象时，会对基本数据类型的变量重新复制一份，而对于引用类型的变量只拷贝了引用，并没有对引用指向的对象进行拷贝。

深拷贝是指在拷贝对象时，同时对引用指向的对象进行拷贝。

浅拷贝和深拷贝的区别就在于是否拷贝了对象中的引用变量所指向的对象。

2）`public boolean equals(Object obj)` 判断另一对象与此对象是否「相等」。

该方法使用的区分度最高的“==”操作符进行判断，所以只要两个对象不是同一个对象，那么 `equals()` 方法一定返回 `false`。

《阿里巴巴 Java 开发手册》上强调：由于 Object 的 equals 方法容易抛出空指针异常，所以应该使用常量或者确定不为 null 的对象来调用 equals。

正例：`"test".equals(object);`
反例：`object.equals("test");`

在正式的开发项目当中，最经常使用该方法进行判断的就是字符串。不过，建议使用`org.apache.commons.lang3.StringUtils`，不用担心出现空指针异常。具体使用情况如下所示：

```java
StringUtils.equals(null, null)   = true
StringUtils.equals(null, "abc")  = false
StringUtils.equals("abc", null)  = false
StringUtils.equals("abc", "abc") = true
StringUtils.equals("abc", "ABC") = false
```

3）`public native int hashCode()` 返回此对象的哈希码。`hashCode()` 是一个 `native` 方法，而且返回值类型是整形；实际上，该方法将对象在内存中的地址作为哈希码返回，可以保证不同对象的返回值不同。

>A native method is a Java method whose implementation is provided by non-java code.<br>
>`native` 方法是一个 `Java` 调用非 `Java` 代码的接口。该方法的实现由非 `Java` 语言实现，比如 C。这个特征并非 `Java` 所特有，其它的编程语言也有这个机制，比如 `C++`。

`hashCode()` 通常在哈希表中起作用，比如 `HashMap`。

向哈希表中添加 `Object` 时，首先调用 `hashCode()` 方法计算 `Object` 的哈希码，通过哈希码可以直接定位 `Object` 在哈希表中的位置。如果该位置没有对象，可以直接将 `Object` 插入该位置；如果该位置有对象，则调用 `equals()` 方法比较这个对象与 `Object` 是否相等，如果相等，则不需要保存 `Object`；如果不相等，则将该 `Object` 加入到哈希表中。

4）`protected void finalize() throws Throwable` 当垃圾回收机制确定该对象不再被调用时，垃圾回收器会调用此方法。不过，`fnalize` 机制现在已经不被推荐使用，并且在 JDK 9 开始被标记为 `deprecated`（过时的）。

5）`public final Class getClass()` 返回此对象的运行时类。

当我们想知道一个类本身的一些信息（比如说类名），该怎么办呢？这时候就需要用到 `Class` 类，该类包含了与类有关的信息。请看以下代码：

```java
Wanger wanger = new Wanger();
Class c1 = wanger.getClass();
System.out.println(c1.getName());
// 输出 Wanger
```

6）`public String toString()` 返回此对象的字符串表示形式。

《阿里巴巴 Java 开发手册》强制规定：POJO 类必须重写 `toString` 方法；可以使用 Eclipse 直接生成，点击 「Source」→「Generate toString」。示例如下：

```java
class Wanger {
	private Integer age;

	@Override
	public String toString() {
		return "Wanger [age=" + age + "]";
	}
	
}
```

重写 `toString()` 有什么好处呢？当方法在执行过程中抛出异常时，可以直接调用 POJO 的 `toString()` 方法打印其属性值，便于排查问题。

>POJO（Plain Ordinary Java Object）指简单的 Java 对象，也就是普通的 `JavaBeans`，包含一些成员变量及其 `getter / setter` ，没有业务逻辑。有时叫做 VO (value - object)，有时叫做 DAO （Data Transform Object）。


### 03、总结

本篇，我们先谈了面向对象的重要特征继承；然后谈到了继承的终极父类 `Object`。这些知识点都相当的重要，请务必深入理解！

上一篇：[请用面向对象的思想，谈一谈这次面试的过程](http://www.itwanger.com/java/2019/11/14/java-oo-po.html)

下一篇：[Java：接口和抽象类，傻傻分不清楚？](http://www.itwanger.com/java/2019/11/14/java-interface-abstract.html)

