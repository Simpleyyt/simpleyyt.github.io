---
layout: post
categories: kafka
title: 如何正确理解kafka重平衡流程？
tagline: by cxuan
tags: 
  - kafka

---

Kafka 重平衡流程一直是 kafka 比较麻烦和难以理解的地方，此篇文章通过大量的示意图带你了解一下 kafka 重平衡的过程

<!--more-->

## Kafka 重平衡流程

我在  [真的，关于 Kafka 入门看这一篇就够了](https://mp.weixin.qq.com/s?__biz=MzU2NDg0OTgyMA==&mid=2247484768&idx=1&sn=724ebf1ecbb2e9df677242dec1ab217b&chksm=fc45f893cb327185db43ea9363928d71c54c62b1d9048f759671de3737d62e0e2d265289b354&token=1263191457&lang=zh_CN#rd)  中关于消费者描述的时候大致说了一下消费者组和重平衡之间的关系，实际上，归纳为一点就是让组内所有的消费者实例就消费哪些主题分区达成一致。

我们知道，一个消费者组中是要有一个`群组协调者(Coordinator)`的，而重平衡的流程就是由 Coordinator 的帮助下来完成的。

这里需要先声明一下重平衡发生的条件

* 消费者订阅的任何主题发生变化
* 消费者数量发生变化
* 分区数量发生变化
* 如果你订阅了一个还尚未创建的主题，那么重平衡在该主题创建时发生。如果你订阅的主题发生删除那么也会发生重平衡
* 消费者被群组协调器认为是 `DEAD` 状态，这可能是由于消费者崩溃或者长时间处于运行状态下发生的，这意味着在配置合理时间的范围内，消费者没有向群组协调器发送任何心跳，这也会导致重平衡的发生。

### 在了解重平衡之前，你需要知道这两个角色

`群组协调器（Coordinator）`：群组协调器是一个能够从消费者群组中收到所有消费者发送心跳消息的 broker。在最早期的版本中，元数据信息是保存在 ZooKeeper 中的，但是目前元数据信息存储到了 broker 中。每个消费者组都应该和群组中的群组协调器同步。当所有的决策要在应用程序节点中进行时，群组协调器可以满足 `JoinGroup` 请求并提供有关消费者组的元数据信息，例如分配和偏移量。群组协调器还有权知道所有消费者的心跳，消费者群组中还有一个角色就是领导者，注意把它和领导者副本和 kafka controller 进行区分。领导者是群组中负责决策的角色，所以如果领导者掉线了，群组协调器有权把所有消费者踢出组。因此，消费者群组的一个很重要的行为是选举领导者，并与协调器读取和写入有关分配和分区的元数据信息。

`消费者领导者`： 每个消费者群组中都有一个领导者。如果消费者停止发送心跳了，协调者会触发重平衡。

### 在了解重平衡之前，你需要知道状态机是什么

Kafka 设计了一套`消费者组状态机(State Machine)` ，来帮助协调者完成整个重平衡流程。消费者状态机主要有五种状态它们分别是 **Empty、Dead、PreparingRebalance、CompletingRebalance 和 Stable**。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/01.png)

了解了这些状态的含义之后，下面我们用几条路径来表示一下消费者状态的轮转

1. 消费者组一开始处于 `Empty` 状态，当重平衡开启后，它会被置于 `PreparingRebalance` 状态等待新消费者的加入，一旦有新的消费者加入后，消费者群组就会处于 `CompletingRebalance` 状态等待分配，只要有新的消费者加入群组或者离开，就会触发重平衡，消费者的状态处于 PreparingRebalance 状态。等待分配机制指定好后完成分配，那么它的流程图是这样的

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/02.png)



2. 在上图的基础上，当消费者群组都到达 `Stable` 状态后，一旦有新的**消费者加入/离开/心跳过期**，那么触发重平衡，消费者群组的状态重新处于 PreparingRebalance 状态。那么它的流程图是这样的。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/03.png)

3. 在上图的基础上，消费者群组处于 PreparingRebalance 状态后，很不幸，没人玩儿了，所有消费者都离开了，这时候还可能会保留有消费者消费的位移数据，一旦位移数据过期或者被刷新，那么消费者群组就处于 `Dead` 状态了。它的流程图是这样的

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/04.png)

4. 在上图的基础上，我们分析了消费者的重平衡，在 `PreparingRebalance`或者 `CompletingRebalance`  或者 `Stable` 任意一种状态下发生位移主题分区 Leader 发生变更，群组会直接处于 Dead 状态，它的所有路径如下

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/05.png)

>这里面需要注意两点：
>
>一般出现 **Required xx expired offsets in xxx milliseconds** 就表明Kafka 很可能就把该组的位移数据删除了
>
>只有 Empty 状态下的组，才会执行过期位移删除的操作。

### 重平衡流程

上面我们了解到了消费者群组状态的转化过程，下面我们真正开始介绍 `Rebalance` 的过程。重平衡过程可以从两个方面去看：消费者端和协调者端，首先我们先看一下消费者端

### 从消费者看重平衡

从消费者看重平衡有两个步骤：分别是 `消费者加入组` 和 `等待领导者分配方案`。这两个步骤后分别对应的请求是 `JoinGroup` 和 `SyncGroup`。

新的消费者加入群组时，这个消费者会向协调器发送 `JoinGroup` 请求。在该请求中，每个消费者成员都需要将自己消费的 topic 进行提交，我们上面描述群组协调器中说过，这么做的目的就是为了让协调器收集足够的元数据信息，来选取消费者组的领导者。通常情况下，第一个发送 JoinGroup 请求的消费者会自动称为领导者。**领导者的任务是收集所有成员的订阅信息，然后根据这些信息，制定具体的分区消费分配方案**。如图

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/06.png)

在所有的消费者都加入进来并把元数据信息提交给领导者后，领导者做出分配方案并发送 `SyncGroup `请求给协调者，协调者负责下发群组中的消费策略。下图描述了 SyncGroup 请求的过程

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/07.png)

当所有成员都成功接收到分配方案后，消费者组进入到 Stable 状态，即开始正常的消费工作。

### 从协调者来看重平衡

从协调者角度来看重平衡主要有下面这几种触发条件，

* 新成员加入组
* 组成员主动离开
* 组成员崩溃离开
* 组成员提交位移

我们分别来描述一下，先从新成员加入组开始

####新成员加入组

我们讨论的场景消费者集群状态处于`Stable` 等待分配的过程，这时候如果有新的成员加入组的话，重平衡的过程

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/08.png)

从这个角度来看，协调者的过程和消费者类似，只是刚刚从消费者的角度去看，现在从领导者的角度去看

#### 组成员离开

组成员离开消费者群组指的是消费者实例调用 `close()` 方法主动通知协调者它要退出。这里又会有一个新的请求出现 `LeaveGroup()请求` 。如下图所示

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/09.png)

#### 组成员崩溃

组成员崩溃是指消费者实例出现严重故障，宕机或者一段时间未响应，协调者接收不到消费者的心跳，就会被认为是`组成员崩溃`，崩溃离组是被动的，协调者通常需要等待一段时间才能感知到，这段时间一般是由消费者端参数 session.timeout.ms 控制的。如下图所示

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/kafka-rebalance/10.png)

#### 重平衡时提交位移

这个过程我们就不再用图形来表示了，大致描述一下就是 消费者发送 JoinGroup 请求后，群组中的消费者必须在指定的时间范围内提交各自的位移，然后再开启正常的 JoinGroup/SyncGroup 请求发送。
