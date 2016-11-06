---
layout: post
tags:
  - linux
title: 'Linux 版网易云音乐无限网络错误的解决'
category: Linux
---
网易云音乐终于发布 Linux 版本了，但是在 elementaryOS 上播放音乐时出现了网络错误。

<!--more-->

网络上说可能是 gstreamer 依赖的问题，但没有具体的解决方法。我乱试了安装 gstreamer 的依赖，竟然让我试成功了！:satisfied:

具体是安装`gstreamer0.10-plugins-good`的依赖：

```sh
sudo apt-get install gstreamer0.10-plugins-good
```