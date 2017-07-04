---
title: 'Google Maglev 简介'
layout: post
tags:
  - network
category: Network
---
Google Maglev 是一个牛逼的负载均衡器，之所以牛逼，是因为它不用部署专门的物理设备，不像 LVS 一样工作在内核，它是运行在通用 Linux 服务器上的大型分布式软件系统。

<!--more-->

# Google Maglev 工作流程

![Google Maglev 工作流程](https://nos.netease.com/knowledge/3db49483-ce09-4384-9eec-4963db7c0950)

每个 Google 服务都有一个或者多个 VIP，一个 VIP 和物理 IP 的区别在于 VIP 没有绑给某个特定的网卡。

**VIP 宣告：**

Maglev 关联每个 VIP 到具体的 Endpoint，然后通过 BGP 将 VIP 宣告给上游路由器，然后路由器再把 VIP 宣告给 Google 的骨干网，这样使得 VIP 能被访问到。

**当用户访问 www.google.com 时：**

 * 浏览器先发送一个 DNS 请求， DNS 服务返回 VIP。 然后浏览器尝试与该 VIP 建立连接。
 * 当路由器接收到 VIP 数据包，通过 ECMP 将数据包路由到 Maglev 集群中的某台机器上。
 * 当 Maglev 的机器接收到数据包， 从关联到该 VIP 的 Endpoint 中选择一个， 然后用 GRE 封包发送，外层的 IP 即 Endpoint 的物理 IP。
 * 当 Endpoint 处理完数据包进行响应时，源地址用 VIP 填充，目的地址为用户 IP。 使用直接服务返回(Direct Server Return， DSR) ，将响应直接发送给路由器， 这样 Maglev 无需处理响应包。

# Google Maglev 结构

![Maglev 结构](https://nos.netease.com/knowledge/987eed70-e65e-4bac-a9e5-81ade6deb8a5)

Maglev 由控制器（Controller）和 转发器（Forwarder）组成：

 * **控制器向路由器宣告 VIP。**控制器周期性地检查转发器的健康状态，来宣告或者撤回 VIP。确保路由器只转发包到健康的 Magvel。
 * **转发器转发 VIP 流量到 Endpoint。**每个 VIP 都有一个后端池（BP），BP 可能包含 Endpoint 的 IP，也有可能包含其它 BP。每个 BP 会对 Endpoint 进行健康检查，保证数据包转发到健康的 Endpoint。

# 转发器的设计和实现

## 转发器结构

![转发器结构](https://nos.netease.com/knowledge/68165b8d-e769-499c-8b75-ecca8681600b)

转发器直接从网卡接收数据包，通过 GRE/IP 封包，再将它们发回网卡。 Linux 内核不参与这个过程。

 * Steering 处理从 NIC（网卡）接收来的数据包，通过五元组（IP地址，源端口，目的IP地址，目的端口和传输层协议）哈希，然后交给不同的接收队列。
 * 包重写线程从接收队列取包，然后进行后端选择，用 GRE 封包后，发送到传输队列。
 * Muxing 轮询从所有的传输队列取包，再发送给网卡。

## 快速包处理

![转发器包处理](https://nos.netease.com/knowledge/9688207c-2875-4f4c-81d1-81e8681a1fbb)

Maglev 在包处理时绕开了 Linux 内核，因为内核开销非常严重。

如图，转发器和网卡共用数据包池（Packet Pool），转发器中的 Steering 和 Muxing 都通过指针的环形队列，指向该池。

**对于 Seering：**

 * 当网卡接收到数据包时，放在 Recieved 所指位置，并向前移动指针。
 * 分发包到接收队列时，向前移动Processed 指针。
 * 预留未使用数据包，放入队列中并向前移动 Reserved 指针。

**对于 Muxing：**

 * 网卡将 Sent 所指的数据包发出去，并向前移动指针。
 * 将被重写的数据包放入队列中，并向前移动 Ready 指针。
 * 同时将已被发送的包归回给数据包池，并向前移动 Recycled 指针。

整个过程都没有包拷贝。

## 后端选择

Maglev首先检查本地的连接跟踪表，看看该数据包是否属于任何一个已有的连接，如果连接已经建立，则直接将数据包发到该连接对应的服务器上去。如果连接没有建立，此时就需要一致性哈希函数选择一个后端服务器了，并添加到连接跟踪表。

Maglev 一致性哈希的基本思想就是：

 * 有一个共享的`Entry`表，可以通过`Entry[Hash % M]`选择对应后端，`M`为`Entry`表大小。
 * 每个后端对所有`Entry`表位置有自己的优先级排序，存在`permutation`表里。
 * 所有的后端通过优先级顺序轮流填充`Entry` 中的空白位置，直至填满。每次都填充自己优先级最高的空位置。

例如，假设`M = 6`，且
```
B0: permutation[] = { 3, 0, 4, 1, 5, 2, 6 }
B1: permutation[] = { 0, 2, 4, 6, 1, 3, 5 }
B2: permutation[] = { 3, 4, 5, 6, 0, 1, 2 }
```

`permutation`优先级是从大到小，那么最后填充后的`Entry`表为：

```
Entry[] = { B1, B0, B1, B0, B2, B2, B0 }
```

说明：

 > * B0 的优先级最高的位置是`Entry[3]`，其为空，则`Entry[3] = B0`。
 > * B1 是`Entry[0]`，则`Entry[0]=B1`。
 > * B2 是`Entry[3]`，但是已被占，下一个是`Entry[4]`，为空，则`Entry[4]=B2`。
 > * 以此类推，直至填满。

# 操作经验

## VIP 匹配

![VIP匹配](https://nos.netease.com/knowledge/cbe69316-d0fd-469a-9692-95948f0cfb51)

有时候，我们需要利用 Maglev 封包将流量重定向到其他的集群中的相同服务，这就有点麻烦了，因为集群间是独立的，我们不知道其它集群相同服务的 VIP。

VIP 可以通过最长前缀匹配，来决定集群，利用最长后缀匹配决定那个后端池。

如图中的例子，当请求 173.194.71.1 时，通过最长前缀（173.194.71.0/24）选择 C2 集群，通过最长后缀（0.0.0.1/8）决定是 Service 1 后端池。

假定 Maglev 需要将流量转发到 C3（173.194.72.0/24），只需要用相同的后缀构造出 VIP （173.194.72.1）进行转发，即可转发到 Maglev 的 Service 1 后端池。

## 分片处理

分片时，非首个分片只包含三元组（目的IP地址，目的端口和传输层协议），这便无法正确决定如何转发，因为转发根据的是五元组。

**解决方法是：**

每个 Maglev 配置了一个特殊的后端池，包含所有的 Maglev 机器。

一旦接收到分片， Maglev 用三元组哈希选择特定的 Maglev 作为后端进行转发，将它们重定向给相同的 Maglev。

Maglev 为未分片数据包和第二跳的首分片使用相同的后端决策算法，以保证非分片、首个分片和非首个分片选择同一个后端。

Magelv 维护了一个固定大小的分片表，记录了首分片的转发决策。 当 Maglev 收到一个第二跳非首分片， 会从分片表中查找，若匹配则立即转发； 否则，会缓存到分片表中，直到首分片收到或者老化。