---
tags:
  - embedded
  - contiki
layout: post
title: '在 Cygwin 上编译 Contiki'
category: Embedded
---
由于使用 Contiki 需要 Linux 环境，切换来切换去很麻烦，幸好 Windows 下面有 Unix 模拟环境软件，Cygwin。本文主要介绍 8051 核的 Contiki 编译，它的编译需要编译器 SDCC，所以本文介绍 SDCC 的编译。

<!--more-->

官方 8051 核的编译文档请参考 [8051 Requirements](https://github.com/contiki-os/contiki/wiki/8051-Requirements)。

*注：官方的 win32 版的 SDCC 并不适用。*

## 依赖

必须确定安装以下依赖包，可以从 Cygwin 源中直接找到：

 > * gcc
 > * flex
 > * bison
 > * libboost-graph-dev
 > * python
 > * make
 > * texinfo

此外，还需要安装一个工具，[srecord](http://srecord.sourceforge.net/)，下载 Win32 版本后，直接放于`cygwin/bin/`目录下即可。

## 编译 SDCC

### SDCC 源码下载

直接使用`svn`下载即可，本文版本为**9377**：

```sh
svn co svn://svn.code.sf.net/p/sdcc/code/trunk/sdcc
```

### SDCC 源码修改

 * 编辑`device/lib/incl.mk`，找到：
 
 ```makefile
 MODELS = small medium large
 ```
 
 修改成
 
 ```makefile
 MODELS = small large huge
 ```
 
 * 编译`device/lib/Makefile.in`，找到：
 
 ```makefile
 TARGETS += models small-mcs51-stack-auto
 ```
 
 修改成：
 
 ```makefile
 TARGETS += models model-mcs51-stack-auto
 ```

### 编译

进行配置：

```sh
./configure --disable-gbz80-port --disable-z80-port --disable-ds390-port --disable-ds400-port --disable-pic14-port --disable-pic16-port --disable-hc08-port --disable-r2k-port --disable-z180-port --disable-sdcdb --disable-ucsim
```

编译并安装：

```sh
make
make install
```

## 试用

在 Contiki 的`example/hello-world`目录下运行：

```sh
make TARGET=cc2530dk all
```

便可以生成`hello-world.hex`。

用 SmartRF Flash 工具便可以下载。

## 关于编译 cc-tool

还是别折腾了，因为 cygwin 下，libusb 不支持。