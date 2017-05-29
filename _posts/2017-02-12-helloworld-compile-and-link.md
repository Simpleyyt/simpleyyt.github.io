---
layout: post
tags:
  - cpp
  - compiler
title: 'Hello World 的编译链接概述'
category: Program
---

《程序员的自我修养》读书笔记，简单概述一个 Hello World 程序的编译链接过程。

<!--more-->

有如下的`hello.c`源文件：

```c++
#include <stdio.h>

int main()
{
    printf("Hello World\n");
    return 0;
}
```

可以用 GCC 来编译运行：

```sh
$gcc hello.c
$./a.out
Hello World
```

上述步骤可以分解为 4 个步骤，分别是**预处理（Prepressing）**、**编译（Compilation）**、**汇编（Assembly）**和**链接（Linking）**。

## 被隐藏了的过程

### 预处理

```sh
$gcc -E hello.c -o hello.i
```

或者：

```sh
$cpp hello.c > hello.i
```

预编译过程主要处理那些源代码文件听以“#”开始的预编译指令。

经过预编译后的 .i 文件不包含任何宏定义，因为所有的宏已经被展开，并且包含的文件也已经被插入到 .i 文件中。

### 编译

```sh
$gcc -S hello.i -o hello.S
```

或者使用`cc1`：

```sh
$/usr/lib/gcc/i486-linux-gnu/4.1/cc1 hello.c
```

编译过程就是把预处理完的文件进行一系列**词法分析**、**语法分析**、**语义分析**及**优化**后产生的相应的汇编代码文件。

实际上 gcc 这个命令只是这些后台程序的包装，它会根据不同的参数要求去调用预编译编译程序 cc1、汇编器 as、链接器 ld。

### 汇编

```sh
$as hello.s -o hello.c
```
或者：

```sh
$gcc -c hello.s -o hello.o
```

汇编器是将汇编代码转变成机器可以执行的指令，每一个汇编语句几乎都对应一条机器指令。

### 链接

```sh
$ld -static crt1.o crti.o crtbeginT.o hello.o -start-group -lgcc -lgcc_eh -lc -end-group crtend. crtn.o
```

## 编译器做了什么

编译过程一般可以分为6步：扫描、语法分析、语义分析、源代码优化、代码生成和目标代码优化。

以如下代码为例：

```c++
array[index] = (index + 4) * (2 + 6)
```

### 词法分析

源代码被输入到**扫描器（Scanner）**，进行词法分析，生成一系列的**记号（Token）**。

|记号|类型|
|---|---|
|array|标识符|
|[|左方括号|
|index|标识符|
|]|右方括号|
|=|赋值|
|(|左圆括号|
|index|标识符|
|+|加号|
|4|数字|
|)|右圆括号|
|*|乘号|
|(|左圆括号|
|2|数字|
|+|加号|
|6|数字|
|)|右圆括号|

在识别记号的同时，扫描器也将标识符存放到符号表，将数字、字符串常量存放到文字表等。

有一个叫 lex 的程序可以实现词法分析，它会按照用户之前描述好的词法规则将输入的字符串分割成一个个记号。

### 语法分析

**语法分析器（Grammar Parser）**将上述产生的记号进行语法分析，从而生成**语法树（Syntax Tree）**。

有一个现有的工具叫 yacc（Yet Another Compiler Compiler）可以构建出语法树。


### 语义分析

**语义分析器（Semantic Analyzer）**完成了对表达式的语法层面的分析，即**静态语义（Static Semantic）**，包括声明和类型匹配，类型的转换。

语义分析阶段后，语法树的表达式都被标识了类型，如有隐式转换，则插入相应的转换节点。

### 中间语言的生成

**源码优化器（Source Code Optimizer）**将整个语法树转换成**中间代码（Intermediate Code）**，常见的有**三地址码（Three-address Code）**和**P-代码（P-Code）**，再进行源码级别的优化。

中间代码使用编译器可以被分为前端和后端。编译器前端负责产生机器无关的中间代码，编译器后端将中间代码转换成目标机器代码。

### 目标代码生成与优化

**代码生成（Code Generator）**将中间代码转换成目标机器代码，**目标代码化化器（Target Code Optimizer）**则对目标代码进行优化。

## 静态链接

**链接（Linking）**的主要内容就是把各个模块之间相互引用的部分都处理好。

链接过程主要包括了**地址和空间分配（Address and Storage Allocation）**、**符号决议（Symbol Resolution）**和**重定位（Relocation）**这些步骤。

