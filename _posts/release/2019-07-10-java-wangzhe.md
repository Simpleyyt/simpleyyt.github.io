---
layout: post
title:  Java 代码界 3% 的王者？看我是如何解错这 5 道题的
tagline: by 沉默王二
categories: java基础
tags: 
    - 沉默王二
---

前些日子，阿里妹（妹子出题也这么难）发表了一篇文章《悬赏征集！5 道题征集代码界前 3% 的超级王者》——看到这个标题，我内心非常非常激动，因为终于可以证明自己技术很牛逼了。

但遗憾的是，凭借 8 年的 Java 开发经验，我发现这五道题自己全解错了！惨痛的教训再次证明，我是那被秒杀的 97% 的工程师之一。

不过，好歹我这人脸皮特别厚，虽然全都做错了，但还是敢于坦然地面对自己。

<!--more-->

### 01、原始类型的 float

第一题是这样的，代码如下：

```java
public class FloatPrimitiveTest {
    public static void main(String[] args) {
        float a = 1.0f - 0.9f;
        float b = 0.9f - 0.8f;
        if (a == b) {
            System.out.println("true");
        } else {
            System.out.println("false");
        }
    }
}
```

乍一看，这道题也太简单了吧？

`1.0f - 0.9f` 的结果为 0.1f，`0.9f - 0.8f` 的结果为 0.1f，那自然 `a == b` 啊。

但实际的结果竟然不是这样的，太伤自尊了。

```java
float a = 1.0f - 0.9f;
System.out.println(a); // 0.100000024
float b = 0.9f - 0.8f;
System.out.println(b); // 0.099999964
```

加上两条打印语句后，我明白了，原来发生了精度问题。

Java 语言支持两种基本的浮点类型： float 和 double ，以及与它们对应的包装类 Float 和 Double 。它们都依据 IEEE 754 标准，该标准用科学记数法以底数为 2 的小数来表示浮点数。

但浮点运算很少是精确的。虽然一些数字可以精确地表示为二进制小数，比如说 0.5，它等于 2<sup>-1</sup>；但有些数字则不能精确的表示，比如说 0.1。因此，浮点运算可能会导致舍入误差，产生的结果接近但并不等于我们希望的结果。

所以，我们看到了 0.1 的两个相近的浮点值，一个是比 0.1 略微大了一点点的 0.100000024，一个是比 0.1 略微小了一点点的 0.099999964。

Java 对于任意一个浮点字面量，最终都舍入到所能表示的最靠近的那个浮点值，遇到该值离左右两个能表示的浮点值距离相等时，默认采用偶数优先的原则——这就是为什么我们会看到两个都以 4 结尾的浮点值的原因。

### 02、包装器类型 Float

再来看第二题，代码如下：

```java
public class FloatWrapperTest {
    public static void main(String[] args) {
        Float a = Float.valueOf(1.0f - 0.9f);
        Float b = Float.valueOf(0.9f - 0.8f);
        if (a.equals(b)) {
            System.out.println("true");
        } else {
            System.out.println("false");
        }
    }
}
```

乍一看，这道题也不难，对吧？无非是把原始类型的 float 转成了包装器类型 Float，并且使用 `equals` 替代 `==` 进行判断。

这一次，我以为包装器会解决掉精度的问题，所以我猜想输出结果为 `true`。但结果再次打脸——虽然我脸皮厚，但仍然能感觉到脸有些微微的红了起来。

```java
Float a = Float.valueOf(1.0f - 0.9f);
System.out.println(a); // 0.100000024
Float b = Float.valueOf(0.9f - 0.8f);
System.out.println(b); // 0.099999964
```

加上两条打印语句后，我明白了，原来包装器并不会解决精度的问题。

```java
private final float value;
public Float(float value) {
    this.value = value;
}
public static Float valueOf(float f) {
    return new Float(f);
}
public boolean equals(Object obj) {
    return (obj instanceof Float)
           && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
}
```

从源码可以看得出来，包装器 Float 的确没有对精度做任何处理，况且 `equals` 方法的内部仍然使用了 `==` 进行判断。

### 03、switch 判断 null 值的字符串

来看第三题，代码如下：

```java
public class SwitchTest {
    public static void main(String[] args) {
        String param = null;
        switch (param) {
            case "null":
                System.out.println("null");
                break;
            default:
                System.out.println("default");
        }
    }
}
```

这道题就有点令我雾里看花了。 

我们都知道，switch 是一种高效的判断语句，比起 `if/else` 真的是爽快多了。尤其是 JDK 1.7 之后，switch 的 case 条件可以是 char, byte, short, int, Character, Byte, Short, Integer, String, 或者 enum 类型。

本题中，param 类型为 String，那么我认为是可以作为 switch 的 case 条件的，但 param 的值为 null，null 和 "null" 肯定是不匹配的，我认为程序应该进入到 default 语句输出 default。

但结果再次打脸！程序抛出了异常：

```
Exception in thread "main" java.lang.NullPointerException
	at com.cmower.java_demo.Test.main(Test.java:7)
```

也就是说，`switch ()` 的括号中不允许传入 null。为什么呢？

我翻了翻 JDK 的官方文档，看到其中有这样一句描述，我直接搬过来大家看一眼就明白了。

>When the switch statement is executed, first the Expression is evaluated. If the Expression evaluates to null, a NullPointerException is thrown and the entire switch statement completes abruptly for that reason. Otherwise, if the result is of a reference type, it is subject to unboxing conversion.

大致的意思就是说，switch 语句执行的时候，会先执行 `switch ()` 表达式，如果表达式的值为 null，就会抛出 `NullPointerException` 异常。

那到底是为什么呢？

```java
public static void main(String args[])
{
    String param = null;
    String s;
    switch((s = param).hashCode())
    {
    case 3392903: 
        if(s.equals("null"))
        {
            System.out.println("null");
            break;
        }
        // fall through

    default:
        System.out.println("default");
        break;
    }
}
```

借助 jad，我们来反编译一下 switch 的字节码，结果如上所示。原来 `switch ()` 表达式内部执行的竟然是 `(s = param).hashCode()`，当 param 为 null 的时候，s 也为 null，调用 `hashCode()` 方法的时候自然会抛出 `NullPointerException` 了。

### 04、BigDecimal 的赋值方式

来看第四题，代码如下：

```java
public class BigDecimalTest {
    public static void main(String[] args) {
        BigDecimal a = new BigDecimal(0.1);
        System.out.println(a);
        BigDecimal b = new BigDecimal("0.1");
        System.out.println(b);
    }
}
```

这道题真不难，a 和 b 的唯一区别就在于 a 在调用 BigDecimal 构造方法赋值的时候传入了浮点数，而 b 传入了字符串，a 和 b 的结果应该都为 0.1，所以我认为这两种赋值方式是一样的。

但实际上，输出结果完全出乎我的意料：

```java
BigDecimal a = new BigDecimal(0.1);
System.out.println(a); // 0.1000000000000000055511151231257827021181583404541015625
BigDecimal b = new BigDecimal("0.1");
System.out.println(b); // 0.1
```

这究竟又是怎么回事呢？

这就必须看官方文档了，是时候搬出 `BigDecimal(double val)` 的 JavaDoc 镇楼了。

1. The results of this constructor can be somewhat unpredictable. One might assume that writing new BigDecimal(0.1) in Java creates a BigDecimal which is exactly equal to 0.1 (an unscaled value of 1, with a scale of 1), but it is actually equal to 0.1000000000000000055511151231257827021181583404541015625. This is because 0.1 cannot be represented exactly as a double (or, for that matter, as a binary fraction of any finite length). Thus, the value that is being passed in to the constructor is not exactly equal to 0.1, appearances notwithstanding.

解释：使用 double 传参的时候会产生不可预期的结果，比如说 0.1 实际的值是 0.1000000000000000055511151231257827021181583404541015625，说白了，这还是精度的问题。（既然如此，为什么不废弃呢？）

2. The String constructor, on the other hand, is perfectly predictable: writing new BigDecimal("0.1") creates a BigDecimal which is exactly equal to 0.1, as one would expect. Therefore, it is generally recommended that the String constructor be used in preference to this one.

解释：使用字符串传参的时候会产生预期的结果，比如说 `new BigDecimal("0.1")` 的实际结果就是 0.1。

3. When a double must be used as a source for a BigDecimal, note that this constructor provides an exact conversion; it does not give the same result as converting the double to a String using the Double.toString(double) method and then using the BigDecimal(String) constructor. To get that result, use the static valueOf(double) method.

解释：如果必须将一个 double 作为参数传递给 BigDecimal 的话，建议传递该 double 值匹配的字符串值。方式有两种：

```java
double a = 0.1;
System.out.println(new BigDecimal(String.valueOf(a))); // 0.1
System.out.println(BigDecimal.valueOf(a)); // 0.1
```

第一种，使用 `String.valueOf()` 把 double 转为字符串。

第二种，使用 `valueOf()` 方法，该方法内部会调用 `Double.toString()` 将 double 转为字符串，源码如下：

```java
public static BigDecimal valueOf(double val) {
    // Reminder: a zero double returns '0.0', so we cannot fastpath
    // to use the constant ZERO.  This might be important enough to
    // justify a factory approach, a cache, or a few private
    // constants, later.
    return new BigDecimal(Double.toString(val));
}
```

### 05、ReentrantLock

最后一题，也就是第五题，代码如下：

```java
public class LockTest {
    private final static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        try {
            lock.tryLock();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

问题如下：

A: lock 是非公平锁
B: finally 代码块不会抛出异常
C: tryLock 获取锁失败则直接往下执行

很惭愧，我不知道 ReentrantLock 是不是公平锁；也不知道 finally 代码块会不会抛出异常；更不知道 tryLock 获取锁失败的时候会不会直接往下执行。没法作答了。

连续五道题解不出来，虽然我脸皮非常厚，但也觉得脸上火辣辣的，就像被人狠狠地抽了一个耳光。

容我研究研究吧。

1）lock 是非公平锁

ReentrantLock 是一个使用频率非常高的锁，支持重入性，能够对共享资源重复加锁，即当前线程获取该锁后再次获取时不会被阻塞。

ReentrantLock 既是公平锁又是非公平锁。调用无参构造方法时是非公平锁，源码如下：

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
```

所以本题中的 lock 是非公平锁，A 选项是正确的。

ReentrantLock 还提供了另外一种构造方法，源码如下：

```java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

当传入 true 的时候为公平锁，false 的时候为非公平锁。

那公平锁和非公平锁到底有什么区别呢？

公平锁可以保证请求资源在时间上的绝对顺序，而非公平锁有可能导致其他线程永远无法获取到锁，造成“饥饿”的现象。

公平锁为了保证时间上的绝对顺序，需要频繁的上下文切换，而非公平锁会减少一些上下文切换，性能开销相对较小，可以保证系统更大的吞吐量。

2）finally 代码块不会抛出异常

Lock 对象在调用 unlock 方法时，会调用 `AbstractQueuedSynchronizer` 的 `tryRelease` 方法，如果当前线程不持有锁的话，则抛出 `IllegalMonitorStateException` 异常。

所以建议本题的示例代码优化为以下形式（进入业务代码块之前，先判断当前线程是否持有锁）：

```java
boolean isLocked = lock.tryLock();
if (isLocked) {
    try {
        // doSomething();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}
```

3）tryLock 获取锁失败则直接往下执行

`tryLock()` 方法的 Javadoc 如下：

>Acquires the lock if it is available and returns immediately with the value true. If the lock is not available then this method will return immediately with the value false.

中文意思是如果锁可以用，则获取该锁，并立即返回 true，如果锁不可用，则立即返回 false。

针对本题的话， 在 tryLock 获取锁失败的时候，程序会执行 finally 块的代码。

