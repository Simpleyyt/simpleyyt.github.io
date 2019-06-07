---
layout: post  
title:  Java中"=="和equals()的区别  
tagline: by 炭烧生蚝  
categories: Java  
tag: 
    - Java
---

“判断两个事物是否相等”，是编程中最常见的操作之一，在Java中，判断是否相等有两种方法，一种是使用“==”判断符，另一种是使用“equals()”方法，你是否曾因混用二者导致不可思议的bug？本篇文章将带你深入二者背后的判断原理。

<!--more-->

# 相等判断符"=="

> "=="相等判断符用于比较基本数据类型和引用类型数据。 当比较基本数据类型的时候比较的是数值，当比较引用类型数据时比较的是引用(指针)。

## "=="判断基本类型数据

基本数据类型指的是Java中的八大数据类型：byte，short，int，long，float，double，char，boolean

这八大基本数据类型有个共同的特点是它们在内存中是有具体值的, 比如说一个 int 类型的数据"2"，它在8位数据总线的机器上保存形式为 0000 0010。（8位机器是假设的）

当使用 == 比较两个基本数据类型的时候, 就是在比较它们各自在内存中的值。

为了照顾到要刨根问底的同学，再补充一下两个数值是怎么比较的：cpu 在比较的时候会将两个值作差，然后查看标志寄存器。标志寄存器存放的是运算的结果，里面有一个是否为0的标志位，如果该位为1，证明二者之差为0，二者相等。

## "=="判断引用类型数据

引用数据类型在字面上也是很好理解的, 它就是一个引用, 指向堆内存中一个具体的对象。

比如说`Student stu = new Student();` 这里的 stu 就是一个引用，它指向的是当前 new 出来的 **Student** 对象. 当我们想要操作这个 **Student** 对象时, 只需要操作引用即可, 比如说`int age = stu.getAge();`。

所以用"=="判断两个引用数据类型是否相等的时候，实际上是在判断两个引用**是否指向同一个对象**。

看下面的示例：

```java
public static void main(String[] args) {
    String s1 = "hello";	//s1指向字符串常量池中的"hello"字符串对象
    String s2 = "hello";	//s2也指向字符串常量池中的"hello"字符串对象
    System.out.println(s1 == s2);   //true

    String s3 = new String("hello");   //s3指向的是堆内存中的字符串对象 
    System.out.println(s1 == s3);	//false
}
```

从上面的例子可以看到，由于引用"s1"和"s2"指向的都是常量池中的"hello"字符串，所以返回true。（后面我会发布一篇详细讲述Java字符串的文章，涉及字符串初始化和字符串常量池等知识）

而"s3"指向的是新创建字符串对象，因为只要动用了`new`关键字, 就会在堆内存创建一个新的对象。

也就是说 s1 和 s3 指向的是不同的字符串对象，所以返回false。

# 相等判断方法equals()

> equals()和 == 有着本质的区别，== 可以看作是对“操作系统比较数据手段”的封装，而equals()则是每个对象自带的比较方法，它是Java自定义的比较规则。

equals()和 == 的本质区别更通俗的说法是：==的比较规则是定死的，就是比较两个数据的值。

而 equals() 的比较规则是不固定的，可以由用户自己定义。

看下面的例子: 

```java
public static void main(String[] args) {
    String s1 = "hello";
    String s3 = new String("hello");    
    System.out.println(s1.equals(s3));	//true
}
```

回想前面的案例：用 == 比较的时候, 上面 s1 和 s3 比较出的结果为false。而当用 equals() 比较的时候，得出的结果为 true。

想知道原因我们还得看源码，下面是 String 类中的 equals() 方法的源码。

```java
public boolean equals(Object anObject) {
    if (this == anObject) {	//先比较两个字符串的引用是否相等(是否指向同一个对象), 是直接返回true
        return true;
    }
    if (anObject instanceof String) {	//两个引用不等还会继续比较
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;	//字符串类是用字符数组实现的, 先要拿到两个字符串的字符数组
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {	//然后对两个数组逐个字符地进行比较
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

从上面的源码可以看到, 当调用 String 类型数据的 equals() 方法时，首先会判断两个字符串的引用是否相等，也就是说两个字符串引用是否指向同一个对象，是则返回true。

如果不是指向同一个对象，则把两个字符串中的字符挨个进行比较。由于 s1 和 s3 字符串都是 "hello"，是可以匹配成功的，所以最终返回 true。


# 思考：为什么要设计equals()方法?

通过上面的讲解，相信你已经知道 == 和 equals() 的区别了：一个的比较规则是定死的，一个是可以由编程人员自己定义的。

可是为什么会有 equals() 方法, 而且还可以被自由定制呢? 

这个问题要落到Java语言的核心 —— 面向对象思想了。

Java 不同于面向过程的C语言，Java是一款面向对象的高级语言。如果是面向过程编程，直接操作内存上存储的数据的话，用 == 所定义的规则来判断两个数据是否相等已经足够了。

而Java中万物皆对象，我们经常要面临的问题是这两个对象是否相等，而不是这两串二进制数是否相等，仅有 == 是完全不够用的。

由于Java程序员们会创建各种满足它们业务需求的对象，**系统无法提前知道两个对象在什么条件下算相等，Java干脆把判断对象是否相等的权力交给编程人员**。


具体的措施是：所有的类都必须继承 Object 类，而 Object 类中写有equals()方法。编程人员可以通过重写 equals() 方法来实现自己的比较策略，也可以不重写，使用Object类的equals()比较策略。


```java
//Object类中的equals()方法源码
public boolean equals(Object obj) {
    return (this == obj);
}
```
从 Object 类的 equals() 源码可以看到，如果编程人员没有显示地重写 equals() 方法，则默认比较两个引用是否指向同一个对象。


> 补充: 关于基本数据类型包装类的比较

由于 Java 中万物皆对象，就连基本数据类型也有其对应的包装类，那么它们对应的比较策略是什么呢？

```java
public static void main(String[] args) {
    int a = 3;
    Integer b = new Integer(3);
    System.out.println(b.equals(a));	//true, 自动装箱
}
```

从上面的代码可以看到尽管两个引用不同, 但是输出的结果仍为 true, 证明 Integer 包装类重写了 equals() 方法，追踪其源码：

```java
//Integer类中的equals方法
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

从源码看到，基本类型包装类在重写equals()后，比较的还是基本数据类型的值。

# 结束

通过探索 == 和 equals() 的区别，我们摸清楚了二者别后的比较策略，同时也对 Java 中 equals() 方法的设计进行了思考，相信大家在今后的 Java 编程实战中不会再为相等判断而烦恼了。