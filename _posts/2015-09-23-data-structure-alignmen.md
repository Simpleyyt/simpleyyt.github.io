---
published: true
layout: post
title: 字节对齐
category: program
tags: 
  - cpp
  - c
---


Linux 沿用的对齐策略是，2字节数据类型（例如`short`）的地址是2的倍数，而较大的数据类型（例如`int`、`int*`、`float`和`double`）的地址必须是4的倍数。

<!--more-->

Windows 要求任何 K 字节基本对象的址址必须是 K 的倍数，K = 2, 4 或者 8。特别的，它要求一个`double`或者`long long`类型数据的地址应该是8的倍数。

## 对齐准则

**四个重要的基本概念：**

 > 1. 数据类型自身的对齐值：char型数据自身对齐值为1字节，short型数据为2字节，int/float型为4字节，double型为8字节。
 > 2. 结构体或类的自身对齐值：其成员中自身对齐值最大的那个值。
 > 3. 指定对齐值：#pragma pack (value)时的指定对齐值value。
 > 4. 数据成员、结构体和类的有效对齐值：自身对齐值和指定对齐值中较小者，即有效对齐值=min{自身对齐值，当前指定的pack值}。

其中，有效对齐值`N`是最终用来决定数据存放地址方式的值。有效对齐`N`表示**“对齐在`N`上”**，即该数据的**“存放起始地址 % N = 0”**。而数据结构中的数据变量都是按定义的先后顺序存放。第一个数据变量的起始地址就是数据结构的起始地址。结构体的成员变量要对齐存放，结构体本身也要根据自身的有效对齐值圆整(即结构体成员变量占用总长度为结构体有效对齐值的整数倍)。

**例1：**
```cpp
struct A{
    int    a;
    char   b;
    short  c;
};
struct B{
    char   b;
    int    a;
    short  c;
};
```

**结果：**`sizeof(strcut A)`值为8；`sizeof(struct B)`的值却是12。 

**例2：**
```cpp
#pragma pack(2)  //指定按2字节对齐
struct C{
    char  b;
    int   a;
    short c;
};
#pragma pack()   //取消指定对齐，恢复缺省对齐
```

**结果：**`sizeof(struct C) = 8`。

## 栈内存对齐

在VC/C++中，栈的对齐方式不受结构体成员对齐选项的影响。总是保持对齐且对齐在4字节边界上。（并未考证64位）

## 位域对齐

位域成员不能单独被取`sizeof`值。下面主要讨论含有位域的结构体的`sizeof`。

C99 规定`int`、`unsigned int`和`bool`可以作为位域类型，但编译器几乎都对此作了扩展，允许其它类型的存在。位域作为嵌入式系统中非常常见的一种编程工具，优点在于压缩程序的存储空间。

**其对齐规则大致为：**

  > 1. 如果相邻位域字段的类型相同，且其位宽之和小于类型的`sizeof`大小，则后面的字段将紧邻前一个字段存储，直到不能容纳为止；
 > 2. 如果相邻位域字段的类型相同，但其位宽之和大于类型的`sizeof`大小，则后面的字段将从新的存储单元开始，其偏移量为其类型大小的整数倍；
 > 3. 如果相邻的位域字段的类型不同，则各编译器的具体实现有差异，VC6 采取不压缩方式，Dev-C++ 和 GCC 采取压缩方式；
 > 4. 如果位域字段之间穿插着非位域字段，则不进行压缩；
 > 5. 整个结构体的总大小为最宽基本类型成员大小的整数倍，而位域则按照其最宽类型字节数对齐。

**例3：**

```cpp
struct BitField{
    char element1  : 1;
    char element2  : 4;
    char element3  : 5;
};
```
位域类型为`char`，第1个字节仅能容纳下`element1`和`element2`，所以`element1`和`element2`被压缩到第1个字节中，而`element3`只能从下一个字节开始。因此`sizeof(BitField)`的结果为2。

**例4：**

```cpp
struct StructBitField{
    int element1   : 1;
    int element2   : 5;
    int element3   : 29;
    int element4   : 6;
    char element5  :2;
    char stelement;  //在含位域的结构或联合中也可同时说明普通成员
};
```

位域中最宽类型`int`的字节数为4，因此结构体按4字节对齐，在 VC6 中其`sizeof`为16。

**例5：**

```cpp
struct BitField4{
    char element1  : 3;
    char  :0;
    char element3  : 5;
};
```

长度为0的位域告诉编译器将下一个位域放在一个存储单元的起始位置。如上，编译器会给成员`element1`分配3位，接着跳过余下的4位到下一个存储单元，然后给成员`element3`分配5位。故上面的结构体大小为2。

---

*本文参考：<http://www.cnblogs.com/clover-toeic/p/3853132.html>*
