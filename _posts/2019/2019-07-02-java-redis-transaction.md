---
layout: post
categories: Redis
title: 聊聊 Redis 的事务
tagline: by 子悠
tags: 
  - 子悠

---

### 背景

提到事务想必大家一定不会陌生，工作面试中多多少少都会了解到，这篇文章主要带大家再简单回忆一下事务的基本知识，然后重点介绍下 Redis 的事务，关于 Redis 的事务有何不同我们继续往下看就知道了。
<!--more-->

#### 什么事务
说到事务，首先我们需要知道什么是事务。首先事务是作为单个逻辑工作单元执行的一系列操作，这些操作作为一个整体一起向系统提交，要么都执行，要么都不执行。事务是一个不可分割的逻辑单元。


#### 事务的四大特性

1. A（原子性）事务的各步操作是不可分的，保证一系列的操作要么都完成，要么都不完成；

2.  C（一致性）事务完成，数据必须处于一致的状态；

3. I（隔离性）对数据进行修改的所有并发事务彼此之间是相互隔离，这表明事务必须是独立的，不应以任何方式依赖或影响其他事务；

4. D（持久性）表示事务对数据处理结束后，对数据更改必须持久化，不管是事务成功还是回滚。事务日志都能够保持事务的永久性。

以上是常规的事务以及事务的特性。下面我们来看一下什么是Redis的事务，以及Redis事务有什么特殊性质。



### Redis 事务

关于Redis的性质官方文档如下

> [MULTI](https://redis.io/commands/multi), [EXEC](https://redis.io/commands/exec), [DISCARD](https://redis.io/commands/discard) and [WATCH](https://redis.io/commands/watch) are the foundation of transactions in Redis. They allow the execution of a group of commands in a single step, with two important guarantees:
>
> 1. All the commands in a transaction are serialized and executed sequentially.
>
> 2. Either all of the commands or none are processed, so a Redis transaction is also atomic. 

意思是说 `MULTI`,`EXEC`,`DISCARS`,`WATCH` 这四个命令是 Redis 的基石，它们允许在保证下面两点的条件下单步执行一组命令。

1. 事务中所有的命令都是都是顺序的并且按顺序执行；
2. 要么处理所有命令，要么不处理任何命令，因此Redis事务也是原子的



#### Redis 事务的使用

我们先来看下Redis事务的使用方式，关键命令如下`Multi`,`Exec`,`Discard` ,`WATCH`

使用Multi命令表示开启一个事务
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/redis1.jpg)

开启一个事务过后中间输入的所有命令都不会被立即执行，而是被加入到队列中缓存起来，当收到Exec命令的时候Redis服务会按入队顺序依次执行命令。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/redis2.jpg)

从上面的两个截图中我们可以看出以下几点

1. `multe` 命令可以开启一个事务，开启成功会 redis 会返回 OK 状态；

2. 在`multi`命令后输入的命令不会被立即执行，而是被加入的队列中，并且加入成功 redis 会返回 QUEUED，表示加入队列成功，如果这里的命令输入错误了，或者命令参数不对，Redis 会返回 ERR 如下图，并且此次事务无法继续执行了。这里需要注意的是在 Redis 2.6.5 版本后是会取消事务的执行，但是在 2.6.5 之前 Redis 是会执行所有成功加入队列的命令。详细信息可以看官方文档。

   > If there is an error while queueing a command, most clients will abort the transaction discarding it.

   ![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/redis3.jpg)

3. 输入`exec`命令后会依次执行加入到队列中的命令



### Redis 事务中的错误

上面提到了在 Redis 的事务中命令在加入队列的时候如果出错，那么此次事务是会被取消执行的。这种错误在执行`exec` 命令前 Redis 服务就可以探测到，但是在 Redis 事务中还有一种错误，那就是所有命令都加入队列成功，了，但是在执行`exec`命令的过程中出现了错误，这种错误 Redis 是无法提前探测到的，那么这种情况下 Redis 的事务是怎么处理的呢？

可能到这里你会说：这还用说，回滚啊，傻子都知道。

但是重点来了，真的是这样的吗？下面我们看一个例子，如图：

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/redis4.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/redis5.jpg)

上面我们测试的过程是我们先通过命令`get a`获取`a` 的值为 5，然后开启一个事务，在事务中执行两个动作，第一个是自增`a` 的值，另一个是通过命令`hset a b 3`来设置`a` 中 `b` 的值，我们可以看到这里`a`  的类型是字符串，但是第二个命令也成功的加入到了队列，Redis 并没有报错。但是最后在执行`exec` 命令的时候，第一条命令执行成功了，看到返回结果是 6，第二条命令执行失败了，提示的错误信息表示类型不对。

**然后我们在通过`get a`命令发现`a`的值已经被改变了，不再是之前的 5 了，说明虽然事务失败了但是命令执行的结果并没有回滚！**

看到这里的小伙伴估计都惊呆了，估计会说我在扯犊子，但是事实就是如此，那么到底是为什么会这样呢？我们看下官方给的解释。

### Redis 为什么不支持事务回滚

> - Redis commands can fail only if called with a wrong syntax (and the problem is not detectable during the command queueing), or against keys holding the wrong data type: this means that in practical terms a failing command is the result of a programming errors, and a kind of error that is very likely to be detected during development, and not in production.
> - Redis is internally simplified and faster because it does not need the ability to roll back.

上面总共提到两点解释，第一点意思是说在开发环境中就能避免掉语法错误或者类型不匹配的情况，在生产上是不会出现的；第二点是说 Redis 的内部是简单的快速的，所以不需要支持回滚的能力。

从这里我们可以看出 Redis 的开发者很自信，所以我猜测他们用的洗发露肯定是海飞丝的🤣。



### 小结

简单总结一下，今天刚开始跟大家回顾了一个事务的定义以及事务的四大特性，然后引入 Redis 的事务，并且通过例子给大家演示了 Redis 事务的使用以及特性。另外如果大家觉得感兴趣的话可以自己去读读官方文档，加深印象。

对于今天的文章你有什么想要说的或者分享的吗？欢迎到 Java 极客技术的知识星球中跟我们一起交流。这里有着一批热爱技术的人在一起成长，欢迎你的加入。

