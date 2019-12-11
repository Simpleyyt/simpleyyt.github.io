---
layout: post
categories: Redis
title: 一文带你了解 Redis 的慢日志相关底层原理
tags:
  - 子悠

---

### 01、前言

相信很多小伙伴在使用 Redis 的时候都知道 Redis 有相关慢日志的查询功能，并且多多少少都看过。那 Redis 底层到底是如果创建慢日志以及慢日志的结构是什么样子的呢？这篇文章就带大家认识一下。我们先看一张慢日志的截图

![1](http://www.justdojava.com/assets/images/2019/java/image_ziyou/1201.png)使用`slowlog get 2`命令查看最近的两条慢日志信息，如上图，我们可以看到每条日志中包含的信息有六个部分组成，从上到下编号为 0-5，依次代表的意思是

- 0：日志的唯一编号 ID
- 1：命令执行的当前时间戳
- 2：命令执行的耗时时长，单位微妙
- 3：具体的执行命令和参数
- 4：客户端的 ip 和端口（4.0 版本以上才支持）
- 5：客户端名称（4.0 版本以上支持）

如上图所示，第一条慢日志的 ID 是 41，命令执行的时间戳是 1575729996，并且执行了 16129 微妙，具体执行的命令就是`slowlog get`，ip 和端口是`27.38.56.88:8223`，客户端的名称没有设置。

### 02、慢日志命令设置

#### 查看命令

上面我们已经大概的知道的一条慢日志的格式，自然的我们可以想到的问题是一个命令执行多长时间，我们就可以认为是慢查询，以及慢日志最多能保存多少条。

我们可以通过`config get slowlog-log-slower-than` 命令来查看 Redis 的时长设置，以及通过`config get slowlog-max-len` 来查看最大慢日志条数。如下图。

![2](http://www.justdojava.com/assets/images/2019/java/image_ziyou/1202.png)

#### 设置命令

上面我们使用`config get` 命令查看了时长设置和条数设置，相反的我们可以用`config set`来设置相关参数，如下图，我们先查看一下配置，然后再通过`config set slowlog-log-slower-than 1000` 命令和 `config set slowlog-max-len 64` 命令来设置具体的值：

![3](http://www.justdojava.com/assets/images/2019/java/image_ziyou/1203.png)

通过上面的操作我们可以看到相关的配置已经更改生效了。

1. slowlog-log-slower-than：这个参数的意思是任何命令执行超过这个时间就会被记录为慢日志，单位是微秒。
2. slowlog-max-len：这个参数表示记录的慢日志的最大条数，设置了这个值过后，新的日志加进来，ID 最小的日志就会被删除。

为了验证上面的第二点，我这边将`slowlog-log-slower-than`设置为 10 微秒，`slowlog-max-len` 设置为 5 条来进行试验，首先第一次使用`slowlog get`命令查询的时候 5 条慢日志的编号是从 83-87，

![4](http://www.justdojava.com/assets/images/2019/java/image_ziyou/1204.png)

再次使用`slowlog get`命令查询的编号结果是84-88，说明 ID 为 83 的那一条已经被删除了。

![5](http://www.justdojava.com/assets/images/2019/java/image_ziyou/1205.png)

### 03、慢日志的存储原理

#### 存储结构

```c
struct redisServer {
	long long slowlog_entry_id;//下一条慢查询日志的 ID
	list *slowlog;//保存了所有慢查询日志的链表
	long long slowlog_log_slower_than;//服务器配置 slowlog-log-slower-than 选项的值
	unsigned long slowlog_max_len;//服务器配置的 slowlog-max-len 的值
}
```

- 在 Redis 的服务器状态中保存了慢日志的相关属性`slowlog_entry_id` 属性的初始值是 0 每创建一条慢日志的时候就会增加 1。
- slowlog 链表里面存储了所有的慢日志，链表是由`slowlogEntry`结构组成的，每个`slowlogEntry`代表一条慢日志。
- `slowlog_log_slower_than` 和 `slowlog_max_len` 是前面服务器配置的相关参数。

#### slowlog 链表

```
typedef struct slowlogEntry {
	long long id;//唯一标识符
	time_t time; //命令执行的时间，格式为 unix 时间戳
	long long duration;//命令消耗的时间，以微妙为单位
	robj **argv;//命令与命令参数
	int argc; //命令与命令参数个数
} slowlogEntry;
```

![image-20191211220858341](http://www.justdojava.com/assets/images/2019/java/image_ziyou/1206.png)

`slowlogEntry` 实体的相关字段含义如下：

- id: 标识慢日志的唯一 ID
- time: 命令执行的时间戳
- duration: 命令执行是耗时，单位为微妙
- agrv: 命令和参数数组
- argc: 命令和参数的个数

如上图就表示了在执行命令`set number 520` 发生了慢日志，命令执行耗时 10 微妙。

### 04、操作慢日志

知道了慢日志的存储结构，我们就需要考虑在执行命令的时候，如何根据条件去创建慢日志。

首先我们需要在命令的执行前后记录时间戳，然后相减计算出命令的执行耗时，然后根据 Redis 服务器配置`slowlog-log-slower-than` 进行对比，决定是否记录慢日志，另外在记录慢日志的时候需要根据`slowlog_max_len` 值判断是否要删除最久的日志信息。伪代码如下：

```c
//1. 记录命令执行前的时间戳
long before = now();
//2. 执行命令
execute(argv, argc);
//3. 记录命令执行后的时间戳
long after = now();
//4. 调用创建慢日志函数
slowlogPushEntryIfNeed(argv, argc, after - before);
```

`slowlogPushEntryIfNeed` 函数主要用来判断是否插入数据，以及是否删除旧数据。

```c
void slowlogPushEntryIfNeed(robj **argv, int argc, long long duration) {
	//1. 判断是否开启慢日志
	if (server.slowlog_log_slower_than < 0) return;
	//2. 如果超时，则插入慢日志
  if (duraton > server.slowlog_log_slower_than){
    //插入
  }
  while (listLength(server.slowlog) > server.slowlog_max_len) {
    //删除
  }
}
```

我们先判断服务器是否配置了超时参数，如果超时参数小于 0 则直接返回，否则再比较命令执行时间是否超时，如果超时则插入慢日志；最后在比较慢日志的条数是否达到上限，如果达到则进行删除。

### 05、总结

Redis 在日常工作中经常会使用，其中还有很多细节需要我们慢慢研究和学习，公号已经发过好多关于 Redis 相关的文章了，后续还会有其他内容的相关文章，欢迎关注。最后欢迎大家到我们《Java 极客技术》知识星球中来跟我们一起学习，一起进步。

