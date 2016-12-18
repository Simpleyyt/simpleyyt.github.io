---
title: 'C++ 函数重载与函数匹配'
layout: post
tags:
  - cpp
category: Program
---
《C++ Primer》笔记，整理关于函数重载与函数匹配的笔记。

<!--more-->

## 函数重载

```c++
void func(int a); //原函数

void func(double a); //正确：形参类型不同
void func(int a, int b); // 正确：形参个数不同
int func(int a); //错误：只有返回类型不同

typedef int int32;
void func(int32 a); //与原函数等价：形参类型相同
void func(const int a); //与原函数等价：顶层 const 将被忽略
void func(int); //与原函数等价：只是省略了形参名字
```

**函数重载有如下的规则：**

 * 名字相同，形参类型不一样。
 * 不允许两个函数除了返回类型外其他所有的要素都相同。
 * 顶层`const`的形参无法和没有顶层`const`的形参区分。

其中返回类型不同时编译时会出错，而类型别名、项层`const`、省略形参名字只是重复声明而已，只要不定义，编译就不会出错，比如：

```c++
//只定义了其中一个
void func(int a);
void func(const int a) {}
```

## 函数匹配

### 名字查找

函数匹配的第一步便是**名字查找（name lookup）**，确定**候选函数**。

名字查找有两方面：

 * 常规查找（normal lookup）
 * 实参决定的查找（argument-dependent lookup，ADL）

所有函数调用都会进行常规查找，只有函数的实参包括类类型对象或指向类类型对象的指针/引用的时候，才会进行实参决定的查找。

**常规查找**

```c++
void func(int a);	//1
namespace N
{
	//作用域
	void func() {}	//2
	void func(double a) {}	//3
	...
	void test1()
	{
		func(); //候选函数为函数2和3
	}
	
	void test2()
	{
		using ::func; //将函数1加入当前作用域
		func(); //候选函数为函数1
	}
	...
}
```

从函数被调用的局部作用域开始，逐渐向上层寻找被调用的名字，一旦找到就停止向上寻找，将找到的所有名字加入候选函数。

此外，using语句可以将其他作用域的名字引用到当前作用域。

**ADL查找**

```c++
void func() {} //1
//第一个实参所在命名空间
namespace Name1 {
    class T {
        friend void func(T&) {} //2
    };
    void func(T) {} //3
}
//第二个实参的间接父类所在命名空间
namespace Name00 {
    class T00 {
        friend void func(int) {} //4
    };
    void func() {} //5
}
//第二个实参父类所在命名空间
namespace Name0 {
    class T0:public Name00::T00 {
        friend void func(int) {} //6
    };
    void func() {} //7
}
//第二个实参所在命名空间
namespace Name2 {
    class T:public Name0::T0 {
        friend void func(T&) {} //8
    };
    void func(T) {} //9
}
void test()
{
    Name1::T t1;
    Name2::T t2;
    //9个函数全是候选函数  
    //第1个函数是normal lookup找到的
    //后8个函数全是argument-dependent lookup找到的
    func(&t1,t2);
}
```

从第一个类类型参数开始，依次遍历所有类类型参数。对于每一个参数，进入其类型定义所在的作用域（类内友元函数也包括在内），并依次进入其基类、间接基类……定义所在的作用域，查找同名函数，并加入候选函数。

**注意：**在继承体系中上升的过程中，不会因为找到同名函数就停止上升，这不同于常规查找。

类中的运算符重载也遵循 ADL 查找，其候选函数集既包括成员函数，也应该包括非成员函数。

```c++
namespace N
{
	class A
	{
	public:
		void operator+(int a) {} //1
	};
	void operator+(A &a, int a) {} //2
};
void operator+(A &a, int a) {} //3

void test()
{
	N::A a;
	a + 1; //1、2、3都是候选函数
}
```

### 确定可行函数

第二步便是从候选函数中选出可行函数，选择的标准如下：

 * 形参数量与本次调用提供的实参数量相等
 * 每个实参的类型与对应的形参类型相同，或者能转换成形参类型

```c++
//以下为候选函数
void func(int a, double b) {} //可行函数
void func(int a, int b) {} //可行函数：实参可转化成形参类型
int func(int a, double b) {} //可行函数

void func(int a) {} //非可行函数：形参数量不匹配
void func(int a, int b[]) {} //非可行函数：实参不能转换成形参

void test()
{
	func(1, 0.1);
}
```

### 寻找最佳匹配

从可行函数中选择最匹配的函数，如果有多个形参，则最佳匹配条件为：

 * 该函数每个实参的匹配都不劣于其他可行函数需要的匹配。
 * 至少有一个实参的匹配优于其他可行函数提供的匹配。

否则，发生二义性调用错误。

```c++
//可行函数
void func(int a, float b) {}
void func(int a, int b) {}

void test()
{
	func(1, 0.1); //二义性错误：double 向 int 的转换与向 float 的转换一样好
	func(1, 1); //调用 void func(int a, int b)
}
```

为了确定最佳匹配，实参类型到形参类型的转换等级如下：

 1. 精确匹配：
   * 实参类型和形参类型相同。
   * 实参从数组类型或函数类型转换成对应的指针类型。
   * 向实参添加顶层`const`或者从实参中删除顶层`const`。
 2. 通过`const`转换实现的匹配。
 3. 通过类型提升实现的匹配。
 4. 通过算术类型转换或指针转换实现的匹配。
 5. 通过类类型转换实现的匹配。

*一般不会存在这个阶段不会同时存在两个以上的精确匹配，因为两个精确的匹配在本质上是等价的，在定义重载函数时，编译器可能就报出重定义的错误了。*

挑几个重点的来详细说一下。

**指针转换实现的匹配**

 * 0 或`nullptr`能转换成任意指针类型。
 * `T *` 能转换成 `void *`，`const void *`转换成`const void*`。
 * 派生类向基类类型的转换。
 * 函数与函数指针的形参类型必须精确匹配。

**类类型转换实现的匹配**

两个类型提供相同的类型转换将产生二义性问题。

```c++
struct B;
struct A
{
	A() = default;
	A(const B&);	//把一个 B 转换成 A
};

struct B
{
	operator A() const; // 也是把一个 B 转换成 A
};

A f(const A&);

B b;
A a = f(b); //二义性错误：f(B::operator A()) 还是 f(A::A(const B&))

A a1 = f(b.operator A()); //正确：使用 B 的类型转换运算
A a2 = f(A(b)); //正确：使用 A 的构造函数
```

类当中定义了多个参数都是算术类型的构造函数或类型转换运算符，也会产生二义性问题。

```c++
struct A
{
	A(int = 0);
	A(double);
	operator int() const;
	operator double() const;
};

void f(long double);

A a;
f(a); //二义性错误：f(A::operator int()) 还是 f(A::operator double())？

long l;
A a2(l); //二义性错误：A::A(int) 还是 A::A(double)？

short s;
A a3(s); //正确：使用 A::A(int)
```

当我们使用两个用户定义的类型转换时，如果转换函数之前或之后存在标准类型转换，则标准类型转换将决定最佳匹配到底是哪个。