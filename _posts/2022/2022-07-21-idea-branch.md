---
layout: post
categories: Java
title: IntelliJ IDEA 的这个 BUG 存在三年了！
tagline: by 子悠
tags: 
  - 子悠
---

`Java` 程序员使用最多的 `IDE` 那一定是 `IntelliJ IDEA`（如果你还在用 `Eclipse` 或者 `MyEclipse` 我建议你换换），阿粉作为一名`Java` 程序员日常工作中几乎天天使用，不知道你有没有遇到过这种场景，那就是如果远程仓库中多了新的分支，每次通过右下角的分支管理 `fetch` 了远程分支过后，在搜索框中找不到获取到的分支，而是要关闭窗口，重新再打开搜索一下才能进行 `checkout` 操作，而且移除了分支也是一样，不会动态更新。

<!--more-->

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h4evxdg1kkg21f80t4b29.gif)

虽然说问题不大，但是当分支多了操作的次数多了以后，还是很烦心的，一开始遇到的时候总觉得是自己使用的不对，或者软件的版本有问题，但是最近升级到最新版本过后，发现这个问题还是存在，无奈的 `Google` 了一下，不搜不知道一搜索才发现，原来这个 `Bug` 已经存在了三年了，而且还一直处于 `Open` 的状态。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h4evr804dtj220k0u0jye.jpg)

这个 `BUG` 的描述跟我们的问题是完全一样的，说的是 fetch 结束过后，分支列表并没有更新。而且通过右边的面板我们可以看到，这还是一个未关闭的高优先级的 `BUG`。`BUG` 的完整链接地址我放在文末了，感兴趣的小伙伴可以去看看。而且在这个 `BUG` 下面还关联着很多类似的 `BUG`，看来这个问题是一个大家一直在关注的问题，只是官方一直没有解决。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h4evu6c3roj21ap0u0n28.jpg)

上面很多关联的 BUG 都被引入到这个 BUG 中了，从关联的 BUG 数量和标题来看，基本上都是在吐槽分支没刷新的问题。其实看到这里，让阿粉想到了一句话，那就是：**又不是不能用！**哈哈哈，虽然说这个问题不大，分支不多的情况下也不会很麻烦，但是还是感觉会有点美中不足。

**毕竟程序员不喜欢自己有 `BUG`，但是喜欢给别人找 `BUG`**。另外从 `BUG` 的评论中我们可以看到最近的一个评论是在 19 号，说明这个问题是一直有人在关注的，就是得不到官方的关注。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h4evyuxob6j21cy0u0wjq.jpg)

其实换个角度想想也是可以理解的，这个问题本身遇到的场景和频率都不会很多，而且真正遇到的时候还可以通过再次操作点开窗口来解决，这么看来这个 `BUG` 其实并不重要又不紧急，哪个开发手头上没挂着几个无伤大雅的优化需求呢。



https://youtrack.jetbrains.com/issue/IDEA-229545/List-of-branches-is-not-updated-at-once-after-fetch-is-done
