---
title: 'C++ 字面值常量'
layout: post
tags:
  - cpp
category: Program
---
在类型转换和函数匹配中，字面值类型也是非常值得关注的。

<!--more-->

## 整型和浮点型字面值

```c++
20 /* 十进制 */	024 /* 八进制 */	0x14 /* 十六进制 */
```

默认情况下，十进制字面值是带符号数，八进制和十六进制既可能是带符号的也可能是无符号的。

十进制字面值类型是`int`、`long`和`long long`中尺寸最小的。

八进制和十六进制字面值类型是`int`、`unsigned int`、`long`、`unsigned long`、`long long`和`unsigned long long`中的尺寸最小者。

严格来说，十进制字面值不会是负数，负数只是对字面值取负值而已。

默认的，浮点型字面值是一个`double`。

## 转义序列

`\x`后紧跟1个或多个十六进制数字，或`\`后紧跟1个、2个或3个八进制数字。

## 指定字面值的类型

|前缀|含义|类型|
|---|---|---|
|u|Unicode 16 字符|char16_t|
|U|Unicode 32 字符|char32_t|
|L|宽字符|wchar_t|
|u8|UTF-8（仅用于字符串字面常量）|char|

|后缀|最小匹配类型|后缀|类型|
|---|---|---|---|
|u or U|unsigned|f 或 F|float|
|l or L|long|l 或 L|long double|
|ll or LL|long long|

