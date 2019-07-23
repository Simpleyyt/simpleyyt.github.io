---
layout: post  
title: 浅谈Java中字符串的初始化及字符串操作类。    
tagline: by 炭烧生蚝  
categories: Java  
tag: 
    - Java
---

在深入学习字符串类之前, 我们先搞懂JVM是怎样处理新生字符串的. 
 当你知道字符串的初始化细节后, 再去写`String s = "hello"`或`String s = new String("hello")`等代码时, 就能做到心中有数。

<!--more-->


首先得搞懂字符串常量池的概念。

常量池是Java的一项技术, 八种基础数据类型除了float和double都实现了常量池技术. 这项技术从字面上是很好理解的: 把经常用到的数据存放在某块内存中, 避免频繁的数据创建与销毁, 实现数据共享, 提高系统性能。

字符串常量池是Java常量池技术的一种实现, 在近代的JDK版本中(1.7后), 字符串常量池被实现在Java堆内存中。

下面通过三行代码让大家对字符串常量池建立初步认识:

```java
public static void main(String[] args) {
    String s1 = "hello";
    String s2 = new String("hello");
    System.out.println(s1 == s2);   //false
}
```

我们先来看看第一行代码`String s1 = "hello";`干了什么.

![字符串常量池内存图](https://user-gold-cdn.xitu.io/2019/5/25/16aede2ab2ddeeca?w=765&h=389&f=png&s=26653)

对于这种直接通过双引号""声明字符串的方式, 虚拟机首先会到字符串常量池中查找该字符串是否已经存在. 如果存在会直接返回该引用, 如果不存在则会在堆内存中创建该字符串对象, 然后到字符串常量池中注册该字符串。

在本案例中虚拟机首先会到字符串常量池中查找是否有存在"hello"字符串对应的引用. 发现没有后会在堆内存创建"hello"字符串对象(内存地址0x0001), 然后到字符串常量池中注册地址为0x0001的"hello"对象, 也就是添加指向0x0001的引用. 最后把字符串对象返回给s1。

温馨提示: 图中的字符串常量池中的数据是虚构的, 由于字符串常量池底层是用HashTable实现的, 存储的是键值对,  为了方便大家理解, 示意图简化了字符串常量池对照表, 并采用了一些虚拟的数值。

&nbsp;
下面看`String s2 = new String("hello");`的示意图：


![字符串常量池内存图](https://user-gold-cdn.xitu.io/2019/5/25/16aede56fde9c26d?w=792&h=398&f=png&s=32253)

当我们使用new关键字创建字符串对象的时候, JVM将不会查询字符串常量池, 它将会直接在堆内存中创建一个字符串对象, 并返回给所属变量。

所以s1和s2指向的是两个完全不同的对象, 判断s1 == s2的时候会返回false。

&nbsp;

> 如果上面的知识理解起来没有问题的话, 下面看些难点的. 

```java
public static void main(String[] args) {
    String s1 = new String("hello ") + new String("world");
    s1.intern();
    String s2 = "hello world";
    System.out.println(s1 == s2);   //true
}
```

第一行代码`String s1 = new String("hello ") + new String("world");`的执行过程是这样子的: 

1.依次在堆内存中创建"hello "和"world"两个字符串对象

2.然后把它们拼接起来  (底层使用StringBuilder实现, 后面会带大家读反编译代码)

3.在拼接完成后会产生新的"hello world"对象, 这时变量s1指向新对象"hello world"

执行完第一行代码后, 内存是这样子的: 

![字符串常量池内存图](https://user-gold-cdn.xitu.io/2019/5/25/16aedb1abe075763?w=761&h=393&f=png&s=24041)


&nbsp;
第二行代码`s1.intern();`

String类的源码中有对`intern()`方法的详细介绍, 翻译过来的意思是: 当调用`intern()`方法时, 首先会去常量池中查找是否有该字符串对应的引用, 如果有就直接返回该字符串; 如果没有, 就会在常量池中注册该字符串的引用, 然后返回该字符串。

由于第一行代码采用的是new的方式创建字符串, 所以在字符串常量池中没有保存"hello world"对应的引用, 虚拟机会在常量池中进行注册, 注册完后的内存示意图如下: 


![字符串常量池内存图](https://user-gold-cdn.xitu.io/2019/5/25/16aedcc45a1f576f?w=774&h=397&f=png&s=25692)

第三行代码`String s2 = "hello world";`

这种直接通过双引号""声明字符串背后的运行机制我们在第一个案例提到过, 这里正好复习一下。

首先虚拟机会去检查字符串常量池, 发现有指向"hello world"的引用. 然后把该引用所指向的字符串直接返回给所属变量。

执行完第三行代码后, 内存示意图如下: 


![](https://user-gold-cdn.xitu.io/2019/5/25/16aedd066d47343e?w=765&h=393&f=png&s=40313)

如图所示, s1和s2指向的是相同的对象, 所以当判断s1 == s2时返回true。

&nbsp;
**最后我们对字符串常量池进行总结**:

当用new关键字创建字符串对象时, 不会查询字符串常量池; 当用双引号直接声明字符串对象时, 虚拟机将会查询字符串常量池. 说白了就是: 字符串常量池提供了字符串的复用功能, 除非我们要显式创建新的字符串对象, 否则对同一个字符串虚拟机只会维护一份拷贝。

&nbsp;
## 配合反编译代码验证字符串初始化操作.

相信看到这里, 再见到有关的面试题, 你已经无所畏惧了, 因为你已经懂得了背后原理。

在结束之前我们不妨再做一道压轴题

```
public class Main {
    public static void main(String[] args) {
        String s1 = "hello ";
        String s2 = "world";
        String s3 = s1 + s2;
        String s4 = "hello world";
        System.out.println(s3 == s4);
    }
}
```
> 这道压轴题是经过精心设计的, 它不但照应上面所讲的字符串常量池知识, 也引出了后面的话题. 

如果看这篇文章是你第一次往底层探索字符串的经历, 那我估计你不能立即给出答案. 因为我第一次见这几行代码时也卡壳了。

首先第一行和第二行是常规的字符串对象声明, 我们已经很熟悉了, 它们分别会在堆内存创建字符串对象, 并会在字符串常量池中进行注册。

影响我们做出判断的是第三行代码`String s3 = s1 + s2;`, 我们不知道`s1 + s2`在创建完新字符串"hello world"后是否会在字符串常量池进行注册。

说白了就是我们不知道这行代码是以双引号""形式声明字符串, 还是用new关键字创建字符串。

这时, 我们应该去读一读这段代码的反编译代码. 如果你没有读过反编译代码, 不妨借此机会入门。

在命令行中输入`javap -c 对应.class文件的绝对路径`, 按回车后即可看到反编译文件的代码段。

```
C:\Users\liuyj>javap -c C:\Users\liuyj\IdeaProjects\Test\target\classes\forTest\Main.class
Compiled from "Main.java"
public class forTest.Main {
  public forTest.Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String hello
       2: astore_1
       3: ldc           #3                  // String world
       5: astore_2
       6: new           #4                  // class java/lang/StringBuilder
       9: dup
      10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
      13: aload_1
      14: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      17: aload_2
      18: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      21: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      24: astore_3
      25: ldc           #8                  // String hello world
      27: astore        4
      29: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
      32: aload_3
      33: aload         4
      35: if_acmpne     42
      38: iconst_1
      39: goto          43
      42: iconst_0
      43: invokevirtual #10                 // Method java/io/PrintStream.println:(Z)V
      46: return
}
```
- 首先调用构造器完成Main类的初始化
- `       0: ldc           #2                  // String hello`
- 从常量池中获取"hello "字符串并推送至栈顶, 此时拿到了"hello "的引用
- `    2: astore_1`
- 将栈顶的字符串引用存入第二个本地变量s1, 也就是s1已经指向了"hello "
- `3: ldc           #3                  // String world`
- ` 5: astore_2`
- 重复开始的步骤, 此时变量s2指向"word"
- ` 6: new           #4                  // class java/lang/StringBuilder`
- 刺激的东西来了: 这时创建了一个StringBuilder, 并把其引用值压到栈顶
- `    9: dup`
- 复制栈顶的值, 并继续压入栈定, 也就意味着栈从上到下有两份StringBuilder的引用, 将来要操作两次StringBuilder. 
- `10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V`
- 调用StringBuilder的一些初始化方法, 静态方法或父类方法, 完成初始化. 
- 13: aload_1
- 把第二个本地变量也就是s1压入栈顶, 现在栈顶从上往下数两个数据依次是:s1变量和StringBuilder的引用
- `14: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;`
- 调用StringBuilder的append方法, 栈顶的两个数据在这里调用方法时就用上了. 
- 接下来又调用了一次append方法(之前StringBuilder的引用拷贝两份就用途在此)
- 完成后, StringBuilder中已经拼接好了"hello world", 看到这里相信大家已经明白虚拟机是如何拼接字符串的了. 接下来就是**关键环节**

&nbsp;
- `21: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;`
- `    24: astore_3`
- 拼接完字符串后, 虚拟机调用StringBuilder的`toString()`方法获得字符串`hello world`, 并存放至s3. 
- 激动人心的时刻来了, 我们之所以不知道这道题的答案是因为不知道字符串拼接后是以new的形式还是以双引号""的形式创建字符串对象. 
- 下面是我们追踪StringBuilder的`toString()`方法源码:
```java
    @Override
    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }
```
ok, 这道题解了, s3是通过new关键字获得字符串对象的。

回到题目, 也就是说字符串常量表中没有存储"hello world"的引用, 当s4以引号的形式声明字符串时, 由于在字符串常量池中查不到相应的引用, 所以会在堆内存中新创建一个字符串对象. 所以s3和s4指向的不是同一个字符串对象, 结果为false。

# 详解字符串操作类

明白了字符串常量池, 我相信关于字符串的创建你已经有十足的把握了. 但是这还不够, 作为一名合格的Java工程师, 我们还必须对字符串的操作做到了如指掌. 注意! 不是说你不用查api能熟练操作字符串就了如指掌了, 而是说对String, StringBuilder, StringBuffer三大字符串操作类背后的实现了然于胸, 这样才能在开发的过程中做出正确, 高效的选择。

## String, StringBuilder, StringBuffer的底层实现

点进String的源码, 我们可以看见String类是通过char类型数组实现的。
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
    ...
}    
```

接着查看StringBuilder和StringBuffer的源码, 我们发现这两者都继承自AbstractStringBuilder类, 通过查看该类的源码, 得知StringBuilder和StringBuffer两个类也是通过char类型数组实现的。

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;
    ...
}
```

而且通过StringBuilder和StringBuffer继承自同一个父类这点, 我们可以推断出它俩的方法都是差不多的. 通过查看源码也发现确实如此, 只不过StringBuffer在方法上添加了`synchronized`关键字, 证明它的方法绝大多数方法都是线程同步方法. 也就是说在多线程的环境下我们应该使用StringBuffer以保证线程安全, 在单线程环境下我们应使用StringBuilder以获得更高的效率。

既然如此, 我们的比较也就落到了StringBuilder和String身上了。

## 关于StringBuilder和String之间的讨论
通过查看StringBuilder和String的源码我们会发现两者之间一个关键的区别: 对于String, 凡是涉及到返回参数类型为String类型的方法, 在返回的时候都会通过new关键字创建一个新的字符串对象; 而对于StringBuilder, 大多数方法都会返回StringBuilder对象自身。

```java
/**
 * 下面截取几个String类的方法
 */
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}

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

/**
 * 下面截取几个StringBuilder类的方法
 */
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}

@Override
public StringBuilder replace(int start, int end, String str) {
    super.replace(start, end, str);
    return this;
}
```


就因为这点区别, 使得两者在操作字符串时在不同的场景下会体现出不同的效率。

下面还是以拼接字符串为例比较一下两者的性能：

```java
public class Main {
    public static int time = 50000;

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        String s = "";
        for(int i = 0; i < time; i++){
            s += "test";
        }
        long end = System.currentTimeMillis();
        System.out.println("String类使用时间: " + (end - start) + "毫秒");

    }
}
//String类使用时间: 4781毫秒
```
```java
public class Main {
    public static int time = 50000;

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < time; i++){
            sb.append("test");
        }
        long end = System.currentTimeMillis();
        System.out.println("StringBuilder类使用时间: " + (end - start) + "毫秒");

    }
}
//StringBuilder类使用时间: 5毫秒
```

就拼接5万次字符串而言, StringBuilder的效率是String类的956倍。

我们再次通过反编译代码看看造成两者性能差距的原因, 先看String类. (为了方便阅读代码, 我删除了计时部分的代码, 并重新编译, 得到的main方法反编译代码如下)

```java
public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String, 将""空字符串加载到栈顶
       2: astore_1                          //存放到s变量中
       3: iconst_0                          //把int型数0压栈
       4: istore_2                          //存到变量i中
       5: iload_2                           //把i的值压到栈顶(0)
       6: getstatic     #3                  // Field time:I 拿到静态变量time的值, 压到栈顶
       9: if_icmpge     38                  // 比较栈顶两个int值, for循环中的判定, 如果i比time小就继续执行, 否则跳转
       
//从这里开始, 就是for循环部分
      12: new           #4                  // class java/lang/StringBuilder
      15: dup
      16: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
      19: aload_1
      20: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      23: ldc           #7                  // String test
      25: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      28: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      31: astore_1                          //每拼接完一次, 就把新的字符串对象引用保存在第二个本地变量中
//到这里一次for循环结束
      32: iinc          2, 1                //变量i加1
      35: goto          5                   //继续循环
      38: return
```

从反汇编代码中可以看到, 当用String类拼接字符串时, 每次都会生成一个StringBuilder对象, 然后调用两次append()方法把字符串拼接好, 最后通过StringBuilder的toString()方法new出一个新的字符串对象。

也就是说每次拼接都会new出两个对象, 并进行两次方法调用, 如果拼接的次数过多, 创建对象所带来的时延会降低系统效率, 同时会造成巨大的内存浪费. 而且当内存不够用时, 虚拟机会进行垃圾回收, 这也是一项相当耗时的操作, 会大大降低系统性能。


下面是使用StringBuilder拼接字符串得到的反编译代码：

```java
  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/StringBuilder
       3: dup
       4: invokespecial #3                  // Method java/lang/StringBuilder."<init>":()V
       7: astore_1
       8: iconst_0
       9: istore_2
      10: iload_2
      11: getstatic     #4                  // Field time:I
      14: if_icmpge     30

//从这里开始执行for循环内的代码
      17: aload_1
      18: ldc           #5                  // String test
      20: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      23: pop
//到这里一次for循环结束      
      24: iinc          2, 1
      27: goto          10
      30: return
```

可以看到StringBuilder拼接字符串就简单多了, 直接把要拼接的字符串放到栈顶进行append就完事了, 除了开始时创建了StringBuilder对象, 运行时期没有创建过其他任何对象, 每次循环只调用一次append方法. 所以从效率上看, 拼接大量字符串时, StringBuilder要比String类给力得多。


当然String类也不是没有优势的, 从操作字符串api的丰富度上来讲, String是要多于StringBuilder的, 在日常操作中很多业务都需要用到String类的api。

在拼接字符串时, 如果是简单的拼接, 比如说`String s = "hello " + "world";`, String类的效率会更高一点。

但如果需要拼接大量字符串, StringBuilder无疑是更合适的选择。


讲到这里, Java中的字符串背后的原理就讲得差不多, 相信在了解虚拟机操作字符串的细节后, 你在使用字符串时会更加得心应手. 字符串是编程中一个重要的话题, 本文围绕Java体系讲解的字符串知识只是字符串知识的冰山一角. 字符串操作的背后是数据结构和算法的应用, 如何能够以尽可能低的时间复杂度去操作字符串, 又是一门大学问。