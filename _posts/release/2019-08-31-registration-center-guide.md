---
layout: post
category: Java
title: 分布式系统面试系列04-服务注册中心如何进行选型的？服务发现慢遇到了么？怎么解决？
tagline: by 乔二爷
tags:
    - Java
---
上家公司我们的技术栈是基于 SpringCloud ，注册中心默认的也是使用Eureka，但是出来面试的时候被问到了关于注册中心的选型是怎么来做的。今天这篇整好梳理一下服务注册中心如何来进行选型，还有服务发现慢的问题都是怎么来解决的，希望对大家有所帮助。

<!--more-->

服务注册中心，当前用得比较多的就是 Eureka 跟 Zookeeper 了。

Eureka 是 SpringCloud 自带的组件，而 Zookeeper 则是 Dubbo 默认选择的。
我们以前在做服务这块其实是基于 Spring Cloud 技术栈来做的，没有选择Dubbo。所以，Eureka 也就作为了我们的服务注册中心首选。

在选择服务服务注册中心之前，我们一般会选择是基于 Spring Cloud 或者 Dubbo 来作为我们的微服务框架。选择好以后那么服务注册中心，一般情况下Dubbo作为服务框架的，一般注册中心会选择zk， Spring Cloud作为服务框架的，一般服务注册中心会选择Eureka。

除了这些对比以外还有其他的一些对比，是我们不得不去了解的。

### 1、高可用

#### eureka 集群模式

Eureka 集群模式，是peer-to-peer的，集群里面的每个机器的地位是相等的，不存在什么主从什么的说法。每个服务可以向任意一个Eureka 实例进行 **服务注册和服务发现**,集群里面任意一个Eureka 实例接收到写请求以后，会自动同步给其他所有的 Eureka 实例。

看图一：
![](http://www.justdojava.com/assets/images/2019/java/image_qry/2019-08-31-registration-center-guide/1.png)

#### zookeeper 
Zookeeper 服务注册与发现的原理则是 Leader + Follower两种角色。只有Leader 可以负责写，也就是服务注册，他可以把数据同步给所有的 Follower,读的时候，也就是服务发现，是 Leader 和 Follower 都可以进行读取。

看图二：

![](http://www.justdojava.com/assets/images/2019/java/image_qry/2019-08-31-registration-center-guide/2.png)



### 2、在一致性保证方面

**在分布式系统中，有一个非常出明的定理就是 CAP 定律了。这三者 Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性）在一个分布式系统中是不可兼得的。**

要么保证 CP 要么保证 AP。之前写过一篇分布式事物里面解释过为什么在这两种搭配里面选择，不过多阐述了。

Zookeeper 是有一个 Leader 节点会接收数据，然后同步写到其他的 Follower 节点去。一旦 leader 挂掉，就要重新进行选举新的 leader，在新选举 leader 未完成前，集群是不可用的，当 leader 选择好了，集群可以恢复继续写了，保证了数据的一致性，**这个过程为了保证 C，牺牲了 A**。 


而 Eureka 是 peer 模式，可能数据还没有同步完成，结果自己就宕了。此时还是可以继续从别的机器上拉取注册信息，只是不是新的而以，这个过程服务可以向其他没有宕机的节点进行注册。

Eureka 保证了服务的可用性，当节点重新启动起来以后，数据还是会同步过来，一致性方面就是 最终一致性，**所以Eureka 保证了 A 牺牲了 C**。

### 3、服务的时效性方面

Zookeeper 的时效性更好一些，注册或者是服务挂了，一般秒级别就能感知到。


而 Eureka，默认的配置可能会有从几十秒到分钟级别。上线一个新的服务，到其他人发现他可能要一分钟，还可能不止,因为Spring Cloud是通过 ribbon 去获取每个服务缓存的 Eureka 的注册表进行负载均衡的。ribbon 本身还有自己的缓存机制，还有自己的时间间隔。

当服务发生故障了，默认是隔 60秒采取检查，发现这个服务上一次是在 60s 之前，Eureka 默认是超过 90秒才会任务它已经死掉了。这时候差不多已经过去 2分钟了。

再则，Eureka 里面默认是 30 秒才会把 ReadWrite 缓存的数据同步到 ReadOnly 缓存中，其他服务默认也是 30秒 才会去重新拉取一次 ReadOnly 缓存到本地服务表中。
这一趟下来算算时间还是很长的。

### 4、容量

zk 不太适合大规模的服务实例，因为服务上下线的时候需要瞬间推送数据通知到其他所有的实例，所以服务实例到几千个的时候，可能会导致网络带宽被打满。

Eureka 也是同样比较难支撑大规模的服务实例，因为它的每个节点都需要保存全量的实例数据，几千实例服务可能会扛不住。


### 5、Eureka 服务发现慢的问题及调优

zk是因为一上线服务跟下线服务就会立马发生数据同步，同时有服务进行节点的监听，很快的时间内就可以感知到服务上线下线。所以基本上不需要怎么去优化。

我们在前面的SpringCloud 底层原理文章中讲过 Eureka 的运行机制。没看过的同学，先去看看吧。

再结合到上面的Eureka时效性问题，我们可以发现对于 Eureka 来说，由于他的缓存机制会导致服务会存在发现慢的问题。

我们可以针对下面的几个点来优化：

1. 优化服务发现时间；
2. 优化定时从ReadWrite缓存到 ReadOnly 缓存的时间；
3. 优化定时心跳检查的时间；

![](http://www.justdojava.com/assets/images/2019/java/image_qry/2019-08-31-registration-center-guide/3.png)

针对项目的参数调整，给一个大概的配置：

```
而 eureka，必须优化参数
## Eureka  两个缓存之间的同更新时间 配置成 3s
eureka.server.responseCacheUpdateIntervalMs = 3000
## Eureka client 去 Eureka 中拉取服务注册表的时间 配置 3 秒
eureka.client.registryFetchIntervalSeconds = 30000
## 心跳上报时间 默认 30s ，调整为 3s 
eureka.client.leaseRenewalIntervalInSeconds = 30
## 线程多少时间检查一次心跳 默认 60s ,可以配置为 6000,6秒中
eureka.server.evictionIntervalTimerInMs = 60000
## 服务发现的时效性，默认90 秒钟才下线。可以调整为 9s
eureka.instance.leaseExpirationDurationInSeconds = 90
```

这个参数的配置可能会由于SpringCloud 的版本不同，写法也不同。

### 6、代码示例

最后贴一个代码示例 [https://github.com/heyxyw/spring-cloud](https://github.com/heyxyw/spring-cloud)，还是之前给的示例代码，在那之上已经加上了这些参数配置，大家拉下来跑一跑，体验体验，光看不练不顶用。

里面更新了执行东西 动态网关、灰度发布方案、超时重试优化配置，后续会出针对这些的讲解，还有分布式事物方案的落地、幂等相关的实现。

![](http://www.justdojava.com/assets/images/2019/java/image_qry/2019-08-31-registration-center-guide/4.png)

### 7、学习资料

以上内容是学习《中华石杉老师-21天互联网Java进阶面试训练营（分布式篇）》课程时自己整理的笔记，希望对大家有所帮助。
