---
layout: post
title: Kafka 第一篇 - 基本概述和快速搭建
tagline: by cxuan
categories: Kafka
tags: 
  - Java,Kafka

---

kafka 现在在企业应用和互联网项目中的应用越来越多了，本篇文章就从 kafka 的基础开始带你一展 kafka 的宏图

<!--more-->

## 什么是 Kafka

`Kafka` 是一个分布式流式平台，它有三个关键能力

1. 订阅发布记录流，它类似于企业中的`消息队列` 或 `企业消息传递系统`
2. 以容错的方式存储记录流
3. 实时记录流

### Kafka 的应用

1. 作为消息系统
2. 作为存储系统
3. 作为流处理器

Kafka 可以建立流数据管道，可靠性的在系统或应用之间获取数据。

建立流式应用传输和响应数据。

### Kafka 作为消息系统

Kafka 作为消息系统，它有三个基本组件

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/01.png)

* Producer : 发布消息的客户端
* Broker：一个从生产者接受并存储消息的客户端
* Consumer : 消费者从 Broker 中读取消息

在大型系统中，会需要和很多子系统做交互，也需要消息传递，在诸如此类系统中，你会找到源系统（消息发送方）和 目的系统（消息接收方）。为了在这样的消息系统中传输数据，你需要有合适的数据管道

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/02.png)

这种数据的交互看起来就很混乱，如果我们使用消息传递系统，那么系统就会变得更加简单和整洁

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/03.png)

* Kafka 运行在一个或多个数据中心的服务器上作为集群运行
* Kafka 集群存储消息记录的目录被称为 `topics`
* 每一条消息记录包含三个要素：**键（key）、值（value）、时间戳（Timestamp）**

### 核心 API

Kafka 有四个核心API，它们分别是

* Producer API，它允许应用程序向一个或多个 topics 上发送消息记录
* Consumer API，允许应用程序订阅一个或多个 topics 并处理为其生成的记录流
* Streams API，它允许应用程序作为流处理器，从一个或多个主题中消费输入流并为其生成输出流，有效的将输入流转换为输出流。
* Connector API，它允许构建和运行将 Kafka 主题连接到现有应用程序或数据系统的可用生产者和消费者。例如，关系数据库的连接器可能会捕获对表的所有更改

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/04.png)

## Kafka 基本概念

Kafka 作为一个高度可扩展可容错的消息系统，它有很多基本概念，下面就来认识一下这些 Kafka 专属的概念

### topic

Topic 被称为主题，在 kafka 中，使用一个类别属性来划分消息的所属类，划分消息的这个类称为 topic。topic 相当于消息的分配标签，是一个逻辑概念。主题好比是数据库的表，或者文件系统中的文件夹。

### partition

partition 译为分区，topic 中的消息被分割为一个或多个的 partition，它是一个物理概念，对应到系统上的就是一个或若干个目录，一个分区就是一个 `提交日志`。消息以追加的形式写入分区，先后以顺序的方式读取。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/05.png)

>注意：由于一个主题包含无数个分区，因此无法保证在整个 topic 中有序，但是单个 Partition 分区可以保证有序。消息被迫加写入每个分区的尾部。Kafka 通过分区来实现数据冗余和伸缩性

分区可以分布在不同的服务器上，也就是说，一个主题可以跨越多个服务器，以此来提供比单个服务器更强大的性能。

### segment

Segment 被译为段，将 Partition 进一步细分为若干个 segment，每个 segment 文件的大小相等。

### broker

Kafka 集群包含一个或多个服务器，每个 Kafka 中服务器被称为 broker。broker 接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。broker 为消费者提供服务，对读取分区的请求作出响应，返回已经提交到磁盘上的消息。

broker 是集群的组成部分，每个集群中都会有一个 broker 同时充当了 `集群控制器(Leader)`的角色，它是由集群中的活跃成员选举出来的。每个集群中的成员都有可能充当 Leader，Leader 负责管理工作，包括将分区分配给 broker 和监控 broker。集群中，一个分区从属于一个 Leader，但是一个分区可以分配给多个 broker（非Leader），这时候会发生分区复制。这种复制的机制为分区提供了消息冗余，如果一个 broker 失效，那么其他活跃用户会重新选举一个 Leader 接管。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/06.png)

### producer

生产者，即消息的发布者，其会将某 topic 的消息发布到相应的 partition 中。生产者在默认情况下把消息均衡地分布到主题的所有分区上，而并不关心特定消息会被写到哪个分区。不过，在某些情况下，生产者会把消息直接写到指定的分区。

### consumer

消费者，即消息的使用者，一个消费者可以消费多个 topic 的消息，对于某一个 topic 的消息，其只会消费同一个 partition 中的消息

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/07.png)

在了解完 Kafka 的基本概念之后，我们通过搭建 Kafka 集群来进一步深刻认识一下 Kafka。

## 确保安装环境

### 安装 Java 环境

在安装 Kafka 之前，先确保Linux 环境上是否有 Java 环境，使用 `java -version` 命令查看 Java 版本，推荐使用Jdk 1.8 ，如果没有安装 Java 环境的话，可以按照这篇文章进行安装（https://www.cnblogs.com/zs-notes/p/8535275.html）

### 安装 Zookeeper 环境

Kafka 的底层使用 Zookeeper 储存元数据，确保一致性，所以安装 Kafka 前需要先安装 Zookeeper，Kafka 的发行版自带了 Zookeeper ，可以直接使用脚本来启动，不过安装一个 Zookeeper 也不费劲

#### Zookeeper 单机搭建

Zookeeper 单机搭建比较简单，直接从 https://www.apache.org/dyn/closer.cgi/zookeeper/ 官网下载一个稳定版本的 Zookeeper ，这里我使用的是 `3.4.10`，下载完成后，在 Linux 系统中的 `/usr/local` 目录下创建 zookeeper 文件夹，使用`xftp` 工具(xftp 和 xshell 工具都可以在官网 https://www.netsarang.com/zh/xshell/ 申请免费的家庭版)把下载好的 zookeeper 压缩包放到 /usr/local/zookeeper 目录下。

如果下载的是一个 tar.gz 包的话，直接使用 `tar -zxvf zookeeper-3.4.10.tar.gz`解压即可

如果下载的是 zip 包的话，还要检查一下 Linux 中是否有 unzip 工具，如果没有的话，使用 `yum install unzip` 安装 zip 解压工具，完成后使用 `unzip zookeeper-3.4.10.zip`  解压即可。

解压完成后，cd 到 `/usr/local/zookeeper/zookeeper-3.4.10` ，创建一个 data 文件夹，然后进入到 conf 文件夹下，使用 `mv zoo_sample.cfg zoo.cfg` 进行重命名操作

然后使用 vi 打开 zoo.cfg ，更改一下`dataDir = /usr/local/zookeeper/zookeeper-3.4.10/data` ，保存。

进入bin目录，启动服务输入命令` ./zkServer.sh start` 输出下面内容表示搭建成功
![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/08.png)  

关闭服务输入命令，`./zkServer.sh stop`

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/09.png)

使用 `./zkServer.sh status` 可以查看状态信息。

#### Zookeeper 集群搭建

#### 准备条件

准备条件：需要三个服务器，这里我使用了CentOS7 并安装了三个虚拟机，并为各自的虚拟机分配了`1GB`的内存，在每个 `/usr/local/` 下面新建 zookeeper 文件夹，把 zookeeper 的压缩包挪过来，解压，完成后会有 zookeeper-3.4.10 文件夹，进入到文件夹，新建两个文件夹，分别是 `data` 和` log `文件夹

> 注：上一节单机搭建中已经创建了一个data 文件夹，就不需要重新创建了，直接新建一个 log 文件夹，对另外两个新增的服务需要新建这两个文件夹。

#### 设置集群

新建完成后，需要编辑 conf/zoo.cfg 文件，三个文件的内容如下

```properties
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper/zookeeper-3.4.10/data
dataLogDir=/usr/local/zookeeper/zookeeper-3.4.10/log
clientPort=12181
server.1=192.168.1.7:12888:13888
server.2=192.168.1.8:12888:13888
server.3=192.168.1.9:12888:13888
```

server.1 中的这个 1 表示的是服务器的标识也可以是其他数字，表示这是第几号服务器，这个标识要和下面我们配置的 `myid` 的标识一致可以。

`192.168.1.7:12888:13888` 为集群中的 ip 地址，第一个端口表示的是 master 与 slave 之间的通信接口，默认是 2888，第二个端口是leader选举的端口，集群刚启动的时候选举或者leader挂掉之后进行新的选举的端口，默认是 3888

**现在对上面的配置文件进行解释**

`tickTime`: 这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间`维持心跳`的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。

`initLimit`：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 5个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒

`syncLimit`: 这个配置项标识 Leader 与Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是5*2000=10秒

`dataDir`: 快照日志的存储路径

`dataLogDir`: 事务日志的存储路径，如果不配置这个那么事务日志会默认存储到dataDir指定的目录，这样会严重影响zk的性能，当zk吞吐量较大的时候，产生的事务日志、快照日志太多

`clientPort`: 这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。

#### 创建 myid 文件

在了解完其配置文件后，现在来创建每个集群节点的 myid ，我们上面说过，这个 myid 就是 `server.1` 的这个 1 ，类似的，需要为集群中的每个服务都指定标识，使用 `echo` 命令进行创建

```properties
# server.1
echo "1" > /usr/local/zookeeper/zookeeper-3.4.10/data/myid
# server.2
echo "2" > /usr/local/zookeeper/zookeeper-3.4.10/data/myid
# server.3
echo "3" > /usr/local/zookeeper/zookeeper-3.4.10/data/myid
```

#### 启动服务并测试

配置完成，为每个 zk 服务启动并测试，我在 windows 电脑的测试结果如下

**启动服务（每台都需要执行）**

```properties
cd /usr/local/zookeeper/zookeeper-3.4.10/bin
./zkServer.sh start
```

**检查服务状态**

使用 `./zkServer.sh status` 命令检查服务状态

192.168.1.7  --- follower

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/10.png)

192.168.1.8  --- leader

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/11.png)

192.168.1.9  --- follower

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/12.png)

zk集群一般只有一个leader，多个follower，主一般是相应客户端的读写请求，而从主同步数据，当主挂掉之后就会从follower里投票选举一个leader出来。

## Kafka 集群搭建

### 准备条件

- 搭建好的 Zookeeper 集群
- Kafka 压缩包 （https://www.apache.org/dyn/closer.cgi?path=/kafka/2.3.0/kafka_2.12-2.3.0.tgz）

在 `/usr/local` 下新建 `kafka` 文件夹，然后把下载完成的 tar.gz 包移到 /usr/local/kafka 目录下，使用 `tar -zxvf 压缩包` 进行解压，解压完成后，进入到 kafka_2.12-2.3.0 目录下，新建 log 文件夹，进入到 config 目录下

我们可以看到有很多 properties 配置文件，这里主要关注 `server.properties` 这个文件即可。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/13.png)

kafka 启动方式有两种，一种是使用 kafka 自带的 zookeeper 配置文件来启动（可以按照官网来进行启动，并使用单个服务多个节点来模拟集群http://kafka.apache.org/quickstart#quickstart_multibroker），一种是通过使用独立的zk集群来启动，这里推荐使用第二种方式，使用 zk 集群来启动

### 修改配置项

需要为`每个服务`都修改一下配置项，也就是`server.properties`， 需要更新和添加的内容有 

```properties
broker.id=0 //初始是0，每个 server 的broker.id 都应该设置为不一样的，就和 myid 一样 我的三个服务分别设置的是 1,2,3
log.dirs=/usr/local/kafka/kafka_2.12-2.3.0/log

#在log.retention.hours=168 下面新增下面三项
message.max.byte=5242880
default.replication.factor=2
replica.fetch.max.bytes=5242880

#设置zookeeper的连接端口
zookeeper.connect=192.168.1.7:2181,192.168.1.8:2181,192.168.1.9:2181
```

配置项的含义

```properties
broker.id=0  #当前机器在集群中的唯一标识，和zookeeper的myid性质一样
port=9092 #当前kafka对外提供服务的端口默认是9092
host.name=192.168.1.7 #这个参数默认是关闭的，在0.8.1有个bug，DNS解析问题，失败率的问题。
num.network.threads=3 #这个是borker进行网络处理的线程数
num.io.threads=8 #这个是borker进行I/O处理的线程数
log.dirs=/usr/local/kafka/kafka_2.12-2.3.0/log #消息存放的目录，这个目录可以配置为“，”逗号分割的表达式，上面的num.io.threads要大于这个目录的个数这个目录，如果配置多个目录，新创建的topic他把消息持久化的地方是，当前以逗号分割的目录中，那个分区数最少就放那一个
socket.send.buffer.bytes=102400 #发送缓冲区buffer大小，数据不是一下子就发送的，先回存储到缓冲区了到达一定的大小后在发送，能提高性能
socket.receive.buffer.bytes=102400 #kafka接收缓冲区大小，当数据到达一定大小后在序列化到磁盘
socket.request.max.bytes=104857600 #这个参数是向kafka请求消息或者向kafka发送消息的请请求的最大数，这个值不能超过java的堆栈大小
num.partitions=1 #默认的分区数，一个topic默认1个分区数
log.retention.hours=168 #默认消息的最大持久化时间，168小时，7天
message.max.byte=5242880  #消息保存的最大值5M
default.replication.factor=2  #kafka保存消息的副本数，如果一个副本失效了，另一个还可以继续提供服务
replica.fetch.max.bytes=5242880  #取消息的最大直接数
log.segment.bytes=1073741824 #这个参数是：因为kafka的消息是以追加的形式落地到文件，当超过这个值的时候，kafka会新起一个文件
log.retention.check.interval.ms=300000 #每隔300000毫秒去检查上面配置的log失效时间（log.retention.hours=168 ），到目录查看是否有过期的消息如果有，删除
log.cleaner.enable=false #是否启用log压缩，一般不用启用，启用的话可以提高性能
zookeeper.connect=192.168.1.7:2181,192.168.1.8:2181,192.168.1.9:2181 #设置zookeeper的连接端口
```

### 启动 Kafka 集群并测试

- 启动服务，进入到 `/usr/local/kafka/kafka_2.12-2.3.0/bin` 目录下

```properties
# 启动后台进程
./kafka-server-start.sh -daemon ../config/server.properties
```

- 检查服务是否启动

```properties
# 执行命令 jps
6201 QuorumPeerMain
7035 Jps
6972 Kafka
```

- kafka 已经启动
- 创建 Topic 来验证是否创建成功

```properties
# cd .. 往回退一层 到 /usr/local/kafka/kafka_2.12-2.3.0 目录下
bin/kafka-topics.sh --create --zookeeper 192.168.1.7:2181 --replication-factor 2 --partitions 1 --topic cxuan
```

对上面的解释

> --replication-factor 2   复制两份
>
> --partitions 1 创建1个分区
>
> --topic 创建主题

查看我们的主题是否出创建成功

```properties
bin/kafka-topics.sh --list --zookeeper 192.168.1.7:2181
```

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/14.png)

> 启动一个服务就能把集群启动起来

**在一台机器上创建一个发布者**

```properties
# 创建一个broker，发布者
./kafka-console-producer.sh --broker-list 192.168.1.7:9092 --topic cxuantopic
```

**在一台服务器上创建一个订阅者**

```properties
# 创建一个consumer， 消费者
bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.7:9092 --topic cxuantopic --from-beginning
```

> 注意：这里使用 --zookeeper 的话可能出现 `zookeeper is not a recognized option` 的错误，这是因为 kafka 版本太高，需要使用 `--bootstrap-server` 指令

**测试结果**

发布

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/15.png)

消费

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/kafka/16.png)

### 其他命令

显示 topic

```properties
bin/kafka-topics.sh --list --zookeeper 192.168.1.7:2181

# 显示
cxuantopic
```

查看 topic 状态

```properties
bin/kafka-topics.sh --describe --zookeeper 192.168.1.7:2181 --topic cxuantopic

# 下面是显示的详细信息
Topic:cxuantopic PartitionCount:1 ReplicationFactor:2 Configs:
Topic: cxuantopic Partition: 0 Leader: 1 Replicas: 1,2 Isr: 1,2

# 分区为为1  复制因子为2   主题 cxuantopic 的分区为0 
# Replicas: 0,1   复制的为1，2
```

`Leader` 负责给定分区的所有读取和写入的节点，每个节点都会通过随机选择成为 leader。

`Replicas` 是为该分区复制日志的节点列表，无论它们是 Leader 还是当前处于活动状态。

`Isr` 是同步副本的集合。它是副本列表的子集，当前仍处于活动状态并追随Leader。

至此，kafka 集群搭建完毕。

### 验证多节点接收数据

刚刚我们都使用的是 相同的ip 服务，下面使用其他集群中的节点，验证是否能够接受到服务

在另外两个节点上使用

```properties
bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.7:9092 --topic cxuantopic --from-beginning
```

然后再使用 broker 进行消息发送，经测试三个节点都可以接受到消息。

## 配置详解

在搭建 Kafka 的时候我们简单介绍了一下 `server.properties` 中配置的含义，现在我们来详细介绍一下参数的配置和概念

### 常规配置

这些参数是 kafka 中最基本的配置

- broker.id

每个 broker 都需要有一个标识符，使用 broker.id 来表示。它的默认值是 0，它可以被设置成其他任意整数，在集群中需要保证每个节点的 broker.id 都是唯一的。

- port

如果使用配置样本来启动 kafka ，它会监听 9092 端口，修改 port 配置参数可以把它设置成其他任意可用的端口。

- zookeeper.connect

用于保存 broker 元数据的地址是通过 zookeeper.connect 来指定。 localhost:2181 表示运行在本地 2181 端口。该配置参数是用逗号分隔的一组 hostname:port/path 列表，每一部分含义如下：

hostname 是 zookeeper 服务器的服务名或 IP 地址

port 是 zookeeper 连接的端口

/path 是可选的 zookeeper 路径，作为 Kafka 集群的 chroot 环境。如果不指定，默认使用跟路径

- log.dirs

Kafka 把消息都保存在磁盘上，存放这些日志片段的目录都是通过 `log.dirs` 来指定的。它是一组用逗号分隔的本地文件系统路径。如果指定了多个路径，那么 broker 会根据 "最少使用" 原则，把同一分区的日志片段保存到同一路径下。要注意，broker 会向拥有最少数目分区的路径新增分区，而不是向拥有最小磁盘空间的路径新增分区。

- num.recovery.threads.per.data.dir

对于如下 3 种情况，Kafka 会使用可配置的线程池来处理日志片段

服务器正常启动，用于打开每个分区的日志片段；

服务器崩溃后启动，用于检查和截断每个分区的日志片段；

服务器正常关闭，用于关闭日志片段

默认情况下，每个日志目录只使用一个线程。因为这些线程只是在服务器启动和关闭时会用到，所以完全可以设置大量的线程来达到井行操作的目的。特别是对于包含大量分区的服务器来说，一旦发生崩愤，在进行恢复时使用井行操作可能会省下数小时的时间。设置此参数时需要注意，所配置的数字对应的是 log.dirs 指定的单个日志目录。也就是说，如果 num.recovery.threads.per.data.dir 被设为 8，并且 log.dir 指定了 3 个路径，那么总共需要 24 个线程。

- auto.create.topics.enable

默认情况下，Kafka 会在如下 3 种情况下创建主题

当一个生产者开始往主题写入消息时

当一个消费者开始从主题读取消息时

当任意一个客户向主题发送元数据请求时

- delete.topic.enable

如果你想要删除一个主题，你可以使用主题管理工具。默认情况下，是不允许删除主题的，delete.topic.enable 的默认值是 false 因此你不能随意删除主题。这是对生产环境的合理性保护，但是在开发环境和测试环境，是可以允许你删除主题的，所以，如果你想要删除主题，需要把 delete.topic.enable 设为 true。

### 主题默认配置

Kafka 为新创建的主题提供了很多默认配置参数，下面就来一起认识一下这些参数

- num.partitions

num.partitions 参数指定了新创建的主题需要包含多少个分区。如果启用了主题自动创建功能（该功能是默认启用的），主题分区的个数就是该参数指定的值。该参数的默认值是 1。要注意，我们可以增加主题分区的个数，但不能减少分区的个数。

- default.replication.factor

这个参数比较简单，它表示 kafka保存消息的副本数，如果一个副本失效了，另一个还可以继续提供服务default.replication.factor 的默认值为1，这个参数在你启用了主题自动创建功能后有效。

- log.retention.ms

Kafka 通常根据时间来决定数据可以保留多久。默认使用 log.retention.hours 参数来配置时间，默认是 168 个小时，也就是一周。除此之外，还有两个参数 log.retention.minutes 和 log.retentiion.ms 。这三个参数作用是一样的，都是决定消息多久以后被删除，推荐使用 log.retention.ms。

- log.retention.bytes

另一种保留消息的方式是判断消息是否过期。它的值通过参数 `log.retention.bytes` 来指定，作用在每一个分区上。也就是说，如果有一个包含 8 个分区的主题，并且 log.retention.bytes 被设置为 1GB，那么这个主题最多可以保留 8GB 数据。所以，当主题的分区个数增加时，整个主题可以保留的数据也随之增加。

- log.segment.bytes

上述的日志都是作用在日志片段上，而不是作用在单个消息上。当消息到达 broker 时，它们被追加到分区的当前日志片段上，当日志片段大小到达 log.segment.bytes 指定上限（默认为 1GB）时，当前日志片段就会被关闭，一个新的日志片段被打开。如果一个日志片段被关闭，就开始等待过期。这个参数的值越小，就越会频繁的关闭和分配新文件，从而降低磁盘写入的整体效率。

- log.segment.ms

上面提到日志片段经关闭后需等待过期，那么 `log.segment.ms` 这个参数就是指定日志多长时间被关闭的参数和，log.segment.ms 和 log.retention.bytes 也不存在互斥问题。日志片段会在大小或时间到达上限时被关闭，就看哪个条件先得到满足。

- message.max.bytes

broker 通过设置 `message.max.bytes` 参数来限制单个消息的大小，默认是 1000 000， 也就是 1MB，如果生产者尝试发送的消息超过这个大小，不仅消息不会被接收，还会收到 broker 返回的错误消息。跟其他与字节相关的配置参数一样，该参数指的是压缩后的消息大小，也就是说，只要压缩后的消息小于 mesage.max.bytes，那么消息的实际大小可以大于这个值

这个值对性能有显著的影响。值越大，那么负责处理网络连接和请求的线程就需要花越多的时间来处理这些请求。它还会增加磁盘写入块的大小，从而影响 IO 吞吐量。



**这是我(cxuan)系列文章的第一篇，如果有帮助，欢迎分享和转发，敬请期待下一篇文章。**



文章参考：

[Kafka【第一篇】Kafka集群搭建](https://www.cnblogs.com/luotianshuai/p/5206662.html)

https://juejin.im/post/5ba792f5e51d450e9e44184d

https://blog.csdn.net/k393393/article/details/93099276

《Kafka权威指南》

https://www.learningjournal.guru/courses/kafka/kafka-foundation-training/broker-configurations/

