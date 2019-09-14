---
layout: post
category: Java
title: 记一次使用 Arthas 热更新线上代码（误）
tagline: by 惊奇
tags: 
  - Java
published: true
---

> 引用参考第二条 -  Arthas提醒您： 诊断千万条，规范第一条，热更不规范，同事两行泪

<!--more-->

# 起因
在一次迭代中，出现了一个低级错误，`if` 语句中的判断逻辑出现了错误，刚好这个功能场景在开发和测试过程中很少触发，使用的用户也不多，不过的确影响到了少部分用户，所以还是需要进行修复。

想着只是更新一行代码，如果走正常的发布流程，需要通过以下步骤：

> 提交代码 -> 提测打包 -> 测试环境git验证 -> Release 环境验证 -> 预发环境验证 -> 线上环境

**如果你的应用体积不小，而且线上机器很多，花费的时间可能足够喝很多杯 `Java` :-O**

---
# Arthas
之前使用过 `Alibaba` 开源的诊断工具 `Arthas` ，下图是官方文档中提到的功能：

![arthas_summary](http://www.justdojava.com/assets/images/2019/java/image_yjq/arthas/arthas_summary.png)

**不仅可以用来排查问题，还能够使用它 `redefine` 进行热更新。**

**刚好之前也看到一篇文章介绍如何进行 `一条龙更新`，所以就开始了尝试，先从本地开发测试开始。**

---
# 选择方案

## 方案一：jad/mc/redefine线上热更新一条龙

开发时写下的 `java` 程序是高级语言，需要通过编译生成 `.class` 文件才能在 `jvm` 中运行。

**所以在一个运行中的程序中进行热更新，需要先将它使用 `jad` [Java decompile]反编译，修改 `.java` 文件后使用 `mc` [Memory complile] 编译出 `.class` 文件，最后使用 `redefine` 命令更新虚拟机中的程序。**

首先可以跟着教程来一次尝试 [https://alibaba.github.io/arthas/arthas-tutorials?language=cn&id=arthas-advanced](https://alibaba.github.io/arthas/arthas-tutorials?language=cn&id=arthas-advanced)

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/arthas/arthas_tutorials.png)

```bash
# 反编译
$ jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java
# 修改文件
$ vim /tmp/UserController.java
# 查找加载的 ClassLoader
$ $ sc -d *UserController | grep classLoaderHash
 classLoaderHash   6bc28484
# 编译
$ mc -c 6bc28484 /tmp/UserController.java -d /tmp
# 热更新
$ redefine /tmp/com/example/demo/arthas/user/UserController.class
redefine success, size: 1
```

跟着教程 `demo`，发现代码逻辑被修改，返回我修改后的结果，心里在狂喜，可以不用喝这些多咖啡！

---
### 实际项目中反编译失败
但在工作中的项目中使用，发现出现了这个问题：**反编译后的类不完整**

查看通过 `jad` 命令反编译后的文件：
```bash
/*
 * Decompiled with CFR.
 *
 * Could not load the following classes:
 * ....
 */
```

在不完整的文件中进行修改后，进行 `mc` 命令编译，将会提示如下：

```bash
[arthas@7281]$ mc -c 6bc28484 /tmp/xxxx.java -d /tmp
Memory compiler error, exception message: Compilation Error
......
, please check $HOME/logs/arthas/arthas.log for more details.
Affect(row-cnt:0) cost in 884 ms.
```

可以看到，如果有复杂的类，并一定能够成功反编译，遭遇了失败，开始排查原因

---
### 反编译失败原因

开源的好处的是，大家可以一起参与到其中，提出问题和解决问题，在 `github` 项目 `arthas` 的 `issue` 中，通过关键字 `jad 反编译` 找到了原因

[横云断岭 Arthas源码分析--jad反编译原理](https://github.com/alibaba/arthas/issues/763)：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/arthas/jad_defects.png)

**可能好巧不巧，实际项目中的代码，就遇上了这 1% 情况（顺便看了其它类，发现这种情况还不少...)**

当时对于 `jad` 机制是有点不了解，所以想先通过上面的提示的工具 `dumpclass` 去精确获取 `java` 字节码，但奈何有些难用，尝试了几遍还是没能拿下来，于是开始换种思路。

---
## 方案二：直接拿到 `.class` 文件

既然前面的操作，获取修改后的 `.class` 文件，都是为了最后一步 `redefine` 所服务，那只要获取精确的 `.class` 文件就可以了，跳过前面的步骤也可以。

**于是与一个前辈讨论后，他建议我使用 `IDEA` 工具编译后的 `.class` 文件**

于是将本地代码进行修改，进行打包编译，得到想要的 `.class` 文件，然后将这个文件上传到测试环境，进行替换。

``` bash
[arthas@63]$ redefine /tmp/xxxxxx.class
redefine success, size: 1
```

**出现了成功的提示，同时进行了接口测试，发现执行的代码是修改后的逻辑！**

---
# 总结

通过上面说的，正常来说只需要简单四步就能进行热更新

**一、使用 `jad` 反编译出 `.java` 文件**

**二、编辑文件，修改逻辑**

**三、使用 `mc` 编译修改后的文件**

**四、使用 `redefine` 热更新**

当然，也可能遇到 `jad` 反编译失败的场景，可以参考方案二，直接拿到修改后的 `.class` 文件，然后继续进行操作。

---
# 题外话
标题最后一个字为什么叫【误】呢，是因为我经过测试，跟前辈们讨论后，在发布流程规范、 `IDEA` 提供的 `.class` 文件不确定性还有工具误用的把控上考虑，觉得目前不适合使用，于是还是走了正常的发布流程。

**最后，Arthas提醒您： 诊断千万条，规范第一条，热更不规范，同事两行泪**

**以后要避免再犯这些小错误，恩，换言之，我喝了很多杯 `Java` :-(**

---
# 参考资料

1. [Arthas 用户文档](https://alibaba.github.io/arthas/index.html)

2. [jad/mc/redefine线上热更新一条龙](http://hengyunabc.github.io/arthas-online-hotswap/)

3. [横云断岭 Arthas源码分析--jad反编译原理](https://github.com/alibaba/arthas/issues/763)