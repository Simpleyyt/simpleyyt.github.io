---
title: 'OpenWRT 启动速度优化'
layout: post
tags:
  - openwrt
  - linux
category: Embedded
---

OpenWRT 开机到 WiFi 启动需要花费 20 多秒，实在是太慢了， 对一些简单的应用来说无法接受。经过一些尝试，我把它优化在 8 秒以内。

<!--more-->

## 移除不用的包

可以通过`make menuconfig`来移除一些用不着的软件包，如果不用 OpenWRT 的设置网页可以去掉`luci`软件包，如果不用上网可以移除`firewall`等等。

当然最耗时的还有一些内核模块，比如说 USB 和 I2C 驱动什么的，如果不用可以移除。可以通过`make kernel_menuconfig`移除。

## 启动项

可以在目录`/etc/rc.d/`下查看启动项，不用的可以通过`/etc/init.d/`下的脚本来禁用，比如说`telnet`什么的。

## 关闭 failsafe

如果不用 failsafe，可以关掉，可以省去几秒的等待时间。自行修改`/lib/`下的脚本是没用的，得到 OpenWrt 的源码目录去修改，删掉 failsafe 的相关脚本就行。

## 优化配置

可以优化`/etc/config/`目录下的配置文件，无用的网络接口可以删掉。将 WiFi 的信道改成固定值可以节省非常多的时间。