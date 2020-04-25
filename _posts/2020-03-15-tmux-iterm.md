---
layout: post
categories: Java
title: 整合神器 tmux，帮助你的 iTerm2 成为最帅的那个
tagline: by 小黑
published: false
tags: 
  - 小黑
---

## 前言

阿粉在之前文章中教过大家如何结合 **zsh** 让 **iterm2** 发挥最佳效果。

什么还没有看过？赶紧看下补一下前提知识：[收集了这么多实用技巧，帮助你的 iterm2 成为最帅的那个！](https://mp.weixin.qq.com/s/M72UK9GsYkqOReWD-nokBw)

<!--more-->

上篇文中，阿粉提到每次上线发布的时候，都会打开很多 iTerm 窗口，通过 tab 页拖拽方式让所有窗口可以同时显示。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/00831rSTly1gcogoud5wqj31gf0u0ay7.jpg)

最近阿粉发现一个强大的工具 『**tmux**』，可以仅仅在一个 iTerm 窗口实现上面的多屏的效果。

为什需要使用 tmux？阿粉在网上找到一张有趣的图。

![来源：其他网站](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/00831rSTly1gcogwq4y53j30xx0u01ky.jpg)


## tmux 

tmux 是一个终端复用器（terminal multiplexer），可以让我们在一个 iTerm 窗口运行多个终端程序。

tmux 使用效果如下:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/00831rSTly1gcohf3r4aaj31i10u01ky.jpg)

这里我们需要了解一下 tmux 中三个概念: 

- 会话（**Session**）
- 窗口（**Window**）
- 窗格（**Pane**）

一个会话内我们可以开启多个窗口，而一个窗口内又可以拆分为多个窗格。

**快键前置键**

tmux 中有很多快键键，但是这些快捷键都需要通过一个**前置**快键键唤起。默认**前置**建为 `Ctrl+b`，也就是说每次需要先按下 `Ctrl+b`，然后再按下其他键，快捷键才会生效。

### 安装

tmux 安装方式比较简单，运行如下命令即可：

```bash
# Mac
brew install tmux
# Ubuntu 或 Debian
sudo apt-get install tmux
# CentOS 或 Fedora
sudo yum install tmux
```

### 会话管理

**新建 session**

使用 tmux 之前我们首先需要新建一个 **Session**，命令如下：

```shell
# 新建 session 名称默认为 0
tmux 
# 新建 session，使用 -s 自定义 session 名字
tmux new -s <session-name>
```

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/yd1pg.gif)

**保存会话**

进入会话之后，进行相关操作，比如使用 SSH 连上远端服务器。这时如果想退出去的时候，可以保存当前会话信息。下次可以直接重新进入这个会话，不用重新再次使用 SSH 连接了。

```shell
# 保存当前会话
tmux detach
# 推荐使用快捷键
Ctrl+b d
```

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/kupec.gif)

**接入会话**

**tmux attach**  可以接入上次保存的会话。

```shell
# 默认进入上次保存的会话
tmux attach
# 可以使用 -t 指定会话名字。
tmux attach -t <session-name>
```

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/dzmrs.gif)

**查看会话**

如果之前同时保存了多个会话，我们可以使用 **tmux ls** 查看当前所有会话。

```bash
# 查看会话
tmux ls
```

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/00831rSTly1gcoi3ia55oj30m605y428.jpg)

**杀死会话**

使用 **tmux kill-session** 我们可以杀死某个会话。

```bash
# 默认杀死最近使用会话
tmux kill-session
# 使用 -t 指定会话名称
tmux kill-session -t <session-name>
```

### 窗口管理

**新建窗口**

```bash
# 新建一个指定名称的窗口
tmux new-window -n <window-name>
```

**切换窗口**

```bash
# 切换到指定名称的窗口
tmux select-window -t <window-name>
```

这里推荐使用快捷键 **Ctrl+b w**：从列表中选择窗口。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/00831rSTly1gcoi9gem7nj31i10u0hdt.jpg)

### 窗格操作

我们使用命令可以将一个窗口划分为多个窗格，不过阿粉还是建议使用快键键操作。

```bash
# 划分上下两个窗格 命令：tmux split-window
Ctrl+b %
# 划分上下两个窗格 命令：tmux split-window -h
Ctrl+b "
```

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/raw8f.gif)

**切换 pane**

```bash
## 切换当前所在窗格
Ctrl+b 方向键
```

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/m7i8d.gif)

**窗格大小调整**

```bash
Ctrl+b alt+：方向键 调整窗格大小。
```

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/j6dy7.gif)

**其他窗格常用快捷键**

- `Ctrl+b x`：关闭当前窗格。
- `Ctrl+b !`：将当前窗格拆分为一个独立窗口。
- `Ctrl+b z`：当前窗格全屏显示，再使用一次会变回原来大小。
- `Ctrl+b q`：显示窗格编号。

## 问题

默认配置下，tmux 相关操作只能使用键盘，这对于刚开始使用的小伙伴很不友好。另外如果在分屏的情况下，使用鼠标进行复制粘贴，就会发现复制文本串行的现象。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/dbkl9.gif)

原本只想复制右上窗口的内容，但是复制的时候却串行了，将左边窗口的内容也复制了。对于这种情况我们需要做一些特殊配置。

## 自定义 tmux 配置

我们可以修改 tmux 相关配置开启鼠标，复制/粘贴功能。

不过这里不推荐大家一个个去官网找配置参数，Github 上有个大神开开源其 tmux 配置，我们可以将 tmux 配置如下:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/00831rSTly1gcnzf0tmxtg31d60bi1kx.gif)

**Github 地址：**https://github.com/gpakosz/.tmux

** 安装方法：**

```
$ cd
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s -f .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local .
```

如果需要修改配置，建议在  `~/.tmux.conf.local` 中配置。

上面操作完成之后，重新打开 tmux ，就可以看到界面变化了。若未生效，可以运行如下配置：

```bash
tmux source ~/.tmux.conf
```

> 这个配置下， tmux 默认快键前置键可以使用 `Ctrl+b`，也可以使用  `Ctrl+a`。

默认情况下，未开启鼠标模式，需要使用如下如下快捷键打开； 

```bash
Ctrl+b m
```

开启之后，就可以愉快使用鼠标。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/j5c4b.gif)

## 其他注意点

tmux 会话窗口中，我们是无法访问 macos 系统剪贴板，我们需要安装 **reattach-to-user-namespace**。

安装方式如下：

```shell
$ brew install reattach-to-user-namespace
```

安装成功之后，复制过程中可能碰到以下情况：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/3gsru.png)

我们需要在 **iTerm** 打开如下配置：

![image-20200309220124267](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200315/00831rSTly1gco0wiebbvj317m0u07wh.jpg)

## 总结

这篇文章，阿粉介绍 tmux 的使用方法，合理使用 tmux 可以有效提高日常的生产力。

## 参考文档

1. https://www.ruanyifeng.com/blog/2019/10/tmux.html
2. https://harttle.land/2015/11/06/tmux-startup.html