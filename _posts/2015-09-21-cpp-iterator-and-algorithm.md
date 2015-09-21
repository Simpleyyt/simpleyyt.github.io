---
published: true
title: C++ 之迭代器与算法
category: program
tags: 
  - cpp
  - c
layout: post
---



C++ 有插入迭代器、流迭代器、反向迭代器、移动迭代器，泛型算法结构有适用的迭代器类别：输入迭代器、输出迭代器、前向迭代器、双向迭代器、随机访问迭代器。

<!--more-->

## 再探迭代器

**迭代器**:

 * 插入迭代器
 * 流迭代器
 * 反向迭代器
 * 移动迭代器

### 插入迭代器

```cpp
it = t //在it指定的当前位置插入值t。
*it，++it,it++ //不会做任何事情，都返回it
```
 * **`back_inserter`**: `push_back`的迭代器
 * **`front_inserter`**: `push_front`的迭代器
 * **`inserter`**: 第二个参数指向给定容器的迭代器
 
### 流迭代器

```cpp
istream_iterator<T> in(is); //in从输入流is读取类型为T的值
istream_iterator<T> end;    //读取类型为T的值istream_iterator迭代器，表示尾后位置
in1 == in2  //in1和in2必须读取相同类型
in1 != in2
*in //返回从流中读取的值
in->mem
++in, in++
```

```cpp
ostream_itertor<T> out(os);
ostream_iterator<T> out(os, d); 
out = val
*out, ++out, out++ //不做任何事
```

可以为任何定义了输入运算符(>>)的类型创建`istream_iterator`对象，类似的(<<)可以创建`ostream_iterator`对象。

### 反向迭代器

 > `rbegin`、`rend`、`crbegin`、`crend`
 
 **反向迭代器需要递减运算符**
 
 不可能从一个`forward_list`或一个流迭代器创建反向迭代器。
 
## 泛型算法结构

**迭代器类别**:

 * **输入迭代器**：`==` `！=` `++` `*` `->` 不保迭代器状态。
 * **输出迭代器**：`++` `*` 只能赋值一次。
 * **前向迭代器**：输入和输出迭代器的操作，多次读写，多遍扫描。
 * **双向迭代器**：前置和后置递减运算符(`--`)。
 * **随机访问迭代器**：常量时间访问序列，(`<` `<=` `>` `>=` `+` `+=` `-` `-=` `-` `[]`）。
 
### 算法形参模式

```cpp
alg(beg, end, other, args);
alg(beg, end, dest, other, args);
alg(beg, end, beg2, other, args);
alg(beg, end, beg2, end2, other, args);
```

 * 一些算法使用重载形式传递一个谓词
 * _if版本算法
 * 区分拷贝元素的版本和不拷贝的版本
 
### 特定容器算法

链表类型`list`和`forward_list`定义了几个成员函数形式的算法：`sort` `merge` `remove` `reverse` `unique`。

```cpp
lst.merge(lst2)
lst.merge(lst2,comp)
lst.remove(val)
lst.remove_if(pred)
lst.reverse()
lst.sort()
lst.sort(comp)
lst.unique()
lst.unique(pred)
```

**splice成员**:

```cpp
lst.splice(args)
lst.splice_after(args)
```
