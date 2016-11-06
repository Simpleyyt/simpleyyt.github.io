---
tags:
  - linux
  - embedded
layout: post
title: 'Ubuntu 上自定义特殊波特率'
category: Embedded
---
在做嵌入式开发时，经常会需要特殊的串口波特率，比如 esp8266 的 74880 波特率。Ubuntu 下的 minicom 与 screen 都不支持这个波特率，可以使用`setserial`将特殊波特率映射到 38400。

<!--more-->

我简单试了一下，PL2303 和 CH341 都不支持用`setserial`进行特殊波特率的设置，而 FTDI 支持。

首先，先安装`setserial`：

```sh
sudo apt-get install setserial
```

查看`base_baud`：

```sh
sudo setserial -a /dev/ttyUSB0
```

我得到的`base_baud`为 24000000。

然后，进行分频：

```sh
sudo setserial -v /dev/ttyUSB0 spd_cust divisor $((24000000/74880))
```

就可以得到特殊波特率 74880，获取其它波特率的方法过程类似。

然后，只需用波特率 38400 进行连接即可。