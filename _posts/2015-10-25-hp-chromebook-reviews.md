---
tags:
  - daily
layout: post
title: 'HP Chromebook 11 折腾体验'
category: daily
---
前不久淘了一台 Chromebook，hp chromebook 11，ARM 架构。外观挺新的，轻巧方便。11吋的屏幕，可以随身携带。只带 ChromeOS，开机快速，浏览网页也挺快，挺流畅的，基本不卡。

<!--more-->

## 翻墙问题

本来用的是红杏进行翻墙，后来再阅兵前红杏挂了，一直都没有找到替找品，所幸学校的网络是可以上 Google 的，所以也就没有太在意。:sunglasses:

## 电池及续航

说实话，这台二手的 Chromebook续航能力不是很好，充满电也就能用3、4个小时，由于没有配充电器，用1A以下的充电器根本充不进去；用2A的充电器充电时，也会提示低功率充电。

这算是这台机子不尽人意的一个地方。

## 能否用作写代码

作为程序员，最关心的便是能否用它来写代码。Chromebook 用来写 javascript 还是不错的，毕竟 ChromeOS 本身就是 Chrome 浏览器，对于 Web 开发者来说是福音，Chrome Web App Store 上有好多 IDE。

除了 Javascript 之外，还有 python shell，lua shell，等等可以玩玩，但是做工程就算了吧。

在 ChromeOS 上写 Chrome 应用也不错，应用商店提供了 Develop IDE，只是在本 Chromebook 上跑起来有点卡:worried:。

ChromeOS 还提供了一个 NaCl Development Environment，上面有一本堆 shell 工具，比 crosh 好太多了，有 git，vim 之类的，但是不能访问下载目录。在本 chromebook 也不支持 gcc 编译器:weary: :sob:。

## 开发者模式与 crouton

可以进行开发模式，这样的 crosh 就可以支持 linux shell 一样的指令。但是每次开机都得按 Ctrl + D，挺麻烦的，所以就没搞过。

crouton 是以 chroot 的方式，在 ARM 架构下也没有什么用，而且性能也不高，所以还是建议不要折腾。

## Web IDE

写代码的另一种方式当然是 Web IDE 了，我试用了一下，Cloud9，Koding 都挺不错的，但是在墙内可能有点慢。

## ARC

在 Chromebook 运行 Android 应用，缺点是大部分运行都出问题了，而且速度也挺慢，分辨率感人。

## 感受

整台 hp chromebook 11 的感觉还是不错的，至少比平板好，而且这个价，值得。