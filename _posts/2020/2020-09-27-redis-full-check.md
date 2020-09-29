---
layout: post
categories: Java
title: Redis 实例对比工具之 Redis-full-check
tagline: by 子悠
tags: 
  - 子悠
---
Hello 大家好，我是鸭血粉丝，前面一篇文章给大家介绍了 `SpringBoot` 项目是如何从单机切换接入集群的，没看过的小伙伴可以去看一下[SpringBoot 项目接入 Redis 集群](https://mp.weixin.qq.com/s/QWrejQxS09G9ms90cnoZHQ) 。这篇文章给大家介绍一个 Redis 工具 redis-full-check，主要是用来校验迁移数据过后的准确性，下面我们来看一下。

<!--more-->

# 安装

Redis-full-check 是阿里开源的一个工具，GitHub 地址 https://github.com/alibaba/RedisFullCheck，安装前我们需要找一台 Linux 机器，并且 GLIBC的版本需要高于 2.14，不然使用的时候会提示 `/lib64/libc.so.6: version GLIBC_2.14 not found` 。下载我们有两种方式，第一种是在本地直接下载，然后上传到服务器上面；另一个是直接在服务器上面执行`wget https://github.com/alibaba/RedisFullCheck/releases/download/release-v1.4.8-20200212/redis-full-check-1.4.8.tar.gz`进行下载。下载完成过后解压`tar xzvf redis-full-check-1.4.8.tar.gz`。具体的过程我们如下进行：

1. 检查当前服务器的 GLIBC 版本，执行命令`strings /lib64/libc.so.6 |grep GLIBC_ `，如下图，如果出现高于 2.14 的即可，如果没有可以考虑换一台服务器或者自己更新，但是更新有风险请谨慎，具体的更新方法自行百度;

   ![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/redis-full-check/1.png)

2. 下载压缩包，执行:`wget https://github.com/alibaba/RedisFullCheck/releases/download/release-v1.4.8-20200212/redis-full-check-1.4.8.tar.gz` 下载完成后解压。阿粉这里已经下过了， 就不重复下载了，解压后进入目录，输入`./redis-full-check  -v` 如果能正常看到版本号就说明下载安装成功了。

   ![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/redis-full-check/2.png)

   ![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/redis-full-check/3.png)



# 使用

在使用这个工具之前，你需要的是两台不同的 Redis 实例，阿粉这边因为是从单机切换到集群，所以已经有了。下面就有单机和集群给大家演示。我们执行如下命令：`./redis-full-check -s "172.20.xxx.xxx:6379" -p "sourcePassword" --sourcedbfilterlist=0 -t "172.20.xxx.xxx:6379;172.20.yyy.yyy:6379" -a "targetPassword" --targetdbtype=1`

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/redis-full-check/4.png)

说明：

1. -s: 表示源 Redis 实例
2. p：源 Redis 密码
3. --sourcedbfilterlist：匹配指定的 db 库，单集 Redis 是可以设置特定 db 库的，集群环境不行，根据自己的情况决定是否采用；
4. -t：目标 Redis，阿粉这边是集群所以会有多个节点，每个节点用分号隔开，另外注意文档上说这里必须填写所有的 master 节点或者所有的 slave 节点，不能混合填写。阿粉这里填的都是 master 节点是成功，但是全部 slave 好像没成功，大家可以自己试试。
5. -a：表示目标 Redis 的密码
6. --targetdbtype=1：目标 Redis 环境的类型，0：db(standalone单节点、主从)，1: cluster（集群版），2: 阿里云

详细的参数如下：

```shell
 -s, --source=SOURCE               源redis库地址（ip:port），如果是集群版，那么需要以分号（;）分割不同的db，只需要配置主或者从的其中之一。例如：10.1.1.1:1000;10.2.2.2:2000;10.3.3.3:3000。
  -p, --sourcepassword=Password     源redis库密码
      --sourceauthtype=AUTH-TYPE    源库管理权限，开源reids下此参数无用。
      --sourcedbtype=               源库的类别，0：db(standalone单节点、主从)，1: cluster（集群版），2: 阿里云
      --sourcedbfilterlist=         源库需要抓取的逻辑db白名单，以分号（;）分割，例如：0;5;15表示db0,db5和db15都会被抓取
  -t, --target=TARGET               目的redis库地址（ip:port）
  -a, --targetpassword=Password     目的redis库密码
      --targetauthtype=AUTH-TYPE    目的库管理权限，开源reids下此参数无用。
      --targetdbtype=               参考sourcedbtype
      --targetdbfilterlist=         参考sourcedbfilterlist
  -d, --db=Sqlite3-DB-FILE          对于差异的key存储的sqlite3 db的位置，默认result.db
      --comparetimes=COUNT          比较轮数
  -m, --comparemode=                比较模式，1表示全量比较，2表示只对比value的长度，3只对比key是否存在，4全量比较的情况下，忽略大key的比较
      --id=                         用于打metric
      --jobid=                      用于打metric
      --taskid=                     用于打metric
  -q, --qps=                        qps限速阈值
      --interval=Second             每轮之间的时间间隔
      --batchcount=COUNT            批量聚合的数量
      --parallel=COUNT              比较的并发协程数，默认5
      --log=FILE                    log文件
      --result=FILE                 不一致结果记录到result文件中，格式：'db    diff-type    key    field'
      --metric=FILE                 metric文件
      --bigkeythreshold=COUNT       大key拆分的阈值，用于comparemode=4
  -f, --filterlist=FILTER           需要比较的key列表，以分号（;）分割。例如："abc*|efg|m*"表示对比'abc', 'abc1', 'efg', 'm', 'mxyz'，不对比'efgh', 'p'。
  -v, --version
```

# 查看结果

执行完上面的命令过后在当前目录下会生成三个文件，分别是result.db.1，result.db.2，result.db.3。我们可以通过 sqlite3 工具进行查询，如下所示：

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/redis-full-check/5.png)

通过`sqlite3 result.db.3` 命令进入终端，然后从 key 表中查询我们需要的数据。sqlite3 工具是一个类似 MySQL 的数据库，大家可以自己研究下如何使用，后面有机会阿粉再跟大家分享。

从上面的图中可以发现，这个结果看起来很难受，阿粉再教大家几招，让看起来爽一点！进入终端后我们依次输入下面图中命令

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2020/redis-full-check/6.png)

1. `.header on` 打开表头，id 只是序号，key 表示源 Redis 中的 key，type 表示类型，db 表示 key 所在的源 Redis 的 db 库，source_len，和 target_len 分别表示在源 Redis 和目标 Redis 的中 value 的长度。我们可以通过长度来快速查看不同的数据。
2. `.mode column` 设置输出模式
3. `.widht int int...` 设置每列显示的长度，更美观
4. `.quit` 退出终端

通过这个输出结果我们可以明显的看出哪些数据是不一致的，从而对比两个 Redis 实例的数据，需要注意的是 **Redis-full-check 对比的是源实例是否是目标实例的子集**！

## 总结

今天阿粉给大家介绍了一个 Redis 实例数据对比的工具，能真正在生产上使用的一个阿里开源的很优秀的工具，希望对大家有帮助！具体的更多使用细节，大家可以自己研究研究，一个好的工具值得好好深入研究。

# 写在最后

最后邀请你加入我们的知识星球，这里有 1800+ 优秀的人与你一起进步，如果你是小白那你是稳赚了，很多业内经验和干活分享给你；如果你是大佬，那可以进来我们一起交流分享你的经验，说不定日后我们还可以有合作，给你的人生多一个可能。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)
