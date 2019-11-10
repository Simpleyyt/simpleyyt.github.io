---
layout: post
title:  面试必问之：Java 中 == 和 equals 的区别你知道吗
tagline: by 迷恋着你微笑的人
categories: 面试题系列
tags: 
    - 迷恋着你微笑的人
---

关于这个问题，一般初中级面试中都会遇到，还记得我当初实习找工作的时候也遇到了这个问题，现在都还记得自己是怎么回答的：== 是基本类型比较，equals 是对象比较，不懂 hashCode，想起来简直惨不忍睹。于是找了一点小时间，研究了一番整理成文章供大家参考。

<!--more-->

## == 是什么？

在《java核心技术卷 1》中将`==`归类于关系运算符；

`==`常用于相同的基本数据类型之间的比较，也可用于相同类型的对象之间的比较；

- 如果`==`比较的是基本数据类型，那么比较的是两个基本数据类型的值是否相等；
- 如果`==`是比较的两个对象，那么比较的是两个对象的引用，也就是两个对象是否为同一个对象，并不是比较的对象的内容；

下面举例说明：

``` java
public class Test {
    public static void main(String[] args){
        // 对象比较
        User userOne = new User();
        User userTwo = new User();
        System.out.println("userOne==userTwo : "+(userOne==userTwo));

        // 基本数据类型比较
        int a=1;
        int b=1;
        char c='a';
        char d='a';
        System.out.println("a==b  :  "+(a==b));
        System.out.println("c==d  :  "+(c==d));
    }
}
```

实体类：

```
class User {
     String userName;
     String password;
}
```

运行结果：

```
userOne==userTwo : false
a==b  :  true
c==d  :  true
```

对象 userOne 和 userTwo 虽然都是 User 的实例，但对应了堆内存的不同区域，因此他们的引用也不同，所以为 false；a 和 b 都是基本类型因此对比的是值，结果为 true ； c 和 d 也是基本类型 同 a 和 b.


## equals 是什么鬼？

在《java核心技术卷 1》中对 Object 类的描述：Object 类是java中所有类的始祖，在java中每个类都是由Object类扩展而来；每个类都默认继承Object类，所以每一个类都有Object类中的方法；从而每一个类都有equals方法；

equals方法主要用于两个对象之间，检测一个对象是否等于另一个对象；

下边来看一看Object类中equals方法的源码：

``` java 

public boolean equals(Object obj) {
        return (this == obj);
    }

```

可以看出来Object类中的equals方法用的还是`==`,也就是比较的两个对象的引用是否相等，并不是根据对象中的属性来判断两个对象是否相等的；也就是说我们自己定义的类中，如果没有重写equals方法，实际上还是用的`==`来比较的两个对象，则用equals方法比较的结果与用==比较的结果是一样的；

java语言规范要求equals方法具有以下特性：

- 自反性。对于任意不为null的引用值x，x.equals(x)一定是true。
- 对称性）。对于任意不为null的引用值x和y，当且仅当x.equals(y)是true时，y.equals(x)也是true。
- 传递性。对于任意不为null的引用值x、y和z，如果x.equals(y)是true，同时y.equals(z)是true，那么x.equals(z)一定是true。
- 一致性。对于任意不为null的引用值x和y，如果用于equals比较的对象信息没有被修改的话，多次调用时x.equals(y)要么一致地返回true要么一致地返回false。
- 对于任意不为null的引用值x，x.equals(null)返回false。


下面再来看一看比较典型的一个类；

- String 

这是jdk中的类，而且该类重写了equals方法；

下面来看一看该类中的equals方法源码：

``` java 

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

从源码可以看出equals方法是进行的内容比较；

举个例子：

``` java
public class Test {
    public static void main(String[] args){
        // 未重写equals方法的类
        User userOne = new User();
        User userTwo = new User();
        System.out.println("userOne.equals(userTwo) : "+(userOne.equals(userTwo)));
        //重写了equals方法的类
        String a="1111";
        String b="1111";
        System.out.println("a.equals(b) : "+(a.equals(b)));
    }
}
```

实体类：

```
class User {
     String userName;
     String password;
}
```

下面是运行结果：

```
userOne.equals(userTwo) :  false
a.equals(b)  : true
```

说明 String 对比的是对象的值。


## hashCode 有什么作用？

`hashCode`也是`Object`类中的方法；下面看一下`hashCode`方法的源码：

```
public native int hashCode();
```

该方法是一个本地方法；该方法返回对象的散列码（int类型）；它的实现是根据本地机器相关的；

下面是百度百科对hash的说明：

> Hash，一般翻译做散列、杂凑，或音译为哈希，是把任意长度的输入（又叫做预映射pre-image）通过散列算法变换成固定长度的输出，该输出就是散列值。这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，所以不可能从散列值来确定唯一的输入值。简单的说就是一种将任意长度的消息压缩到某一固定长度的消息摘要的函数。

> 散列函数能使对一个数据序列的访问过程更加迅速有效，通过散列函数，数据元素将被更快地定位

1. Java对于eqauls方法和hashCode方法是这样规定的： 

- 如果两个对象相同，那么它们的hashCode值一定要相同；
- 如果两个对象的hashCode相同，它们并不一定相同。
- equals()相等的两个对象，hashcode()一定相等；equals()不相等的两个对象，却并不能证明他们的hashcode()不相等。

2. 什么地方使用hashCode?

Hashcode值主要用于基于散列的集合，如HashMap、HashSet、HashTable…等等；

这些集合都使用到了hashCode，想象一下，这些集合中存有大量的数据，假如有一万条，我们向其中插入或取出一条数据，插入时如何判断插入的数据已经存在？取出时如何取出相同的数据？难道一个一个去比较？这时候，hashCode就提现出它的价值了，大大的减少了处理时间；这个有点类似于MySQL的索引；

3. 举例

``` java
public class Test {
    public static void main(String[] args){
        //未重写hashCode的类
        User userOne = new User("aa","11");
        User userTwo = new User("aa","11");
        System.out.println(userOne.hashCode());
        System.out.println(userTwo.hashCode());
        //重写hashCode的类
        String a = new String("string");
        String b = new String("string");
        System.out.println(a.hashCode());
        System.out.println(b.hashCode());
    }
}
```

实体类：

```
class User {
    private String userName;
    private String password;
    public User(String userName, String password) {
        this.userName = userName;
        this.password = password;
    }
}
```

运行结果：

```
752848266
815033865
-891985903
-891985903
```

根据结果可以看出：userOne 和 userTwo 的 hashCode 值不一致;a 和 b 的 hashCode 一致。


以上便是 == 和 equals 的区别，你都 Get 到了吗？


参考文章：

[Java中的equals()和hashcode()之间关系](https://www.hollischuang.com/archives/1290)  
[HashCode的作用原理和实例解析](https://blog.csdn.net/SEU_Calvin/article/details/52094115)  
[Guide to hashCode() in Java](https://www.baeldung.com/java-hashcode)  
[equals() and hashCode() methods in Java](https://www.geeksforgeeks.org/equals-hashcode-methods-java/)  