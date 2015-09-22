---
published: true
layout: post
title: C++对象模型之复制构造函数
category: program
tags: 
  - cpp
---


*“如果一个 class 未定义出 copy constructor，编译器就自动为它产生出一个”* 这句话是不对的，当 class 展现 **bitwise copy semanics** 时，编译器才会产生出来。

<!--more-->

如果一个 class 没有提供 explicit copy constructor，把每一个内建的或派生的 data member 的值，从某个 object 拷贝一份到另一个 object 身上，递归方式施行 **memberwise initialization**。

例如：

```cpp
class String {
    public:
        // ... 没有 explicit copy constructor
    private:
        char *str;
        int len;
}
```

有如下调用：

```cpp
String noun("book");
String verb = noun;
```

则会施行 memberwise initialization：

```cpp
verb.str = noun.str;
verb.len = noun.len;
```
如果一个 String object 被声明为另一个 class 的 member，那么进行 memberwise initialization 时，会递归实施。

什么时候不展现出 bitwise copy semantics，也就是合成 copy constructor 呢，有4种情况：

 > 1. 当 member object 存在 copy constructor。
 > 2. 当 base class 存在 copy constructor。
 > 3. 当 class 声明了 virtual functions 时。
 > 4. 当继承链中有 virtual base class 时。
 
 前面两种情况，在此不做讨论。
 
## class 声明了 virtual functions

这种情况下，可能需要重新设定 Virtual Table 的指针。

举个例子：

```cpp
class ZooAnimal {
public:
    ZooAnimal();
    virtual ~ZooAnimal();
    
    virtual void animate();
    virtual void draw();
};

class Bear : public ZooAnimal {
public:
    Bear();
    void animate();
    void draw();
    virtual void dance();
};
```

有如下使用：

```cpp
Bear yogi;
Bear winnie = yogi;
```

`winnie`会靠 bitwise copy semantics 完成，`winnie`和`yogi`都指向`Bear class`的 virual table。

如果是如下使用：

```cpp
ZooAnimal franny = yogi; // 会发生切割行为
```

`franny`的 vptr 不可以被设定指向`Bear class`的 virtual table，所以需要重新设定。

## Virtual Base Class 的 Subobject

derived class object 的 virtual base class subobject 位置必须维护，bitwise copy semantics 可能会存坏这个位置。

举个例子：

```cpp
class Raccon : public virtual ZooAnimal {
public:
    Raccoon() { }
    Raccoon(int val) { }
};

class RedPanda : public Raccoon {
public:
    RedPanda() { }
    RedPanda(int val) { }
};
```

如果是以下调用：

```cpp
Raccoon rocky;
Raccoon little_critter = rocky;
```

那么 bitwise copy 就可以了。

如果是：

```cpp
RedPanda little_red;
Raccoon little_critter = litter_red;
```

这时候编译器必须安插代码以设定 virtual base class offset 的初值。
