---
category: Daily
layout: post
tags:
  - daily
title: 'Jekyll Editor 使用入门'
---
Jekyll Editor 专门为 Jekyll 博客定制的强大的 markdown 编辑器，它会自动从`<yourname>.github.io`仓库读取`_post`目录下的博客列表，并可以读取、创建、修改博客。

<!--more-->

## 项目

 * **Github**：https://github.com/Simpleyyt/jekyll-editor.git
 * **Chrome 商店**：https://chrome.google.com/webstore/detail/jekyll-editor/dfdkgbhjmllemfblfoohhehdigokocme
 
## 主要功能

 * 获取博文列表，发布、更新、修改博文
 * 博文自动保存到本地
 * 强大的 Markdown 编辑器
 
## 使用说明

![Jekyll Editor](http://simpleyyt.qiniudn.com/15-10-11/10214115.jpg)

左上角为编辑器的常用工具，包含`emoji`表情。右上角的工具依次为登录、全窗口预览、新博文、博文列表、元数据、提交博文，帮助、关于。

首次使用时，必须先进行登录，也就是与 github 相连。

### 博文列表

![博文列表](http://simpleyyt.qiniudn.com/15-10-11/12709365.jpg)

博文列表会自动获取`<yourname.github.io`仓库读取`_post`目录下`<date>-<slug>.md`格式的文件，即为博文。

### 元数据

![元数据](http://simpleyyt.qiniudn.com/15-10-11/85340312.jpg)

即博文的 yaml 格式数据，博文将会以文件名`<date>-<slug>.md`的格式更新。

*注：当“发布”打勾时，才会真正地发布。*

## 已知 Bug

 * 在获取博文列表时，可能会由于多方面原因卡死
 * 预览窗口的滚动条有时会出现问题
 * 发布时可能会卡死