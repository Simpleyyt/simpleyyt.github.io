---
layout: post
title: 收集了这么多实用技巧，帮助你的 iterm2 成为最帅的那个！
categories: tool
tags:
	- 惊奇
---

## 1 前言

**一个适合后端仔排查问题的 iterm2 终端应该是这样的：**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/itemr2/eight_terminal.png)

交代下为啥要开这么多个窗口，目前阿粉我们的应用是单机部署，一个服务部署在很多台 Linux 服务器上，构建分布式架构。（**实际上服务器数量比这个更多:-O**）

<!--more-->

当然，公司也有日志监控平台，从 UI 上查询信息也很方便，但看过《鸟哥的 Linux 私房菜》后，阿粉感觉里面的命令十分有用，所以在排查问题时，更喜欢直接在终端进行信息统计（grep、awk、sed、wc 和正则表达式来一套），而且终端查询比网页查询更快，所以决定好好改造 iterm2 这个工具！

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/itemr2/pic1.jpg)

## 2 安装 zsh

iterm2 被称为 Mac 下最好用的终端软件，阿粉觉得更重要的是它与 zsh 的结合，才让 iterm2 变得这么好用！ 

![](https://camo.githubusercontent.com/5c385f15f3eaedb72cfcfbbaf75355b700ac0757/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6f686d797a73682f6f682d6d792d7a73682d6c6f676f2e706e67)

在 github 官网上（https://github.com/ohmyzsh/ohmyzsh），它是这么介绍它的：

> oh my zsh 是一个开放源代码，社区驱动的框架，用于管理zsh配置。
> 一旦完成安装，你的终端 shell 一定会成为村里最靓的那个仔（不是的话你可以找我退款！）
> 在命令提示符中进行每次输入，您都将利用数百个功能强大的插件和精美的主题。在咖啡馆时，会有陌生人找你，问你：“太好了！你是不是天才？”

当然，上面的介绍是 「谷歌专业翻译+个人渣翻译」，就是为了突出 zsh 的强大，下面来看下具体如何配置吧

### 2.1 个性化打造

关于 **「iTerm2 + Oh My Zsh 打造舒适终端体验」**，这篇文章已经说得很全面了：https://segmentfault.com/a/1190000014992947

按照上面的提示和步骤，一步一步往下实践，也成功完成了自己的 iterm2 改造：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/iterm2/effect_diagram.png)

来介绍下上面出现过的插件：

- PowerLine：
Powerline 是一个比较酷炫的状态栏工具，主要用于显示状态行和提示信息（上面蓝色的状态栏，当然还有其它主题，读者慢慢去发掘吧）：https://github.com/powerline/powerline
- PowerFonts：
它是一个字体库，用来替换字体，有些主题需要字体的配合，所以从中选择一款好看且兼容主题的字体吧：https://github.com/powerline/fonts
- Solarized 配色：
原来的背景颜色是黑白的，看起来不够冲击力，所以可以从下面的调色板中，选择一款自己喜欢的配置：https://github.com/altercation/solarized
- 选择主题样式：
Robby 主题是默认的主题。但经过博主推荐，我也喜欢上了使用 agnoster 主题。更多主题选择截图请参考该链接：https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
- highlighting：
它是一个代码高亮插件，例如输入正确的 git 命令，字体颜色显示的是黄色，而输入 gti 错误命令时，它将显示红色。它能提前检查我们输入的命令是否正确，让我们得到更快提示：https://github.com/zsh-users/zsh-syntax-highlighting
- autosuggestions：
自动补全功能，它将会记录我们输入过的命令，用于建议和补全，例如效果图中，输入命令 cd ，按下方向右键，就能将命令补全，或者方向上下键，可以选择曾经定位过的路径：https://github.com/zsh-users/zsh-autosuggestions

由于博主写的图文并茂，每一步都有详细操作，阿粉就不重复记录啦，建议大家好好看着步骤进行配置，也可以在每个步骤中选择自己喜欢的插件，做自己的个性化配置~

### 2.2 去掉提示前缀

在配置完后，终端的提示状态栏，存在着一长串前缀提示：xxxx@MacBook-Pro，它是每个人的电脑名字，如果遇到定位的路径长，加上主机名，一整行就放不下，于是找到了消除前缀的办法：

- 定位到主题文件：
打开「访达」，点击顶部状态栏「前往 -> 前往文件夹」（或者按快捷键 Shift + ⌘（command） + G)，接着定位到该目录：~/.oh-my-zsh/themes/

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/iterm2/themes_directory.png)

例如阿粉选择的是 agnoster 主题，于是打开了 agnoster.zsh-theme 文件，然后在最底部的添加了注释符：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/iterm2/remove_prompt_context.png)

加上 # 前缀后，注释掉上下文提示符，然后记得保存，接着打开新的终端窗口，就看不到长长的主机名啦~


## 3 免密登录

既然是远端 Linux 服务器，我们需要通过 SSH 进行登录，常用以下形式进行操作：

```shell
$ ssh root@ip
# if you first login, alter this
The authenticity of host 'host (xxx)' can't be established.
...
Are you sure you want to continue connecting (yes/no)?

#after enter

[entery your password]
```

上面的 root 是登录角色，输入第一行后，如果是第一次登录，会提示你是否确认保存"公钥指纹"，点击回车后，就可以输入登录密码，然后登录。

使用公钥加密是为了 SSH 安全登录，具体加密算法和原理，可以查看该篇文章：[SSH原理与运用（一）：远程登录 http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

可是每次都要进行这样重复的操作，输入 ssh 命令后还要继续输入密码，浪费了宝贵的三秒时间，于是找到了「免密登录」，输入指令后，不需要再输入密码。

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/itemr2/pic2.jpg)

### 3.1 原理

前面提到了“公钥”，ssh 提供了公钥登录，通过公钥登录可以不需要输入密码直接登录

> 公钥登录：
> 用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。**远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录 shell，不再要求密码。**

### 3.2 操作

- **客户端生成公钥**

```shell
$ ssh-keygen -t rsa
```

上述语句输入后，中途会提示你是否需要输入私钥，个人开发时比较少用到，所以可以直接回车，最后会提示你成功生成公钥和私钥。

具体文件生成路径在 `~/.ssh`

```UML
├── authorized_keys     # 已认证的 key
├── id_rsa              # 生成的私钥
├── id_rsa.pub          # 生成的公钥，待会被复制过去的就是这个
└── known_hosts         # 已保存的公钥对应 ip 地址
```

- **拷贝公钥到服务器**

```shell
$ ssh-copy-id -i id_rsa.pub root@ip
```

在本地终端上，输入 ssh-copy-id 命令，将本地公钥拷贝到远端服务器上，成功拷贝后，你将会在远端服务器上的 ~/.ssh/authorized_keys 文件上看到自己电脑的公钥信息。

接着下一次登录，直接输入 ssh 命令，不需密码就能直接登录到服务器~

## 4 使用脚本登录

如果有小伙伴的登录方式跟我们相似，是通过跳板机，服务器信息是通过跳板机管理和访问的，这样无法直接 ssh 连上远端服务器，这时候可以考虑这种方式，在 iterm2 的 profiles 界面，保存登录命令，通过 profile 直接登录。

- **添加脚本**

shell 脚本保存的目录可以自己设置，个人保存在了 **/usr/local/bin/iterm2login.sh** 文件下

```shell
$ sudo vim /usr/local/bin/iterm2login.sh
#!/usr/bin/expect

set timeout 30
spawn ssh -p [lindex $argv 0] [lindex $argv 1]@[lindex $argv 2]
expect {
        "(yes/no)?"
        {send "yes\n";exp_continue}
        "password:"
        {send "[lindex $argv 3]\n"}
        "Password"
        {send "[lindex $argv 3]\n"}
}
interact%
```

保存后记得赋予该脚本可执行权限（由于在个人电脑，直接赋予了 777 权限(#^.^#)）

```shell
$ chmod 777 /usr/local/bin/iterm2login.sh
```

- **保存 profile 信息**

在这里设置配置文件信息

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/iterm2/iterm2_profile.png)

左下角的 + 号，进行添加一个配置文件，接着在右边的 Command 栏，输入以下命令：

```shell
/usr/local/bin/iterm2login.sh port root ip password
```

|字段名|含义|
|--|--|
|port |开放登录的端口号，常见的是 22|
|root |登录者角色 |
|ip |远端服务器 ip |
|password|登录角色对应的密码 |

根据上述格式，可能输入如下：

```shell
/usr/local/bin/iterm2login.sh 22 root 127.0.0.1 123456
```

输入完成后，可以通过快捷键：⌘（command） + O（英文 O），调出配置信息，然后选择刚才保存的配置，iterm2 调用脚本后，会自动帮我们输入密码，进行登录。

- **配置 Shortcut Key**

在右侧，有一行 「Shortcut Keys」，点击下拉选择，可以选一个自己喜欢，而且与其它软件不冲突的快捷键。

例如我使用了 「⌘（command） + Shift + P」，这样可以快速的通过快捷键，直接就能进行登录~

## 5 rz、sz 上传下载

有时候想传点小工具到远端服务器，或者将服务器上的排查结果下载到本地进行分析，这时候就可以祭出我们的大杀器 rz、sz 脚本。

网上也有很多介绍和安装步骤，但是由于配置问题，弄了挺久，最后才成功配置，所以这次记录下自己的配置流程：

- **安装 lrzsz**

HomeBrew 是 mac 上一个很好用的包管理软件，如果没安装的话，可以使用以下命令安装：

```shell
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- **拷贝脚本文件**

定位到 /usr/local/bin 目录

下载下面地址的两个文件并放入 ：https://github.com/Vip-Augus/Vip-Augus.github.io/tree/master/assets/iterm2

- **添加权限**

```shell
$ cd /usrlocal/bin
$ chmod 777 iterm2-*
```

- **添加触发器**

定位到 Perferences -> Profiles -> Advanced -> Triggers -> Edit

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/iterm2/iterm2_triggers.png)

左下角的 + 号，添加以下两个触发器：

```shell
一、SZ 命令
Regular expression: \*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh

二、RZ 命令
Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
```

- **使用方式**

```shell
# 上传本地文件到服务器，直接输入 rz 命令，将会弹出文件选择框
$ rz
# 将服务器的文件发送到本地，输入 sz 命令，同时后面接上服务器的文件路径
$ sz /{patch}/file.out
```

通过上面的配置方式，就可以愉快上传和下载文件了~

## 6 快捷键推荐

像一开始，在一个窗口打开过个 tab 页，是通过拖拽 tab 形式实现的：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/iterm2/iterm2_tab.gif)


接着，在命令发送到所有窗口，是通过广播命令实现的，广播命令也分很多种，例如全部 window 的面板，或者是当前 window 的面板，我使用的是当前 window：

快捷键：「⌘（command） + Shift + i」

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/iterm2/iterm2_boardcast.gif)

使用广播命令，在一个窗口上输入指令，可以将它们发送到其它窗口上，实现多个服务器同时执行指令的功能。

还有更多快捷键，可以参考这篇文章：https://www.jianshu.com/p/a0249778872e

## 7 总结

在工作中，一点一点小工具的使用和积累，逐渐形成习惯，本次将 iterm2 这个工具常用的技巧记录下来，做个备忘，同时希望能帮到大家~


## 8 参考资料

1. [iTerm2 + Oh My Zsh 打造舒适终端体验 https://segmentfault.com/a/1190000014992947](https://segmentfault.com/a/1190000014992947)
2. [Linux配置免密码登录（原理 + 实践） https://blog.csdn.net/syani/article/details/52618840](https://blog.csdn.net/syani/article/details/52618840)
3. [iterm2保存ssh登录密码快速登录 https://www.jianshu.com/p/55216ca70963](https://www.jianshu.com/p/55216ca70963)
4. [Mac osx 下安装iTerm2，并使用rz sz上传下载（附homebrew配置） https://segmentfault.com/a/1190000012166969](https://segmentfault.com/a/1190000012166969)
5. [让你的iTerm更Geek! https://www.jianshu.com/p/a0249778872e](https://www.jianshu.com/p/a0249778872e)

