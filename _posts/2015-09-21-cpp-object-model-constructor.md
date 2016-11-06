---
published: true
layout: post
title: C++ 对象模型之构造函数
category: Program
tags: 
  - cpp
---

看看以下这段代码：

```cpp
class Foo { public: int val; Foo *pnext; };

void foo_bar()
{
    Foo bar;
    if (bar.val || bar.pnext )
        // ... do somthing
    // ...
}
```

上述程序并不会合成出一个 default constructor。什么时候会合成出 default constructor 呢，下面分4种情况。

<!--more-->

## 带有 Default Constructor 的 Memeber Class Object

编译器需要为该 class 合成一个 default constructor，不过这个合成操作只有在 constructor 真正需要被调用时才会发生。

合成的 default constructor、copy constructor、destructor、assignment copy operator 都以`inline`方式完成，如果函数太复杂，不适合做成`inline`，就会合成出 explicit non-inline static 实例。

举个例子：

```cpp
class Foo { public: Foo(); Foot(int ) ... };
class Bar { public: Foo foo; char *str; };

void foo_bar()
{
    Bar bar;
}
```

![对象关系](http://simpleyyt.qiniudn.com/15-9-22/65375122.jpg)

编译器会为`class Bar`合成一个 default constructor 来处理 `Bar::foo`，但它并不初始化`Bar::str`。

合成的 default constructor 可能像这样：

```cpp
inline
Bar::Bar()
{
    foo.Foo::Foo();
}
```

假设程序员提供了 default constructor：

```cpp
Bar::Bar() { str = 0; }
```

则编译器会扩张已存在的 constructors：

```cpp
Bar::Bar()
{
    foo.Foo::Foo();
    str = 0;
}
```

如果有多个 class member objects 都要求 constructor 初始化操作，C++ 语言将以 **member objects 在 class 中的声明顺序**来调用各个 constructors。

## 带有 Default Constructor 的 Base Class

将会合成 Default Constructor，会根据 base class 声明的顺序调用 base class 的 default constructor。

如果有多个 constructors，编译器会扩张现有的每一个 constructors。

## 带有一个 Virtual Function 的 Class

以下两种情况，也需要合成出 default constructor：

 > 1. class 声明（或继承）一个 virtual function。
 > 2. class 派生自一个继承串链，其中有一个或更多的 virtual base classes。
 
 举个例子：
 
```cpp
class Widget {
public:
    virtual void flip() = 0;
    // ...
};

void flip( const Widget& widget ) { widget.flip(); }

void foo()
{
    Bell b;
    Whistle w;
    
    flip(b);
    flip(w);
}
```

![类图](http://simpleyyt.qiniudn.com/15-9-22/58211208.jpg)

编译期间发生两个扩张：

 > 1. virtual function table
 > 2. pointer member （也就是 vptr ）

`flip` 函数可能被改写如下：

```cpp
( *widget.vptr[1] )( &widget )
```

为了让这个机制发挥功效，编译器必须为每一个 Widget（或其派生类）object 的 vptr 设置初值，放置适当的 virtual table 地址。对于 class 所定义的每一个 constructor，编译器会安插一些代码来做这样的事情，如果没有 construcotr，则合成一个。

## 带有一个 Virtual Base Class 的 Class

例如以下代码：

```cpp
class X { public: int i; };
class A : public virtual X { public: int j; };
class B : public virtual X { public: double d; };
class C : public A, public B { public: int k; };

void foo(const A* pa) { pa->i = 1024; }

main()
{
    foo(new A);
    foo(new C);
}
```
`foo()`可能被改写如下：

```cpp
void foo(const A* pa) { pa->_vbcX-> = 1024; }
```

class 所定义的每一个 constructor，编译器会安插代码来初始化`_vbcX`，如果没有 constructors，编译器必须合成一个 default constructor。

## 总结

C++ 新手一般有两个常见的误解：

 > 1. 任何 class 如果没有定义 default constructor，就会被合成出一个来。
 > 2. 编译器合成出来的 default constructor 会显式设定 class 内每一个 data member 的默认值。
 
如你所见，没有一个是真的。

