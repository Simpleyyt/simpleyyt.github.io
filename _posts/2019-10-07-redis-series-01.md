---
layout: post
category: Redis
title: 【Redis 系列 01】Redis 基本概述与安装
tagline: by 子悠
tags:
  - redis
published: true
---

**人生有涯，学海无涯**

Redis 作为当下主流的 NoSQL 数据库，已经成为企业级开发不可或缺的一个重要组件了，现在大大小小的项目都会用到它，并且最新的版本已经达到 5.x 了。从这篇文章开始，我们会用一个系列的文章详细的给大家介绍一下 Redis 底层实现和使用场景，希望能帮助大家更好的使用。

<!--more-->
### 01、概述

本篇是 Redis 系列的第一篇文章，我们主要对 Redis 做一下概述，以及详细的安装教程和简单使用，后面的文章会一步一步带大家走进 Redis 的细节部分。

#### 1.1 Redis 简介

学任何一个技术，我们首先看的必定是官网，Redis 的官网是这样介绍的

> Redis is an open source (BSD licensed), in-memory **data structure store**, used as a database, cache and message broker. It supports data structures such as [strings](https://redis.io/topics/data-types-intro#strings), [hashes](https://redis.io/topics/data-types-intro#hashes), [lists](https://redis.io/topics/data-types-intro#lists), [sets](https://redis.io/topics/data-types-intro#sets), [sorted sets](https://redis.io/topics/data-types-intro#sorted-sets) with range queries, [bitmaps](https://redis.io/topics/data-types-intro#bitmaps), [hyperloglogs](https://redis.io/topics/data-types-intro#hyperloglogs), [geospatial indexes](https://redis.io/commands/geoadd) with radius queries and [streams](https://redis.io/topics/streams-intro.md).

大致的意思是说：Redis 是一个开放源码(基于 BSD 协议)的内存存储数据结构，被用作于数据库，缓存。它支持数据结构有字符串，哈希，列表，集合，排序集合，位图，超日志，GEO。

#### 1.2 特性

1. 完全基于内存，大多数请求都是内存操作，非常快速；
2. 数据结构简单，操作简单；
3. 采用单线程，避免了不必要的上下文切换和竞争条件，不存在多进程或者多线程的切换；
4. 使用多路复用 I/O 模型，非阻塞 IO；
5. 多客户端支持，高可用架构。

### 02、安装

#### 方式一：采用 Docker 来安装 Redis

1. 搜索镜像 `docker search redis`：使用该命令可以搜索出所有的 `redis` 镜像列表![image-20191003102625851](http://justdojava.com/assets/images/2019/java/image_ziyou/100301.png)

2.  如果没有特殊版本需求，可以使用：`docker pull redis` 命令直接安装最新版本 `Redis`
3. 下载过后使用 `docker images | grep redis` 命令查看已经获取到的镜像![image-20191003104525395](http://justdojava.com/assets/images/2019/java/image_ziyou/100302.png)

4. 启动 `Redis` 服务：`docker run --name redis -d -p 6379:6379 redis --requirepass "123321"`
   1. `docker run`：表示创建并运行一个容器；
   2. `--name redis`：表示创建一个名字为 `redis` 的容器；
   3. `-d`：表示后台运行；
   4. `-p 6379:6379`：表示将宿主机的 6379 端口映射到容器的 6379 端口上；
   5. `redis`：表示依赖的镜像名称；
   6. `--requirepass "123321"` ：表示设置密码
5. 查看运行的`Redis` 实例![image-20191003105336215](http://justdojava.com/assets/images/2019/java/image_ziyou/100303.png)

#### 方式二：下载编译安装

1. 进入指定目录`/usr/local/` 执行如下命令

   1. ```shell
      wget http://download.redis.io/releases/redis-5.0.5.tar.gz
      tar -xzvf redis-5.0.5.tar.gz
      cd redis-5.0.5
      make install
      ```

      ![image-20191011001935962](http://justdojava.com/assets/images/2019/java/image_ziyou/redis-series-01.png)

2. 编译安装完成后在 src 目录下会生成 `redis-server, redis-cli `相关命令，同时在`/usr/local/bin`也会存在，如果没有可以拷贝进去，正常是会有的。
3. 修改 redis.conf 配置文件
   1. 配置后台启动：`daemonize yes`；
   2. 调整端口，如果需要的话：`port 6379`；
   3. 设置 redis 日志路径：`logfile /var/log/redis.log`；
   4. 设置 redis 密码：`requirepass "123321"`; **密码一定要设置，不然会被攻击**
4. 使用配置文件启动 redis 服务端：`redis-server redis.conf`
5. 客户端登录并测试

![image-20191011002718840](http://justdojava.com/assets/images/2019/java/image_ziyou/redis-series02.png)

至此，两种方式的安装都介绍完了，大家可以根据自己的情况采用一种方式安装学习。

### 03、使用场景

Redis 作为 NoSQL 数据库，主要的使用场景如下：

1. 缓存：用 Redis 作为缓存框架，将热点数据，高频的访问数据存放到 Redis 中，加快访问速度，降低对底层数据库的依赖，从而提高并发和性能。这个场景是最常用的一个场景，而且 Redis 本身提高过期时间和相应的淘汰策略，这个后面我们会讲到。
2. 计数统计：Redis 本身支持加一和减一命令，单线程情况下可以帮我们做简单的计数功能，可以累积计算 pv 值，也很实用。
3. 排序和集合操作：Redis 的 list 和 zset 数据结构可以用于需要实现排序如排行榜，最新评论，或者获取共同好友等场景。
4. 消息队列：Redis 提供的 发布订阅（`PUB/SUB`）和 阻塞队列的功能，能够满足基本的消息队列的使用。

### 04、总结

这篇文章是 Redis 系列文章的第一篇，简单的给大家介绍了 Redis 的一些概述和基本特性，以及两种安装方式和一些常用的使用场景，下篇文章会开始介绍一些基本的命令使用以及逐渐的深入分析。大家在使用 Redis 的时候有什么坑或者心得可以分享的吗？欢迎到我们《Java 极客技术》知识星球来大家一起讨论，一起进步。

![子悠-知识星球](http://justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)

