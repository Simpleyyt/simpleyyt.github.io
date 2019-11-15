---
layout: post
categories: Redis
title: 一文看懂 Redis 的内存回收策略和 Key 过期策略
tags:
  - 子悠

---

### 01、前言
Redis 作为当下最热门的 Key-Value 存储系统，在大大小小的系统中都扮演着重要的角色，不管是 session 存储还是热点数据的缓存，亦或是其他场景，我们都会使用到 Redis。在生产环境我们偶尔会遇到 Redis 服务器内存不够的情况，那对于这种情况 Redis 的内存是如何回收处理的呢？另外对于带有过期时间的 Key Redis 又是如何处理的呢？

 <!--more-->

### 02、Redis 内存设置

我们都知道如果我们要设置 Redis 的最大内存大小只需要在配置文件`redis.conf` 中配置一行 `maxmemory xxx` 即可，或者我们通过 `config set` 命令在运行时动态配置 Redis 的内存大小。

![image-20191113214004027](http://www.justdojava.com/assets/images/2019/java/image_ziyou/01.png)

### 03、Redis 内存过期策略

#### 3.1、过期策略的配置

那么当 Redis 内存不够的时候，我们要知道 Redis 是根据什么策略来淘汰数据的，在配置文件中我们使用 `maxmemory-policy` 来配置策略，如下图

![image-20191113214158260](http://www.justdojava.com/assets/images/2019/java/image_ziyou/02.png)

我们可以看到策略的值由如下几种：

2. volatile-lru: 在所有带有过期时间的 key 中使用 LRU 算法淘汰数据；
3. alkeys-lru: 在所有的 key 中使用最近最少被使用 LRU 算法淘汰数据，保证新加入的数据正常；
3. volatile-random: 在所有带有过期时间的 key 中随机淘汰数据；
4. allkeys-random: 在所有的 key 中随机淘汰数据；
5. volatile-ttl: 在所有带有过期时间的 key 中，淘汰最早会过期的数据；
6. noeviction: 不回收，当达到最大内存的时候，在增加新数据的时候会返回 error，不会清除旧数据，这是 Redis 的默认策略；

> volatile-lru, volatile-random, volatile-ttl 这几种情况在 Redis 中没有带有过期 Key 的时候跟 noeviction 策略是一样的。淘汰策略是可以动态调整的，调整的时候是不需要重启的，原文是这样说的，我们可以根据自己 Redis 的模式来动态调整策略。
> ”To pick the right eviction policy is important depending on the access pattern of your application, however you can reconfigure the policy at runtime while the application is running, and monitor the number of cache misses and hits using the Redis INFO output in order to tune your setup.“

#### 3.2、策略的执行过程

1. 客户端运行命令，添加数据申请内存；
2. Redis 会检查内存的使用情况，如果已经超过的最大限制，就是根据配置的内存淘汰策略去淘汰相应的 key，从而保证新数据正常添加；
3. 继续执行命令。

#### 3.3、近似的 LRU 算法

Redis 中的 LRU 算法不是精确的 LRU 算法，而是一种经过采样的LRU，我们可以通过在配置文件中设置 `maxmemory-samples 5` 来设置采样的大小，默认值为 5，我们可以自行调整。官方提供的采用对比如下，我们可以看到当采用数设置为 10 的时候已经很接近真实的 LRU 算法了。

![image-20191113231007725](http://www.justdojava.com/assets/images/2019/java/image_ziyou/03.png)

在 Redis 3.x 以上的版本的中做过优化，目前的近似 LRU 算法以及提升了很大的效率，Redis 之所以不采样实际的 LRU 算法，是因为会耗费很多的内存，原文是这样说的

> The reason why Redis does not use a true LRU implementation is because it costs more memory.

### 04、Key 的过期策略

#### 4.1、设置带有过期时间的 key

前面介绍了 Redis 的内存回收策略，下面我们看看 Key 的过期策略，提到 Key 的过期策略，我们说的当然是带有 expire 时间的 key，如下

![image-20191113233350118](http://www.justdojava.com/assets/images/2019/java/image_ziyou/04.png)

通过 `redis> set name ziyouu ex 100` 命令我们在 Redis 中设置一个 key 为 name，值为 ziyouu 的数据，从上面的截图中我们可以看到右下角有个 TTL，并且每次刷新都是在减少的，说明我们设置带有过期时间的 key 成功了。

#### 4.2、Redis 如何清除带有过期时间的 key

对于如何清除过期的 key 通常我们很自然的可以想到就是我们可以给每个 key 加一个定时器，这样当时间到达过期时间的时候就自动删除 key，这种策略我们叫**定时策略**。这种方式对内存是友好的，因为可以及时清理过期的可以，但是由于每个带有过期时间的 key 都需要一个定时器，所以这种方式对 CPU 是不友好的，会占用很多的 CPU，另外这种方式是一种主动的行为。

有主动也有被动，我们可以不用定时器，而是在每次访问一个 key 的时候再去判断这个 key 是否到达过期时间了，过期了就删除掉。这种方式我们叫做**惰性策略**，这种方式对 CPU 是友好的，但是对应的也有一个问题，就是如果这些过期的 key 我们再也不会访问，那么永远就不会删除了。

Redis 服务器在真正实现的时候上面的两种方式都会用到，这样就可以得到一种折中的方式。另外在**定时策略**中，从官网我们可以看到如下说明

> Specifically this is what Redis does 10 times per second:
>
> 1. Test 20 random keys from the set of keys with an associated expire.
> 2. Delete all the keys found expired.
> 3. If more than 25% of keys were expired, start again from step 1.

意思是说 Redis 会在有过期时间的 Key 集合中随机 20 个出来，删掉已经过期的 Key，如果比例超过 25%，再重新执行操作。每秒中会执行 10 个这样的操作。

### 05、总结

今天给大家介绍了一下 Redis 的内存回收和 Key 过期策略的处理，Redis 作为必备的开发组件，我们必须好好掌握，希望今天的文章能帮助大家更好的掌握 Redis 的核心。另外欢迎大家到我们的知识星球中与我们一起进步。

