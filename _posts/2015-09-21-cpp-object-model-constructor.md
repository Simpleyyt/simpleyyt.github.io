---
published: false
layout: post
title: C++ 对象模型之构造函数
category: program
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

上述程序并不会合成出一个 default constructor，什么时候会合成出 default constructor 呢，下面分4种情况。

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