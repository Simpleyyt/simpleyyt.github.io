---
layout: post
categories: Redis
title: Redis 小白入门以及基础搭建
tags:
  - cxuan

---



![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/01.png)

<!--more-->

## Redis 简介

### 什么是 Redis

`Redis` 的全称是 Remote Dictionary Server，它是一款 `开源的` 高性能的 `NoSQL` 数据库，它可以用作 `数据库`、`缓存` 和 `消息队列`。

### 什么是 NoSQL

NoSQL 最常见的解释是 `non-relational`，非关系型数据库，还有一种说法是 `Not Only SQL`，不仅仅是 SQL，NoSQL 仅仅是一个概念，泛指非关系型的数据库，区别于关系数据库，它们不保证关系数据的 ACID 特性。ACID 即

* A (Atomicity) 原子性
* C (Consistency) 一致性
* I (Isolation) 独立性
* D (Durability) 持久性

Redis 通过提供多种键值对的数据类型来适应不同场景下的存储需求。

#### NoSQL 的代表

作为 NoSQL 的代表主要有

* MongoDB
* Redis
* Memcached

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/02.png)

#### NoSQL 的优点

Redis 相较于关系型数据库模型，它还是具有很多优点的

* 易扩展

NoSQL 数据库种类繁多，但是一个共同的特点就是去掉关系数据库的关系型特性，数据之间无关系，这样就非常容易扩展。

* 大数据量，高性能

NoSQL 数据库都具有非常高的读写性能，尤其在大数据量下，同样表现很优秀。

* 灵活的数据模型

NoSQL 无需事先建立字段，这省去了关系型数据库一旦建立字段，可扩展性非常差的不利局面。NoSQL 随时可以存储自定义的数据格式。

* 高可用

NoSQL 在不太影响性能的情况，就可以方便地实现高可用的架构。比如 Cassandra、HBase 模型，通过复制模型也能实现高可用。

### Redis 数据类型

Redis 支持的数据类型主要有五种，它们分别是

* `字符串 - strings`

string 字符串 是 Redis 中最简单的数据类型，它是与 `Memcached` 一样的类型，一个 key 对应一个 value，这种 key/value 对应的方式称为键值对。字符串对很多例子都有用，例如缓存 HTML 片段和网页。

* `集合 - set`

set 是集合，和我们数学中的集合概念相似，对集合的操作有添加删除元素，有对多个集合求交并差等操作。操作中 key 理解为集合的名字。

* `散列 - hash`

hash 是一个键值 (key=>value) 对集合；是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

* `列表 - list`

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。list类型经常会被用于消息队列的服务，以完成多程序之间的消息交换。

* `有序集合 - zset`

Redis zset 和 set 一样也是 string 类型元素的集合，且不允许重复的成员。当你需要一个有序的并且不重复的集合列表时，那么可以选择 sorted set 数据结构。

## Redis 安装

Redis 支持在 Linux、OS X 、BSD 等 POSIX 系统中安装，也支持在 Windows 中安装，在 Windows 、Linux 下的安装请参考 https://www.runoob.com/redis/redis-install.html

我自己的电脑是 mac 系统，mac 系统的安装过程是 https://www.jianshu.com/p/bb7c19c5fc47

如果你用的是 `Homebrew`，它的安装过程是  https://www.jianshu.com/p/e1e5717049e8

### Redis 启动

安装完成 Redis 后，我们需要启动 Redis，不过我们在启动 Redis 之前，需要先了解一下 Redis 的可执行文件，如果你是下载的 `tar.gz` 安装包，可以使用 make install 进行编译，编译完成后会在 `/usr/local/bin` 目录生成下面这几个可执行文件

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/03.png)

下面我们来一起认识一下这些可执行文件的作用

* `redis-benchmark` 是 Redis 性能测试工具，可以使用 redis-benchmark 进行基准测试，比如`redis-benchmark -q -n 100000` ，这个工具使用起来非常方便，同时你可以使用自己的基准测试工具
* `redis-check-rdb`  RDB 文件检查工具，rdb 是 Redis 的一种持久化方式
* `redis-check-aof`  AOF 文件检查工具，aof 也是 Redis 的一种持久化方式，关于这两种持久化方式我们后面会说
* `redis-sentinel`  redis-sentinel 就是一个独立运行的进程，用于监控多个 master-slave 集群，
  自动发现 master 宕机，进行自动切换 slave > master。
* `redis-cli`，redis-cli 和 后面的 redis-server 都是常用的而且比较重要的命令，是你登录 redis 的指令
* `redis-server`，redis-server 是 redis 的后台服务器

Redis 启动分为两种，一种是直接运行 `redis-server`，一种是通过 redis 脚本启动，前者一般用于开发环境，脚本启动一般用于生产环境，这里我们直接使用直接启动的方式。

直接运行 redis-server 即可启动 redis

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/04.png)

也可以通过指定 Redis 的端口来运行 Redis

```shell
redis-server --port 6380
```

这里我们就不贴出图了。

### Redis 停止

考虑到 Redis 有可能正在将内存中的数据同步到硬盘中，强制执行 Redis 的停止命令可能导致数据丢失。正确停止 Redis 的方式应该是向 Redis 发送 `SHUTDOWN` 命令，如下

```shell
redis-cli SHUTDOWN
```

这里你可能还不清楚 `redis-cli` 是什么指令，我们下面会说。

当 Redis 收到 SHUTDOWN 指令后，会先断开所有客户端连接，然后根据配置执行持久化，最后完成退出。

也可以使用 `kill Redis 端口` 来正常结束 Redis ，效果与 SHUTDOWN 命令一样。

### Redis 命令行客户端

下面我们来说一下 `redis-cli` 指令，redis-cli 和 redis-server 都是 Redis 中非常重要的工具，你一定要知道，redis-cli 的全称是 `Redis Command Line Interface`，是 Redis 自带的命令行工具，它主要有下面这几种用法

#### 发送命令

通过 redis-cli 向 Redis 发送命令的方式有两种，一种是将命令作为 redis-cli 的参数执行，比如上面的 redis-cli SHUTDOWN 命令，redis-cli 在执行时会按照默认的主机和端口号来进行连接，默认的主机和端口号分别是 `127.0.0.1` 和 `6379` 连接 Redis

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/05.png)

也可以通过 -h 和 -p 参数自定义地址和端口号

```shell
redis-cli -h 127.0.0.1 -p 6379
```

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/06.png)

Redis 提供了 `PING` 命令来测试客户端与 Redis 的连接是否正常，如果正常会回复到 `PONG` 响应，如下

```shell
redis-cli PING
```

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/07.png)

#### 命令返回值

在大多数情况下，执行一条命令后我们会希望获得命令的返回值，在 Redis 中，Redis 的命令返回值主要有 5 种类型，对于每种类型 redis-cli 的展现结果都不同，下面分别进行说明

**状态回复**

`状态回复（status reply）`是一种最简单的回复，比如向 Redis 发送 SET 命令设置某个键的值时，Redis 会回复 OK 表示设置成功。我们上面说到的 PING 和 PONG 也是一种状态回复。状态回复会直接显示状态信息。

**错误回复**

`错误回复(error reply)`指的是，当出现命令不存在或者命令有错误时，会出现错误回复，并在后面跟上错误信息。比如执行一个不存在的指令

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/08.png)

**整数回复**

Redis 虽然没有整数类型，但是却提供了一些用于整数操作的指令，如递增键值的 `INCR` 命令会以整数形式返回递增后的键值。除此之外，一些其他的命令也会返回整数。`整数回复(integer reply)` 以 integer 为开头然后在后面跟上整型数据

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/09.png)

**字符串回复**

`字符串回复(bulk reply)` 是最常见的一种回复类型，当请求一个字符串类型的键的键值或一个其他类型键中的某个元素时就会得到一个字符串回复。字符串回复以双引号包裹

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/10.png)

还有一种情况时这个请求的键值不存在时会返回 `nli`，表示返回的是一个空结果。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/11.png)

**多行字符串回复**

`多行字符串回复(multi-bulk reply)` 同样很常见，如当请求一个非字符串类型的键的元素列表时就会收到多行字符串回复

### Redis 配置

redis 可以支持自定义配置，比如是否开启持久化、配置日志级别等。由于可配置的选项比较多，通过启动来配置这些选项并不合理，因此可以通过程序员手动配置。启用配置文件的方法是在启动时将配置文件的路径作为启动参数传递给 redis-server，如下

```shell
redis-server /路径/redis.conf
```

通过参数传递同名的配置选项会覆配置文件中相应的参数，如下

```shell
redis-server /路径/redis.conf --loglevel warning
```

Redis 提供了一个配置文件的模版 `redis.conf`，Redis 也支持动态修改配置，可以使用 `CONFIG SET` 命令在不重新 Redis 的情况下动态修改 Redis

```shell
CONFIG SET loglevel warning
```

### Redis 多数据库切换

Redis 是一个字典结构的存储服务器，实际上一个 Redis 实例提供了多个用来存储数据的字典，客户端可以指定将数据存放在哪个字典中。

每个数据库对外都是以一个从 0 开始的递增数字命名，Redis 默认支持 16 个数据库，可以通过配置 `databases` 来修改这一数字。客户端与 Redis 建立连接后会默认选择 0 号数据库，不过可以使用 `select` 命令切换数据库。

```shell
select 1
```

这条指令就会选择 1 号数据库

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/redis/12.png)

这些数据库与我们平常理解的 MySQL 数据库不同。首先 Redis 不支持自定义数据库的名字，每个数据库都以编号命名，开发者必须自己记录哪些数据库存储了哪些数据。另外，Redis 也不支持为每个不同的数据库设置不同的访问密码。最重要的是多个数据库之间并没有完全隔离，比如 `FLUSHALL` 命令会清楚所有数据库中的数据。



文章参考：

《Redis 入门指南》

https://baike.baidu.com/item/NoSQL/8828247?fr=aladdin

https://www.php.cn/redis/421950.html

《Redis 官网》

http://www.redis.cn/topics/benchmarks.html