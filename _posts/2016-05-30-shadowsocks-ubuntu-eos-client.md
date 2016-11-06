---
tags:
  - linux
layout: post
title: 'Shadowsocks eOS/Ubuntu 客户端'
category: Linux
---
Shadowsocks-Qt5 太难用了，趁着有时间，写了一个跟 MacOS 上差不多的客户端，都是暴力 shell 调用。

<!--more-->

项目地址：<http://simpleyyt.github.io/shadowsocks-eos/>。

Shadowsocks elementaryOS/Ubuntu（也许可以） 客户端（指示器）。

## 截图

![](https://raw.githubusercontent.com/Simpleyyt/shadowsocks-eos/master/screenshot/screenshot.png)

## 功能

 * Shadowsocks 指示器，可以开关 Shadowsocks
 * 全局代理与自动代理模式
 * 编辑服务器
 * 获取GWFList更新

## 安装

首先必须先安装`shadowsocks`与`gfwlist2pac`，可以通过`pip`安装：

```sh
sudo pip install shadowsocks gfwlist2pac
```

然后，下载 Shadowsocks 的 deb 包进行安装：<https://github.com/simpleyyt/shadowsocks-eos/releases>。

## Bugs

代码很暴力，凑合能用就行。
