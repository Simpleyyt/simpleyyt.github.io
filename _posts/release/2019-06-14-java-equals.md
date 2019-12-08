---
layout: post
title: Java的== 与 equals区别
tagline: by Jay pan
categories: java基础
tags: 
  - Jaypan
---


碰到“equals”和“==”这两个字符，老感觉差不多；其实还是有一些区别的，今天小编带大家一探究竟！

<!--more-->

### 01、==介绍
> 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。

* **基本数据类型**：byte,short,char,int,long,float,double,boolean。他们之间的比较，应用双等号（==）,比较的是他们的值。

* **引用数据类型**：当他们用（==）进行比较的时候，比较的是他们在内存中的存放地址（确切的说，是堆内存地址）。

举个例子
```
public class IntegerSame {

	@Test
	public void test() {
		int i = 100;//基本数据类型
		int ii = 100;//基本数据类型
		Integer j = 100;//引用类型
		Integer jj = 100;//引用类型
		Integer k = new Integer(100);//引用类型
		Integer kk = new Integer(100);//引用类型
		System.out.println("i的地址:" + System.identityHashCode(i));
		System.out.println("ii的地址:" + System.identityHashCode(ii));
		System.out.println("j的地址:" + System.identityHashCode(j));
		System.out.println("jj的地址:" + System.identityHashCode(jj));
		System.out.println("k的地址:" + System.identityHashCode(k));
		System.out.println("kk的地址:" + System.identityHashCode(kk));
		
		//基本类型相互比较其中的值，所以得出true
		System.out.println("i == ii 结果：" + (i == ii));
		//当int的引用类型Integer与基本类型进行比较的时候，包装类会先进行自动拆箱
		//然后与基本类型进行值比较，所有得出true
		System.out.println("i == j 结果：" + (i == j));
		//同上，包装类先拆箱成基本类型，然后比较，得出true
		System.out.println("i == k 结果：" + (i == k));
		
		//都是引用类型，所有比较的是地址，因为j与jj的地址相同，所有true
		System.out.println("j == jj 结果：" + (j == jj));
		//都是引用类型，所有比较的是地址，因为k与kk的地址相同，所有true
		System.out.println("k == kk 结果：" + (k == kk));
	}

}
```
输入结果：
![](/assets/images/2019/java/image-jay/9af6ea7c2a164f49bf82eec3db587b9a.jpg)
**疑问点：为什么j和jj的地址是一样的，k与kk的地址却不一样呢？**
**答案：在-128~127的Integer值并且以Integer x = value;的方式赋值的参数，x会从包装类型自动拆箱成基本数据类型，以供重用！所以，j、jj的内存地址都是一样的！**
下面我们把100变成1000试试！
```
public class IntegerSame {

	@Test
	public void test() {
		int i = 10000;//基本数据类型
		int ii = 10000;//基本数据类型
		Integer j = 10000;//引用类型
		Integer jj = 10000;//引用类型
		Integer k = new Integer(10000);//引用类型
		Integer kk = new Integer(10000);//引用类型
		System.out.println("i的地址:" + System.identityHashCode(i));
		System.out.println("ii的地址:" + System.identityHashCode(ii));
		System.out.println("j的地址:" + System.identityHashCode(j));
		System.out.println("jj的地址:" + System.identityHashCode(jj));
		System.out.println("k的地址:" + System.identityHashCode(k));
		System.out.println("kk的地址:" + System.identityHashCode(kk));
		
		//基本类型相互比较其中的值，所以得出true
		System.out.println("i == ii 结果：" + (i == ii));
		//当int的引用类型Integer与基本类型进行比较的时候，包装类会先进行自动拆箱
		//然后与基本类型进行值比较，所有得出true
		System.out.println("i == j 结果：" + (i == j));
		//同上，包装类先拆箱成基本类型，然后比较，得出true
		System.out.println("i == k 结果：" + (i == k));
		
		//都是引用类型，所有比较的是地址，因为j与jj的地址相同，所有true
		System.out.println("j == jj 结果：" + (j == jj));
		//都是引用类型，所有比较的是地址，因为k与kk的地址相同，所有true
		System.out.println("k == kk 结果：" + (k == kk));
	}

}
```
输入结果：
![](/assets/images/2019/java/image-jay/801dad9c1efe4b54835afc8e0fa92985.jpg)
当j、jj超出-128~127区间的时候，地址就变了，所以比较的结果就是false。
再看其它的包装器自动拆箱情况：

|类型|描述|
|:--:|:--:|
|Boolean |全部自动拆箱|
|Byte  |全部自动拆箱|
|Short|-128~127区间自动拆箱|
|Integer|-128~127区间自动拆箱|
|Long |-128~127区间自动拆箱|
|Float  |没有拆箱|
|Doulbe  |没有拆箱|
|Character  |0~127区间自动拆箱|
### 02、equals()方法介绍
> 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

* 情况1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
* 情况2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

Boolean、Byte、Short、Integer、Long、Float、Doulbe、Character 8种基本类型的包装类都重写了 equals() 方法，所以比较的时候，如果内容相同，则返回 true，例如：
```
//因为内容相同，返回的都是true
System.out.println("j.equals(jj) 结果：" + (j.equals(jj)));
System.out.println("(k.equals(kk) 结果：" + (k.equals(kk)));
```
![](/assets/images/2019/java/image-jay/c273d4d1e12849d1806bed1bf6817ccc.jpg)

### 03、String类型的比较介绍
> string是一个非常特殊的数据类型，它可以通过String x = value;的方式进行赋值，也可以通过String x = new String(value)方式进行赋值。

String x = value;方式赋予的参数，会放入常量池内存块区域中；
String x = new String(value)方式赋予的参数，会放入堆内存区域中，当成对象处理。
举个例子：
```
public class DemoEquals {

	public static void main(String[] args) {
		String a = new String("ab"); // a 为一个引用
		String b = new String("ab"); // b为另一个引用,对象的内容一样
		String aa = "ab"; // 放在常量池中
		String bb = "ab"; // 从常量池中查找
		System.out.println("a地址：" + System.identityHashCode(a));
		System.out.println("b地址：" + System.identityHashCode(b));
		System.out.println("aa地址：" + System.identityHashCode(aa));
		System.out.println("bb地址：" + System.identityHashCode(bb));
		//地址相同，所以返回true
		if (aa == bb) {
			System.out.println("aa==bb");
		}
		// 地址不同，非同一个对象，所以返回false
		if (a == b) {
			System.out.println("a==b");
		}
		//地址不同，但是内容相同，所以返回true
		if (a.equals(b)) {
			System.out.println("aEQb");
		}
	}
}
```
输入结果：
![](/assets/images/2019/java/image-jay/0afd2698effc47a3a7994c349b673aed.jpg)
为什么string的equals()方法比较返回true，因为string重写了equals()方法，源码如下：
```
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```
如果内容相同，则返回true!

**总结：如果需要比较某个对象是否相同，一定要重写equals()，比较其中的内容是否相同，如果相同，返回true；否则，返回false!**