---
tags:
  - linux
layout: post
title: 'Shadowsocks eOS/Ubuntu 客户端'
category: linux
---
Shadowsocks-Qt5 太难用了，趁着有时间，写了一个跟 MacOS 上差不多的客户端，由于 shadowsocks-libev 太难调用，因此都是暴力 shell 调用。

<!--more-->

项目地址：<http://simpleyyt.github.io/shadowsocks-eos/>。

由于是命令行调用，所以先必须安装`shadowsocks-libev`，这个好像被墙了，比较难安装。

还必须安装`gfwlist2pac`用于 GFWList 的更新。

## 功能

 * Shadowsocks 指示器，可以开关 Shadowsocks
 * 全局代理与自动代理模式
 * 编辑服务器
 * 获取GWFList更新

## 安装

首先必须先安装`shadowsocks-libev`与`gfwlist2pac`。

`shadowsocks-libev`的安装参考：<https://github.com/shadowsocks/shadowsocks-libev>。

`gfwlist2pac`可以通过`pip`安装。

然后，下载 Shadowsocks 的 deb 包进行安装：<https://github.com/simpleyyt/shadowsocks-eos/releases>。

## Bugs

代码很暴力，凑合能用就行。