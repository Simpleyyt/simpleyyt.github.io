---
title: 'Ngnix 是如何解决 epoll 惊群的'
layout: post
tags:
  - network
  - linux
category: Network
---
Ngnix 的 master 进程在创建 socket，`bind`和`listen`之后，`fork`出多个 worker，worker 会 将这个 socket 加入 epoll 中，用`epoll_wait`来处理事件，当有一个新的连接来的时候，所有 worker 都会被唤醒，这就是所谓的 epoll 惊群。

<!--more-->

历史上还存在过 accept 惊群，但现在的内核已经解决了这个问题，内核只会唤醒一个进程。

Ngnix 目前有几种方法解决惊群问题。

## accept_mutex

如果开启了 `accept_mutex` 锁，每个 worker 都会先去抢自旋锁，只有抢占成功了，才把 socket 加入到 epoll 中，accept 请求，然后释放锁。`accept_mutex`效率低下，特别是在长连接的时候，因此不建议使用，默认是关闭的。

## EPOLLEXCLUSIVE

`EPOLLEXCLUSIVE`是4.5+内核新添加的一个 epoll 的标识，Ngnix  在 1.11.3 之后添加了`NGX_EXCLUSIVE_EVENT`。

`EPOLLEXCLUSIVE`标识会保证一个事件发生时候只有一个线程会被唤醒，以避免多侦听下的“惊群”问题。不过任一时候只能有一个工作线程调用 accept，限制了真正并行的吞吐量。

## SO_REUSEPORT

`SO_REUSEPORT` 是惊群最好的解决方法，Ngnix 在 1.9.1 中加入了这个选项，每个 worker 都有自己的 socket，这些 socket 都`bind`同一个端口。当新请求到来时，内核根据四元组信息进行负载均衡，非常高效。


---

*参考：*

 * <http://xiaorui.cc/2015/12/02/使用socket-so_reuseport提高服务端性能/>
 * <http://pureage.info/2015/12/22/thundering-herd.html>
 * <http://www.cnblogs.com/sxhlinux/p/6254396.html>
 * <http://m.blog.csdn.net/russell_tao/article/details/7204260>
 * <https://www.zhihu.com/question/51618274?utm_medium=social&utm_source=wechat_session&from=groupmessage&isappinstalled=1>
 * <http://www.sohu.com/a/148006569_470018>