---
layout: post
categories: Java
title: Git Reset VS Git Revert
tagline: by 子悠
tags: 
  - 子悠
---
Hello，大家好，我是阿粉，之前给大家介绍过 Git 的几个超级实用的命令，没看过的朋友可以去看一下

[那些你应该知道的，但是你一定不知道的 Git 骚操作](https://mp.weixin.qq.com/s/IPW49LY1OchrAJC-VGpEcQ) 今天再给大家介绍一个不常用，但是关键时刻很好用的命令`git revert`。

<!--more-->

# 背景

日常工作中经常都是很多同事一起迭代开发，而且经常会有很多需求的开发在不同的代码分支上，如果出现不小心将某个未完成的功能提交了，并且已经 push 到分支上去了，那我们该怎么办呢？阿粉最近就遇到了这样的问题，之前提交的一个功能代码，虽然是一个完整的功能，但是由于种种原因这个功能被取消了，相关的代码也需要被撤销不能提交到生产上面去，但是在这个 commit 之后也有许多其他功能代码的提交。其他功能还是要正常上线的，不能被影响的。

这个时候很多小伙伴就会说：可以把对应需要撤销的功能代码重新修改掉不就可以了吗？这种方案当时是可以的，特别是如果我们改动的地方不多的情况下，直接将代码修改回来即可，方便又简单快速。但是如果对应要修改的文件很多，而且每个文件修改的地方又很多那就很麻烦了，如果对着提前的修改一行一行的修改就是个非常浪费时间的事情了。

# Git Reset/Revert

遇到这种情况我们能想到的肯定是网上一定有相关的解决方案，并且 Git 一定提供了相应的命令可以帮到我们。通过查询 Git 手册我们发现 Git 提供了两个相关命令让我们回滚版本，分别是 reset 和 revert。那两个之间有什么区别呢？下面阿粉通过示例给大家演示一下这两个命令的使用和区别。

## Git Reset

我们先分几次创建几个文件，然后依次`commit` 和 `push` ，形成多次提交，如下图所示，我们创建了四个文件，每个文件单独 `commit` 和 `push`，然后通过`git log` 命令我们可以查看的整个提交的日志信息，其中就包含了四次的文件提交记录。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/1025/1.png)

现在我们突然发现 test3.txt 文件提交的有问题，我们需要回滚到 test2.txt 的版本，我们可以通过执行命令`git reset --hard 15e32cd0cc909ef6791e4417f5572b5e7886f977`  `--hard` 表示强制回到指定的版本，后面紧跟的是目标的版本号。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/1025/2.png)

再通过`git log` 命令我们可以看到当前的版本已经变掉了，变成我们指定的版本了。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/1025/3.png)

我们再通过`git push -f origin branch` 命令将重置的版本推送到远程服务器上，由于在推送到服务器前我们本地的代码已经落后服务器的代码了，所以我们需要增加`-f` 参数表示强制推送到远程服务器。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/1025/4.png)

细心的朋友可能会发现，我们执行到这里，确实回退到需要的版本了，但是有问题的如果只是 test3.txt 文件，而 test4.txt 文件是没有问题的，我们是可以正常提交的，如果按照这种方式去操作，我们就会把 test4.txt 版本的修改给丢失了。当然我们可以重新再提交写一遍，但是如果这个版本的内容很多，我们是改不过来的。所以这种方式一般我们是不会使用的，只有确保后续所有的修改都不需要的时候我们才可以使用这种方式。

## Git Revert

下面再看下 `git revert ` 命令的使用方式，我们分两次创建两个文件，分别`commit` 和 `push` 到远程，然后通过 `git log`，我们可以看到下面内容，现在最新的版本已经是 test6.txt 了。同样的，这个时候我们发现 test5.txt 的版本有问题，但是 test6.txt 的版本是正确的，我们只想撤销掉 test5.txt 版本的提交。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/1025/5.png)

这个时候我们执行命令`git revert -n 50896fa7d9ba16b63a0fc539bb6620411e5dee4c` 将 test5.txt 版本给撤销，并执行`git commit -m xxx` 进行提交然后在`push` 到远程。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/1025/6.png)

再通过`git log` 我们可以看到，产生了一次新的提交将我们 test5.txt 版本的内容撤销掉了，并且 test6.txt 版本的提交还依旧保留在。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/1025/7.png)

通过上面的演示，我们可以发现`git reset` 和 `git revert` 两个命令虽然都可以进行版本回退，但是在使用的时候还是有很多的差异的。在我们确认了在需要回退的版本之后的提交都可以不需要的时候，我们可以直接使用`git reset` 命令，但是当我们只是需要撤销某个版本的时候，我们就可以使用`git revert` 。

# 总结

今天阿粉通过示例给大家介绍了 `git reset` 和`git revert` 两个命令的使用和区别，在日常工作的难免会遇到需要使用的时候，如果不需要当然是最好，但是万一什么时候需要用了，对大家有帮助也是极好的，如果觉得有用，欢迎点赞转发，让更多的小伙伴看到。

# 写在最后

最后邀请你加入我们的知识星球，这里有 1800+ 优秀的人与你一起进步，如果你是小白那你是稳赚了，很多业内经验和干活分享给你；如果你是大佬，那可以进来我们一起交流分享你的经验，说不定日后我们还可以有合作，给你的人生多一个可能。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)

