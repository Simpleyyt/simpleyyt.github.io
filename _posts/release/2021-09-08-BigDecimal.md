---
layout: post
categories: Java
title: 一文带你详解 BigDecimal Java 小能手必备
tagline: by 子悠
tags: 
  - 子悠

---

作为 `Java` 程序员在日常的工作中，很多时候我们都会遇到一些需要进行数据计算的场景，通常对于不需要计算精度的场景我们都可以使用 `Integer`，`Float` 或者 `Double` 来进行计算，虽然会丢失精度但是偶尔也可以用，如果我们需要精确计算结果的时候，就会用到 `java.math` 包中提供的 `BigDecimal` 类来实现对应的功能了。

<!--more-->

`BigDecimal` 作为精确数据计算的工具，既然是数据计算，那肯定会提供相应的加减乘除的方法来让我们使用，如下：

1. `add(BigDecimal)`：`BigDecimal` 对象中的值相加，返回 `BigDecimal` 对象
2. `subtract(BigDecimal)`：`BigDecimal` 对象中的值相减，返回 `BigDecimal` 对象
3. `multiply(BigDecimal)`：`BigDecimal` 对象中的值相乘，返回 `BigDecimal` 对象
4. `divide(BigDecimal)`：`BigDecimal` 对象中的值相除，返回 `BigDecimal` 对象

需要使用对应的方法的时候，我们首先要创建 `BigDecimal` 对象，然后才能使用，对应的构造方法有 

1. `BigDecimal(int)`：创建一个具有参数所指定整数值的对象
2. `BigDecimal(double)`：创建一个具有参数所指定双精度值的对象
3. `BigDecimal(long)`：创建一个具有参数所指定长整数值的对象

4. `BigDecimal(String)`：创建一个具有参数所指定以字符串表示的数值的对象

通过构造方法创建出的 `BigDecimal` 对象后，通过调用对应的方法以及传入另一个 `BigDecimal` 参数来实现相应的加减乘除方法。如下示例：

```java
package org.test;

import java.math.BigDecimal;

public class TestClass {

    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("11");
        BigDecimal num2 = new BigDecimal("102");
        BigDecimal result1 = num2.add(num1);
        BigDecimal result2 = num2.subtract(num1);
        BigDecimal result3 = num2.multiply(num1);
        System.out.println("num2 + num1 = " + result1);
        System.out.println("num2 - num1 = " + result2);
        System.out.println("num2 * num1 = " + result3);
        System.out.println("num2 / num1 = " + (102 / 11));
        System.out.println("ROUND_UP: num2 / num1 = " + num2.divide(num1, 2, BigDecimal.ROUND_UP));
        System.out.println("ROUND_DOWN: num2 / num1 = " + num2.divide(num1, 2, BigDecimal.ROUND_DOWN));
        System.out.println("ROUND_CEILING: num2 / num1 = " + num2.divide(num1, 2, BigDecimal.ROUND_CEILING));
        System.out.println("ROUND_FLOOR: num2 / num1 = " + num2.divide(num1, 2, BigDecimal.ROUND_FLOOR));
        System.out.println("ROUND_HALF_UP: num2 / num1 = " + num2.divide(num1, 2, BigDecimal.ROUND_HALF_UP));
        System.out.println("ROUND_HALF_DOWN: num2 / num1 = " + num2.divide(num1, 2, BigDecimal.ROUND_HALF_DOWN));
        System.out.println("ROUND_HALF_EVEN: num2 / num1 = " + num2.divide(num1, 2, BigDecimal.ROUND_HALF_EVEN));
    }
}

```

运行的结果如下图所示
![](/Users/silence/IdeaProjects/justdojava.github.io/assets/images/2019/java/image_ziyou/2021/0920/01.png)

从上图中我们看到 BigDecimal 具体使用方式，通过调用对应的方法再传入对应的参数即可。不过再进行除法运算的时候我们可以看到，divide 方法还提供了设置精确位数的参数，并且还可以设置具体的取整方式。取整方式有如下几种：

```java
//绝对值向上取整，远离坐标抽 0 取整
public final static int ROUND_UP =           0;
//绝对值向下取整，向着坐标城 0 取整
public final static int ROUND_DOWN =         1;
//数值方向向上取整，向正无穷方向取整
public final static int ROUND_CEILING =      2;
//数值方向向下取整，向负无穷方向取整
public final static int ROUND_FLOOR =        3;
// >= 0.5 绝对值方向向上取整， < 0.5 绝对值方向向下取整
public final static int ROUND_HALF_UP =      4;
// <= 0.5 绝对值方向向下取整， > 0.5 绝对值方向向上取整
public final static int ROUND_HALF_DOWN =    5;
//舍入模式向“最近邻居”舍入，除非两个邻居等距，在这种情况下，向偶数邻居舍入。如果丢弃的分数左边的数字是奇数，则//行为与 RoundingMode.HALF_UP 相同；如果它是偶数，则表现为 RoundingMode.HALF_DOWN
public final static int ROUND_HALF_EVEN =    6;
public final static int ROUND_UNNECESSARY =  7;
```

除了加减乘除已经方法之外 BigDecimal 也提供两个 BigDecimal 进行对比的方法 compareTo()，用法如下

```java
BigDecimal num1 = new BigDecimal("101");
BigDecimal num2 = new BigDecimal("102");
int i = num2.compareTo(num1);
System.out.println(i);
// 运行结果：1
```

因为 num2 比 num1 数值大，所以返回值为 1；当 num2 与 num1 相等是返回 0；当 num2 小于 num1 时返回-1。

总结格式如下

```java
int a = bigdemical.compareTo(bigdemical2);
//a = -1,表示bigdemical小于bigdemical2；
//a = 0,表示bigdemical等于bigdemical2；
//a = 1,表示bigdemical大于bigdemical2；
```

所以经常会有`if (num2.compareTo(num1) > 0) ` 来进行判断操作。

同时 BigDecimal 也提供直接转换为 int，long，float，double 数值的方法，如下所示，一般使用的情况相对较少。

```java
int i1 = num1.intValue();
long l = num1.longValue();
float v = num1.floatValue();
double v1 = num1.doubleValue();
short i2 = num1.shortValue();
byte b = num1.byteValue();
```

在日常工作中需要精确的小数计算时使用 `BigDecimal`，`BigDecimal` 的性能比 `double` 和 `float` 相对较差，所以只有在需要的时候使用就好。`BigDecimal` 在每次进行加减乘除的时候都会创建一个新的对象，当后面需要使用的时候我们需要保存起来，通常情况我们尽量使用 String 类型的构造函数。