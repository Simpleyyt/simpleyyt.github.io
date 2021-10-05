---
layout: post
categories: Java
title: Equal 与 “==” 引发的血案
tagline: by 子悠
tags: 
  - 子悠

---

众所周知阿里巴巴开发手册里面有一条强制的规则，说的是在包装类对象之间的值比较的时候需要使用 `equals` 方法，在 `-128` 和 `127` 之间的数值比较可以使用 `==`，如下图所示。具体的原因相信大家都知道，虽然规则中提到 `-128` 和 `127` 之间的数值比较可以使用 ==，但是阿粉强烈建议你还是不要这样，包装类统一使用 `equals`，特别是如果有些数值是通过 `API` 或者 `RPC` 接口过来的，一定要注意。

<!--more-->

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2021/1004/00.png)

我们看看下面的程序

```java
public class IntegerEqualTest {

    public static void main(String[] args) {

        Integer a = genA();
        //Integer a = genB();
        Integer b = 0;
        if (a == b) {
            System.out.println("a == 0");
        } else {
            System.out.println("a != 0");
        }
        System.out.println(a == b);
        System.out.println(a == 0);
    }

    private static Integer genA() {
        return new Integer(0);
    }

    private static Integer genB() {
        return 0;
    }
}
```

大家可以先看下上面这一段代码，先猜测一下运行的结果是什么，如果再把 `Integer a = genA();` 这行注释，`Integer a = genB();` 这行放开，运行的结果又是什么。

好，1 2 3 结果如下所示

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2021/1004/01.png)

当我们替换注释那一行的时候，运行结果如下
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2021/1004/02.png)

看到这里其实很多小伙伴都知道是为什么，**因为 genA() 方法里面是使用的 Integer 的构造器，构造的是一个新的对象，所以在使用 `==` 做对比的时候，比较的两个对象是不一样的。**

是的，原因是这个，但是还有一点没说清楚那就是为什么在使用 `genA()` 的时候，下面的结果会不一样。

```
 System.out.println(a == b);//false
 System.out.println(a == 0);//true
```

其实短短的几行代码里面，包含了好几个知识点，分别是自动装箱拆箱以及 `Integer` 的 `-128` 到 `127` 的数字缓存。

### 装箱拆箱

装箱：自动将基本数据类型转换为包装器类型；

拆箱：就是自动将包装器类型转换为基本数据类型。

在装箱的时候自动调用的是 `Integer` 的 `valueOf(int)` 方法。而在拆箱的时候自动调用的是 `Integer` 的 `intValue`方法。

上面的代码中 `Integer b = 0;` 会触发自动的装箱调用 `Integer valueOf()` 方法。而在使用 `a == 0` 这句的时候，会触发自动的拆箱。然后我们看源码会发现有下面缓存的逻辑，其中 `IntegerCache.low ` 是 `-128`，`IntegerCache.high` 默认是 `127`，不过可以通过 `JVM` 参数进行配置。我们这里的代码是 0，所以会从缓存中获取。

```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

为了充分说明 `Integer` 的缓存，我们看下下面这段程序的执行结果

```
Integer c1 = 128;
Integer c2 = 128;
System.out.println(c1 == c2);
```

在运行之前我们先自己分析一下，首先 `Integer c1 = 128` 和 `Integer c2 = 128` 按照我们上面说的，会触发自动装箱调用 `valueOf` 方法，通过 `valueOf `源码我们可以看到在默认的情况下 `128` 已经不再 `Integer` 的缓存里面了，所以 `if` 条件不满足会通过 `new Integer` 构造方法创建两个对象，所以最终的结果应该是输出 `false`。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2021/1004/03.png)

下面再说一下为什么说在 `-128` 和 `127` 以内的也不建议直接使用 `==` 来实现比较，很显然就跟我们上面的`genA()` 方法一样，很多时候不会一下子就知道一个方法值是怎么得到，即使是缓存范围以内，别人也有可能是通过构造函数创建出来的，这样我们在做比较的时候很有可能就会跟预期的不一样，从而产生事故。特别是如果通过 RPC 接口获得返回结果，我们可能连别人的实现方式压根就看不到，更没办法提前知道了。所以我们还是老老实实的按照阿里巴巴的 Java 规范来编写代码，采用`equals` 方法来判断，这样肯定没问题。

> 需要阿里开发手册的同学，在公众号回复【规范】即可获取下载链接。
