---
tags:
  - linux
layout: post
title: '如何上传 ppa'
category: Linux
---
简单记录一下 如何上传包到 launchpad 的 ppa上。

<!--more-->

# 生成 debian 目录

在终端中输入：

```sh
dh_make --createorig
```

便会生成`debian`目录，并且有`.orig.tar.gz`源码包。

根据需要修改`debian`目录里面的相应内容。

# 上传 ppa

在终端中输入：

```sh
dput ppa:<userid>/<ppa-name> <source.changes>
```

即可将软件包上传到 PPA 上。