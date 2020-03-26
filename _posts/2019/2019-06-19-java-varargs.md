---
layout: post
title: java的可变参数
tagline: by 炸鸡可乐
categories: Java
tags: 
  - 炸鸡可乐
---

Java方法中的可变参数类型是一个非常重要的概念，有着非常广泛的应用，今天小编带大家一起去深入的了解java的可变参数使用方式！

<!--more-->

### 01、什么是可变参数
> 在Java5 中提供了变长参数（varargs），也就是在方法定义中可以使用个数不确定的参数。

使用`...`表示可变长参数，例如
```
private void print(String... args){
   ...
}
```
在具有可变长参数的方法中可以把参数当成数组使用，例如可以循环输出所有的参数值。
```
private void print(String... args) {
		for (String string : args) {
			System.out.println("可变参数：" + string);
		}
	}
```
### 02、可变参数的调用方式
调用的时候可以给出任意多个参数也可不给参数，例如：
```
print();
print("test1");
print("var-test1", "var-test2");
```
### 03、可变参数的使用规则
#### 3.1、优先匹配固定参数
> 在调用方法的时候，如果这个方法能够和固定参数的方法匹配，也能够与可变长参数的方法匹配，那么优先选择固定参数的方法。

看下面代码的输出：
```
public class VarArgsTest {

	private void print(String test) {
		System.out.println("固定参数");
	}

	private void print(String... args) {
		System.out.println("可变参数");
	}
	
	public static void main(String[] args) {
		VarArgsTest test = new VarArgsTest();
		test.print("test1");
		test.print("var-test1", "var-test2");
	}
}
```
输出结果：
```
固定参数
可变参数
```
#### 3.2、如果要调用的方法可以和两个可变参数匹配，则出现错误。
例如下面的代码：
```
public class VarArgsTest {

	private void print(String... args) {
		for (String string : args) {
			System.out.println("可变参数：" + string);
		}
	}

	private void print(String test, String... args) {
		for (String string : args) {
			System.out.println(test + ",新的可变参数：" + string);
		}
	}

	public static void main(String[] args) {
		VarArgsTest test = new VarArgsTest();
		test.print("var-test1", "var-test2");
	}
}
```
编译器报错！
![](http://www.justdojava.com/assets/images/2019/java/image-jay/2022999f08ed402183081e125b74ceb1.jpg)

main方法中的两个调用都不能编译通过，因为编译器不知道该选哪个方法调用！
#### 3.3、一个方法只能有一个可变长参数，并且这个可变长参数必须是该方法的最后一个参数
```
private void print(String test, String... args) {
	for (String string : args) {
		System.out.println(test + ",新的可变参数：" + string);
	}
}

private void test(String... args, String test) {
	for (String string : args) {
		System.out.println(test + ",新的可变参数：" + string);
	}
}
```
编译器报错！
![](http://www.justdojava.com/assets/images/2019/java/image-jay/bff366ad5d674348aa9daf71b6ec9da8.jpg)
### 04、可变长参数的使用规范
#### 4.1、避免带有可变长参数的方法重载
如3.2中，编译器虽然知道怎么调用，但人容易陷入调用的陷阱及误区

![](http://www.justdojava.com/assets/images/2019/java/image-jay/2022999f08ed402183081e125b74ceb1.jpg)
#### 4.2、别让null值和空值威胁到变长方法
请看下面的例子：
```
public class VarArgsTest {

	private void print(String test, String... args) {
		for (String string : args) {
			System.out.println(test + ",新的可变字符串参数：" + string);
		}
	}

	private void print(String test, Integer... args) {
        for (Integer integer : args) {
        	System.out.println(test + ",新的可变整型参数：" + integer);
		}
    }

	public static void main(String[] args) {
		VarArgsTest test = new VarArgsTest();
		test.print("hello");
        test.print("hello", null);
	}
}
```
编译器报错！
![](http://www.justdojava.com/assets/images/2019/java/image-jay/2941e4104499401a98b276f39ef76cf1.jpg)

因为两个方法都匹配，编译器不知道选哪个，于是报错了，这里同时还有个非常不好的编码习惯，即调用者隐藏了实参类型，这是非常危险的，不仅仅调用者需要“猜测”该调用哪个方法，而且被调用者也可能产生内部逻辑混乱的情况。对于本例来说应该做如下修改：
```
public static void main(String[] args) {
		VarArgsTest test = new VarArgsTest();
		String[] strs = null;
        test.print("hello", strs);
}
```
#### 4.3、覆写变长方法也要循规蹈矩
看下面一个例子，创建三个类，大家猜测下程序能不能编译通过：
```
/**
 * 父类
 */
public class Base {
	
	void print(String... args) {
        System.out.println("Base......test");
    }
}

/**
 * 子类
 */
public class Sub extends Base{

	void print(String[] args) {
		System.out.println("Sub......test");
	}
}

/**
 * 测试类
 */
public class VarArgsTestDemo {
	
	public static void main(String[] args) {
		// 向上转型
        Base base = new Sub();
        base.print("hello");
        
        // 不转型
        Sub sub = new Sub();
        sub.print("hello");
	}
}
```
编译器报错
![](http://www.justdojava.com/assets/images/2019/java/image-jay/d1116356411647f59dcb02e4854b7ff7.jpg)

第一个能编译通过，这是为什么呢？

事实上，base对象把子类对象sub做了向上转型，形参列表是由父类决定的，当然能通过。而看看子类直接调用的情况，这时编译器看到子类覆写了父类的print方法，因此肯定使用子类重新定义的print方法，尽管参数列表不匹配也不会跑到父类再去匹配下，因为找到了就不再找了，因此有了类型不匹配的错误。

这是个特例，覆写的方法参数列表竟然可以与父类不相同，这违背了覆写的定义，并且会引发莫名其妙的错误。

总结下覆写必须满足的条件：

* 重写方法不能缩小访问权限；
* 参数列表必须与被重写方法相同（包括显示形式）；
* 返回类型必须与被重写方法的相同或是其子类；
* 重写方法不能抛出新的异常，或者超过了父类范围的异常，但是可以抛出更少、更有限的异常，或者不抛出异常。

最后，给出一个有陷阱的例子，大家应该知道输出结果：
```
public class VarArgsTest1 {
	
	private static void test(String s, String... ss) {
        for (int i = 0; i < ss.length; i++) {
            System.out.println(ss[i]);
        }
    }
	
	public static void main(String[] args) {
		test(null);
		test("");
		test("aaa");
		test("aaa","bbb");
	}
}
```
输出结果：
```
bbb
```
