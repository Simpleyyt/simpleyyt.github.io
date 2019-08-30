---
layout: post
category: Java
title: rocketmq 部署启动指南-Docker 版
tagline: by 小黑
tags: 
  - rocketmq
published: true
---

最近学习使用 rocketmq，需要搭建 rocketmq 服务端，本文主要记录 rocketmq 搭建过程以及这个过程踩到的一些坑。

<!--more-->

## 准备工作

在搭建之前，我们需要做一些准备工作，这里我们需要使用 docker 搭建服务，所以需要提前安装 docker。此外，由于 rocketmq 需要部署 broker 与 nameserver ，考虑到分开部署比较麻烦，这里将会使用 docker-compose。

*rocketmq 架构图如下:*

![rocketmq](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190826/rmq-basic-arc.png)

另外，还需要搭建一个 web 可视化控制台，可以监控 mq 服务状态，以及消息消费情况，这里使用 rocketmq-console，同样该程序也将使用 docker 安装。

## 部署过程

首先我们需要 rocketmq docker 镜像，这里我们可以选择自己制作，直接拉取 [git@github.com:apache/rocketmq-docker.git](git@github.com:apache/rocketmq-docker.git) ，然后再制作镜像。 另外还可以直接使用 docker hub 上官方制作的镜像，镜像名： `rocketmqinc/rocketmq`。

接着创建 mq 配置文件 `broker.conf`，文件放置到 `/opt/rocketmq/conf` ，配置如下:

```properties
brokerClusterName = DefaultCluster  
brokerName = broker-a  
brokerId = 0  
deleteWhen = 04  
fileReservedTime = 48  
brokerRole = ASYNC_MASTER  


flushDiskType = ASYNC_FLUSH  
# 如果是本地程序调用云主机 mq，这个需要设置成 云主机 IP
brokerIP1=10.10.101.80 

```

在创建如下文件夹：`/opt/rocketmq/logs`，`/opt/rocketmq/store`，最后创建 docker-compose.yml 文件，配置如下：

```yaml
version: '2'
services:
  namesrv:
    image: rocketmqinc/rocketmq
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - /opt/rocketmq/logs:/home/rocketmq/logs
      - /opt/rocketmq/store:/home/rocketmq/store
    command: sh mqnamesrv
  broker:
    image: rocketmqinc/rocketmq
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    volumes:
      - /opt/rocketmq/logs:/home/rocketmq/logs
      - /opt/rocketmq/store:/home/rocketmq/store
      - /opt/rocketmq/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf
    #command: sh mqbroker -n namesrv:9876
    command: sh mqbroker -n namesrv:9876 -c ../conf/broker.conf
    depends_on:
      - namesrv
    environment:
      - JAVA_HOME=/usr/lib/jvm/jre
  console:
    image: styletang/rocketmq-console-ng
    container_name: rocketmq-console-ng
    ports:
      - 8087:8080
    depends_on:
      - namesrv
    environment:
      - JAVA_OPTS= -Dlogging.level.root=info   -Drocketmq.namesrv.addr=rmqnamesrv:9876 
      - Dcom.rocketmq.sendMessageWithVIPChannel=false
```

**注意点**

这里需要注意 rocketmq broker 与 rokcetmq-console 都需要与 rokcetmq nameserver 连接，需要知道 nameserver ip。使用 docker-compose 之后，上面三个 docker 容器将会一起编排，**可以直接使用容器名代替容器 ip**，如这里 nameserver 容器名 rmqnamesrv。

配置完成之后，运行 docker-compose up 启动三个容器，启动成功后，访问 ip:8087，查看 mq 外部控制台，如果可以看到以下信息，rocketmq 服务启动成功。

![mq 控制台](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190826/image-4ef98b7a.png)

## 初体验 rocketmq

这里将会使用 springboot 快速上手使用 mq，将会使用 `rocketmq-spring-boot-starter` 模块，pom 配置如下：

```pom
<!--在pom.xml中添加依赖-->
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.0.3</version>
</dependency>
```

**消费服务发送方配置如下：**

```properties
## application.properties
rocketmq.name-server=ip:9876
rocketmq.producer.group=my-group
```

**消费服务发送方程序如下：**

```java
@SpringBootApplication
public class ProducerApplication implements CommandLineRunner {
    @Resource
    private RocketMQTemplate rocketMQTemplate;

    public static void main(String[] args){
        SpringApplication.run(ProducerApplication.class, args);
    }

    public void run(String... args) throws Exception {
        rocketMQTemplate.convertAndSend("test-topic-1", "Hello, World!");
        rocketMQTemplate.send("test-topic-1", MessageBuilder.withPayload("Hello, World! I'm from spring message").build());
    }

}
```

**消息消费方配置如下：**

```
## application.properties
rocketmq.name-server=ip:9876
```

**消息消费方运行程序如下：**

```java
@SpringBootApplication
public class ConsumerApplication{

    public static void main(String[] args){
        SpringApplication.run(ConsumerApplication.class, args);
    }

    @Slf4j
    @Service
    @RocketMQMessageListener(topic = "test-topic-1", consumerGroup = "my-consumer_test-topic-1")
    public static class MyConsumer1 implements RocketMQListener<String> {
        public void onMessage(String message) {
            log.info("received message: {}", message);
        }
    }
}
```

## 相关问题

1. 消息发送方消息发送异常，异常如图所示：`Caused by: org.apache.rocketmq.remoting.exception.RemotingTooMuchRequestException: sendDefaultImpl call timeout`。

该异常是由于 brokerip 未设置正确导致，登录 mq 服务控制台，可以查看 broker 配置信息。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190826/image-e2578861.png)

上面 `192.168.128.3:10911` 是 docker 容器 IP，这是一个主机内部 IP。这里需要将 IP 设置为云主机的 IP，需要在 `broker.conf ` 修改 `brokerIP1` 参数。

2. mq 控制台无法正常查看 mq 服务信息。

这个问题主要是 nameserver ip 设置错误导致。查看 mq 控制台运维页面，可以看到此时连接的 nameserver 地址信息。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190826/image-a7437e6a.png)

可以看到这里设置的地址为：`127.0.0.1:9876`。由于这里 mq 控制台使用 docker 容器，容器内直接访问 `127.0.0.1:9876` 将会访问自己内部，而非宿主机内正确程序。

这里需要在 docker 配置环境变量，配置如下：

```
- JAVA_OPTS= -Dlogging.level.root=info   -Drocketmq.namesrv.addr=rmqnamesrv:9876 
```

## 帮助文档
[rocketmq-docker](https://github.com/apache/rocketmq-docker)  
[RocketMq docker 搭建和基本概念]([https://www.jianshu.com/p/072f18681351?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation](https://www.jianshu.com/p/072f18681351?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation))  
[RocketMQ-Spring](https://github.com/apache/rocketmq-spring/blob/master/README_zh_CN.md)