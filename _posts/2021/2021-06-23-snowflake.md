---
layout: post
categories: Java
title: 分布式环境下如何保证 ID 的唯一性
tagline: by 子悠
tags: 
  - 子悠
---

### 前言

首先说下我们为什么需要分布式 ID，以及分布式 ID 是用来解决什么问题的。当我们的项目还处于单体架构的时候，我们使用数据库的自增 ID 就可以解决很多数据标识问题。但是随着我们的业务发展我们的架构就会逐渐演变成分布式架构，那么这个时候再使用数据的自增 ID 就不行了，因为一个业务的数据可能会放在好几个数据库里面，此时我们就需要一个分布式 ID 用来标识一条数据，因此我们需要一个分布式 ID 的生成服务。那么分布式 ID 的服务有什么要求和挑战呢？

<!--more-->

### 要求

1. 全局唯一：既然是用来标识数据唯一的，那么一个分布式 ID 肯定要是全局唯一的，在同一业务下的每个服务下面都是一致的，不会变的，这是一个基本的要求；
2. 全局递增：递增这个也很好理解，我们要保证生成的 ID 是依次递增的，因为很多时候 ID 是给人看的，如果说不具备递增性，就缺乏了很多的可读性；
3. 信息安全：分布式 ID 的安全性也很重要，因为我们提到生成的 ID 是递增的，这就有可能会给竞争对手知道我们的 ID 的生成频率，这种在电商等场景会有很大的问题，但是这个往往跟全局递增有点冲突；
4. 高可用性：分布式 ID 的生成服务必须是高可用，毕竟一旦不能生成 ID，后续的所有服务都无法继续使用；

### 常见的分布式 ID 实现

在当下的互联网当中，根据业务场景以及需求的不同，对于分布式 ID 的实现有如下几种实现方式：

1. UUID；
2. Redis；
3. 变形的数据库自增 ID；
4. 推特雪花算法
5. 美团的 Leaf——雪花算法的变形；

#### UUID

写 Java 的朋友对 UUID 肯定不陌生，`7dbb9f04-d15e-4c88-b74b-72a35e0d7580` 这是一个标准的 UUID，虽然都说 UUID 是全球唯一，具备我们前面提到的要求中的第一点，但是很显然不具备全局递增，这种分布式 ID 可读性很差，如果说只是用来记录日志或者不需要人去理解的场景是可以用，但是不适合我们这里说的业务数据的唯一标识。而且这种无序的 UUID 如果作为主键会很严重影响性能。

#### Redis

Redis 有个 incr 的命令，这个命令是能保证原子递增的，在某种程度上也是可以生成全局 ID，不过使用 Redis 有两个问题：

1. 不美观，虽然说我们需要的是一个全局 ID，但是 incr 命令是从 1 开始的整型，所以会导致全局 ID 的长度不一致，虽然说也可以用来标识唯一业务数据，但是某些场景也缺少可读性，因为不携带日期信息；
2. 依赖 Redis 的高可用，因为 Redis 是基于内存的，为了保证 ID 的不丢失所以需要对 Redis 进行持久化，但是关于 Redis 的两种持久化的方式各有优缺点，详细的可以参考公众号之前的文章 [面试官：请说下 Redis 是如何保证在宕机后数据不丢失的](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247492071&idx=1&sn=7ac7858547e73ef117cac48ce1821551&chksm=c2868e26f5f10730392e01d273703f5f904e0d751d7273f450765be85b44f883bd56151ea956&token=889940930&lang=zh_CN#rd)；

#### 数据库自增 ID

前面我们提到单个数据库在分布式环境下已经没办法使用自增 ID 了，因为每个 MySQL 的实例自增 ID 都是从 1 开始，并且步长都按照 1依次递增，这种情况下我们很容易想到是不是可以考虑给每个数据库设置不同的步长。如果我们设置了不同的步长，这样就可以保证每个数据库实例都可以生成 ID，并且不会重复。虽然简单的系统可以这样用，但是也有几个问题：

1. 依赖数据库 DB，在分布式环境下，如果过多的依赖数据库是有风险的，无法支持高并发的情况，特别是对于一些电商交易的场景，每秒几十万的 QPS，数据库是扛不住的；
2. 不同数据库实例的数据不能直接关联上，需要额外的存储，才能把数据串起来，增加业务复杂度；

#### 推特的雪花算法—— snowflake

`snowflake` 算法是推特开源的分布式 ID 生成算法，这个算法提供了一个标准的思路，很多公司都参考这个算法做了自己的实现，比较有名的是美团的 `Leaf`。这里我们就着重看下雪花算法是怎么实现的。

> 感兴趣的可以去参考文章 https://tech.meituan.com/2017/04/21/mt-leaf.html 看下美团的 leaf 的实现原理。

雪花算法的思想是化整为零，将分布式 ID 的生成分散到每个机房和机器上，采用一个 64 位 long 类型的的结构来表示一个 ID，64 的结构如下所示，第一位符号位 0，然后是 41 位的时间戳，接下来的 10 位是机房加机器，最后的 12 位是序列号。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2021/0627/0.png)

上面这个结构是雪花算法的基本结构，不同公司根据自身的业务会进行相应的调整，有的可以采用 32 位或者其他位数，而且时间戳的位数也可以根据实际情况进行调整，10 位的 workerID 有机房的公司可以用机房加机器组成，没有机房的公司可以直接用机器来组成，序列位也可以根据情况适当调整。

我们可以简单算一下，41 位的时间位是2 ^ 41 / (365 * 24 * 3600 * 1000) = 69 年，每个机器每毫秒可以生成 2 ^ 12 = 4096 个 ID。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2021/0627/1.png)

那是不是说我们这个代码只能运行 69 年呢？其实不是的，这里服务在启动的时候会设置一个初始值，这里的时间戳是用机器的时间减去初始值的差值。那 SnowFlake 算法有什么优缺点呢？

1. 因为有时间戳，所以满足自增的要求，同时也具备一定的可读性；
2. 化整为零每个服务在各自的机器上可以直接生成唯一 ID，只需要配置好机房和机器编号记刻；
3. 长度可以根据业务自行调整；
4. 缺点是依赖机器的时钟，如果说机器的时钟有问题，会导致生成的 ID 可能会重复，这个需要控制；

结合上面的原理，我们可以通过 Java 代码来具体实现，代码如下：

```java
public class SnowFlakeUtil {

    //初始时间戳
    private final static long START_TIMESTAMP = 1624796691000L;
    //数据中心占用的位数
    private final static long DATA_CENTER_BIT = 5;
    //机器标识占用的位数
    private final static long MACHINE_BIT = 5;
    //序列号占用的位数
    private final static long SEQUENCE_BIT = 12;


    /**
     * 每一部分的最大值
     */
    private final static long MAX_SEQUENCE = ~(-1L << SEQUENCE_BIT);
    private final static long MAX_MACHINE_NUM = ~(-1L << MACHINE_BIT);
    private final static long MAX_DATA_CENTER_NUM = ~(-1L << DATA_CENTER_BIT);

    /**
     * 每一部分向左的位移
     */
    private final static long MACHINE_LEFT = SEQUENCE_BIT;
    private final static long DATA_CENTER_LEFT = SEQUENCE_BIT + MACHINE_BIT;
    private final static long TIMESTAMP_LEFT = DATA_CENTER_LEFT + DATA_CENTER_BIT;

    private final long idc;
    private final long serverId;
    private long sequence = 0L;
    private long lastTimeStamp = -1L;

    private long getNextMill() {
        long mill = System.currentTimeMillis();
        while (mill <= lastTimeStamp) {
            mill = System.currentTimeMillis();
        }
        return mill;
    }

    /**
     * 根据指定的数据中心ID和机器标志ID生成指定的序列号
     *
     * @param idc      数据中心ID
     * @param serverId 机器标志ID
     */
    public SnowFlakeUtil(long idc, long serverId) {
        if (idc > MAX_DATA_CENTER_NUM || idc < 0) {
            throw new IllegalArgumentException("IDC 数据中心编号非法！");
        }
        if (serverId > MAX_MACHINE_NUM || serverId < 0) {
            throw new IllegalArgumentException("serverId 机器编号非法！");
        }
        this.idc = idc;
        this.serverId = serverId;
    }

    /**
     * 生成下一个 ID
     *
     * @return
     */
    public synchronized long genNextId() {
        long currTimeStamp = System.currentTimeMillis();
        if (currTimeStamp < lastTimeStamp) {
            throw new RuntimeException("Clock moved backwards.  Refusing to generate id");
        }
        if (currTimeStamp == lastTimeStamp) {
            //相同毫秒内，序列号自增
            sequence = (sequence + 1) & MAX_SEQUENCE;
            //同一毫秒的序列数已经达到最大
            if (sequence == 0L) {
                currTimeStamp = getNextMill();
            }
        } else {
            //不同毫秒内，序列号置为0
            sequence = 0L;
        }
        lastTimeStamp = currTimeStamp;
        return (currTimeStamp - START_TIMESTAMP) << TIMESTAMP_LEFT | idc << DATA_CENTER_LEFT | serverId << MACHINE_LEFT | sequence;
    }

    public static void main(String[] args) {
        SnowFlakeUtil snowFlake = new SnowFlakeUtil(4, 3);
        for (int i = 0; i < 100; i++) {
            System.out.println(snowFlake.genNextId());
        }
    }
}
```

### 参考

1. 知乎·一口气说出9种分布式ID生成方式，面试官有点懵了
2. Leaf——美团点评分布式ID生成系统