---
layout: post
title:  微服务异步架构---MQ之RocketMQ
tagline: by 贾思谦
categories: 微服务
tags: 
    - 贾思谦
---

"我们大家都知道把一个微服务架构变成一个异步架构只需要加一个MQ，现在市面上有很多MQ的开源框架。到底选择哪一个MQ的开源框架才合适呢?"

<!--more-->

## 1

## 什么是MQ？MQ的原理是什么？

------



MQ就是消息队列，是Message Queue的缩写。消息队列是一种通信方式。消息的本质就是一种数据结构。因为MQ把项目中的消息集中式的处理和存储，所以MQ主要有解耦，并发，和削峰的功能。



#### 1，解耦：

MQ的消息生产者和消费者互相不关心对方是否存在，通过MQ这个中间件的存在，使整个系统达到解耦的作用。



如果服务之间用RPC通信，当一个服务跟几百个服务通信时，如果那个服务的通信接口改变，那么几百个服务的通信接口都的跟着变动，这是非常头疼的一件事。



但是采用MQ之后，不管是生产者或者消费者都可以单独改变自己。他们的改变不会影响到别的服务。从而达到解耦的目的。为什么要解耦呢？说白了就是方便，减少不必要的工作量。



#### 2，并发

MQ有生产者集群和消费者集群，所以客户端是亿级用户时，他们都是并行的。从而大大提升响应速度。



#### 3，削峰

因为MQ能存储的消息量很大，所以他可以把大量的消息请求先存下了，然后再并发的方式慢慢处理。



如果采用RPC通信，每一次请求用调用RPC接口，当请求量巨大的时候，因为RPC的请求是很耗资源的，所以巨大的请求一定会压垮服务器。



削峰的目的是用户体验变好，并且使整个系统稳定。能承受大量请求消息。



## 2

## 现在市面上有什么MQ，

## 重点介绍RocketMQ



------



现在市面上的MQ有很多，主要有RabbitMQ，ActiveMQ，ZeroMQ，RocketMQ，Kafka等等，这些都是开源的MQ产品。以前很多人推荐使用RabbitMQ，他也是非常好用的MQ产品，这里不做过多的介绍。Kafka也是高吞吐量的老大，我们这里也不介绍。



我们重点介绍一下RocketMQ，RocketMQ是阿里巴巴在2012年开源的分布式消息中间件，目前已经捐赠给Apache软件基金会，并于并于2017年9月25日成为 Apache 的顶级项目。



作为经历过多次阿里巴巴双十一这种“超级工程”的洗礼并有稳定出色表现的国产中间件，以其高性能、低延时和高可靠等特性近年来已经也被越来越多的国内企业使用。



**功能概览图**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/option.png)



可以看见RocketMQ支持定时和延时消息，这是RabbitMQ所没有的能力。



**RocketMQ的物理结构**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/RocketMQ.png)

从这里可以看出，RocketMQ涉及到四大集群，producer，Name Server，Consumer，Broker。



### Producer集群：



是生产者集群，负责产生消息，向消费者发送由业务应用程序系统生成的消息，RocketMQ提供三种方式发送消息：同步，异步，单向。

 

#### 一，普通消息

**1，同步原理图**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/syn.png)



**同步消息关键代码**

``` java
try {
        SendResult sendResult = producer.send(msg);
        // 同步发送消息，只要不抛异常就是成功
        if (sendResult != null) {
        System.out.println(new Date() + " Send mq message success. Topic is:" + msg.getTopic() + " msgId is: " + sendResult.getMessageId());

    }
    catch (Exception e) {

        System.out.println(new Date() + " Send mq message failed. Topic is:" + msg.getTopic());
        e.printStackTrace();
    }
}
```

**2，异步原理图**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/asyn.png)

**异步消息关键代码**

``` java
producer.sendAsync(msg, new SendCallback() {

@Override
public void onSuccess(final SendResult sendResult) {

       // 消费发送成功
      System.out.println("send message success. topic=" + sendResult.getTopic() + ", msgId=" + sendResult.getMessageId());

 }
 
@Override
public void onException(OnExceptionContext context) { 

   System.out.println("send message failed. topic=" + context.getTopic() + ", msgId=" + context.getMessageId());

}
});
  
```


**3，单向（Oneway）发送原理图**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/oneway.png)

单向只发送，不等待返回，所以速度最快，一般在微秒级，但可能丢失



**单向（Oneway）发送消息关键代码**

``` java

producer.sendOneway(msg);

```



三种发送消息具体代码请参考文档：https://help.aliyun.com/document_detail/29547.html?spm=a2c4g.11186623.6.566.7e49793fuueSlB


#### 二，定时消息和延时消息



**发送定时消息关键代码**



``` java
try {

     // 定时消息，单位毫秒（ms），在指定时间戳（当前时间之后）进行投递，例如 2016-03-07 16:21:00 投递。如果被设置成当前时间戳之前的某个时刻，消息将立刻投递给消费者。
    long timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse("2016-03-07 16:21:00").getTime();
    msg.setStartDeliverTime(timeStamp);

    // 发送消息，只要不抛异常就是成功
    SendResult sendResult = producer.send(msg);
    System.out.println("MessageId:"+sendResult.getMessageId());

}
catch (Exception e) {

    // 消息发送失败，需要进行重试处理，可重新发送这条消息或持久化这条数据进行补偿处理
    System.out.println(new Date() + " Send mq message failed. Topic is:" + msg.getTopic());

    e.printStackTrace();

 }
    
```      


**发送延时消息关键代码**


``` java
try {
   
    // 延时消息，单位毫秒（ms），在指定延迟时间（当前时间之后）进行投递，例如消息在 3 秒后投递
    long delayTime = System.currentTimeMillis() + 3000;

    // 设置消息需要被投递的时间 msg.setStartDeliverTime(delayTime);
     SendResult sendResult = producer.send(msg);
     // 同步发送消息，只要不抛异常就是成功
     if (sendResult != null) {
        System.out.println(new Date() + " Send mq message success. Topic is:" + msg.getTopic() + " msgId is: " + sendResult.getMessageId());
      }
      
} catch (Exception e) {
   
   // 消息发送失败，需要进行重试处理，可重新发送这条消息或持久化这条数据进行补偿处理
    System.out.println(new Date() + " Send mq message failed. Topic is:" + msg.getTopic());

    e.printStackTrace();

 }
    
```

**注意事项**

1，定时和延时消息的 msg.setStartDeliverTime 参数需要设置成当前时间戳之后的某个时刻（单位毫秒）。如果被设置成当前时间戳之前的某个时刻，消息将立刻投递给消费者。

2，定时和延时消息的 msg.setStartDeliverTime 参数可设置40天内的任何时刻（单位毫秒），超过40天消息发送将失败。

3，StartDeliverTime 是服务端开始向消费端投递的时间。 如果消费者当前有消息堆积，那么定时和延时消息会排在堆积消息后面，将不能严格按照配置的时间进行投递。

4，由于客户端和服务端可能存在时间差，消息的实际投递时间与客户端设置的投递时间之间可能存在偏差。

5，设置定时和延时消息的投递时间后，依然受 3 天的消息保存时长限制。例如，设置定时消息 5 天后才能被消费，如果第 5 天后一直没被消费，那么这条消息将在第8天被删除。

6，除 Java 语言支持延时消息外，其他语言都不支持延时消息。

**发布消息原理图**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/Java_SDK_Consumer.png)

#### 三，事务消息



RocketMQ提供类似X/Open XA的分布式事务功能来确保业务发送方和MQ消息的最终一致性，其本质是通过半消息的方式把分布式事务放在MQ端来处理。



**原理图**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/message.png)



其中：

​          1，发送方向消息队列 RocketMQ 服务端发送消息。

​          2，服务端将消息持久化成功之后，向发送方 ACK 确认消息已经发送成功，此时消息为半消息。

​          3，发送方开始执行本地事务逻辑。

​          4，发送方根据本地事务执行结果向服务端提交二次确认（Commit 或是 Rollback），服务端收到 Commit 状态则将半消息标记为可投递，订阅方最终将收到该消息；服务端收到 Rollback 状态则删除半消息，订阅方将不会接受该消息。

​          5，在断网或者是应用重启的特殊情况下，上述步骤 4 提交的二次确认最终未到达服务端，经过固定时间后服务端将对该消息发起消息回查。

​          6，发送方收到消息回查后，需要检查对应消息的本地事务执行的最终结果。

​         7，发送方根据检查得到的本地事务的最终状态再次提交二次确认，服务端仍按照步骤 4 对半消息进行操作。





**RocketMQ的半消息机制的注意事项是**

1，根据第六步可以看出他要求发送方提供业务回查接口。

2，不能保证发送方的消息幂等，在ack没有返回的情况下，可能存在重复消息

3，消费方要做幂等处理。



**核心代码**

**final BusinessService businessService = new BusinessService(); // 本地业务**


``` java
TransactionProducer producer = ONSFactory.createTransactionProducer(properties,
new LocalTransactionCheckerImpl());

producer.start();

Message msg = new Message("Topic", "TagA", "Hello MQ transaction===".getBytes());

try {
    
    SendResult sendResult = producer.send(msg, new LocalTransactionExecuter() {
    
    @Override
    public TransactionStatus execute(Message msg, Object arg) {
    
        // 消息 ID（有可能消息体一样，但消息 ID 不一样，当前消息 ID 在控制台无法查询）
        String msgId = msg.getMsgID();
        
        // 消息体内容进行 crc32，也可以使用其它的如 MD5
        long crc32Id = HashUtil.crc32Code(msg.getBody());
       
        // 消息 ID 和 crc32id 主要是用来防止消息重复
        // 如果业务本身是幂等的，可以忽略，否则需要利用 msgId 或 crc32Id 来做幂等
        // 如果要求消息绝对不重复，推荐做法是对消息体 body 使用 crc32 或 MD5 来防止重复消息
        Object businessServiceArgs = new Object();
        
        TransactionStatus transactionStatus =TransactionStatus.Unknow;
        
        try {
        
        boolean isCommit = businessService.execbusinessService(businessServiceArgs);
        
        if (isCommit) {
        
        // 本地事务成功则提交消息 transactionStatus = TransactionStatus.CommitTransaction;
        
        } else {
        
        // 本地事务失败则回滚消息 transactionStatus = TransactionStatus.RollbackTransaction;
        
        }
        
        } catch (Exception e) {log.error("Message Id:{}", msgId, e);
        
        }
        
        System.out.println(msg.getMsgID());log.warn("Message Id:{}transactionStatus:{}", msgId, transactionStatus.name());
         
         return transactionStatus;
    }
    }, null);
    }

catch (Exception e) {
  
  // 消息发送失败，需要进行重试处理，可重新发送这条消息或持久化这条数据进行补偿处理
   System.out.println(new Date() + " Send mq message failed. Topic is:" + msg.getTopic());

   e.printStackTrace();

}
    
``` 
       



具体代码参考文档：https://help.aliyun.com/document_detail/29548.html?spm=a2c4g.11186623.6.570.5d5738a49FJl1t

**所有消息发布原理图**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/Java_SDK_Producer.png)

producer完全无状态，可以集群部署。



### Name Server集群：

NameServer是一个几乎无状态的节点，可集群部署，节点之间无任何信息同步，NameServer很像注册中心的功能。

听说阿里之前的NameServer 是用ZooKeeper做的，可能因为Zookeeper不能满足大规模并发的要求，所以之后NameServer 是阿里自研的。

NameServer其实就是一个路由表，他管理Producer和Comsumer之间的发现和注册。



### Broker集群：

Broker部署相对复杂，Broker分为Master与Slave，一个Master可以对应多个Slaver，但是一个Slaver只能对应一个Master，Master与Slaver的对应关系通过指定相同的BrokerName。

不同的BrokerId来定义，BrokerId为0表示Master，非0表示Slaver。Master可以部署多个。每个Broker与NameServer集群中的所有节点建立长连接，定时注册Topic信息到所有的NameServer。



### Consumer集群：

#### 订阅方式

消息队列 RocketMQ 支持以下两种订阅方式：

**集群订阅：**同一个 Group ID 所标识的所有 Consumer 平均分摊消费消息。 例如某个 Topic 有 9 条消息，一个 Group ID 有 3 个 Consumer 实例，那么在集群消费模式下每个实例平均分摊，只消费其中的 3 条消息。

``` java

// 集群订阅方式设置（不设置的情况下，默认为集群订阅方式）
properties.put(PropertyKeyConst.MessageModel, PropertyValueConst.CLUSTERING);

```


**广播订阅：**同一个 Group ID 所标识的所有 Consumer 都会各自消费某条消息一次。 例如某个 Topic 有 9 条消息，一个 Group ID 有 3 个 Consumer 实例，那么在广播消费模式下每个实例都会各自消费 9 条消息。

``` java

// 广播订阅方式设置
properties.put(PropertyKeyConst.MessageModel, PropertyValueConst.BROADCASTING);
``` 


**订阅消息关键代码：**


``` java

Consumer consumer = ONSFactory.createConsumer(properties);

consumer.subscribe("TopicTestMQ", "TagA||TagB", **new** MessageListener() { //订阅多个 Tag

public Action consume(Message message, ConsumeContext context) {

   System.out.println("Receive: " + message);
   return Action.CommitMessage;
}
});


//订阅另外一个 Topic

consumer.subscribe("TopicTestMQ-Other", "*", **new** MessageListener() { //订阅全部 Tag

public Action consume(Message message, ConsumeContext context) {

    System.out.println("Receive: " + message);
    return Action.CommitMessage;
}
});

consumer.start();
    
```   

#### 注意事项：

消费端要做幂等处理，所有MQ基本上都不会做幂等处理，需要业务端处理，原因是如果在MQ端做幂等处理会带来MQ的复杂度，而且严重影响MQ的性能。

**消息收发模型**

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/pub-submodel.png)



### 主子账号创建

创建主子账号的原因是权限问题。下面是主账号创建流程图

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/main_process.png)

详细操作地址：https://help.aliyun.com/document_detail/34411.html?spm=a2c4g.11186623.6.555.38c57f91JXUK7o

子账号流程图

![](http://www.justdojava.com/assets/images/2019/java/image_jsq/04_03/quickstart-process-ram-subaccount.png)

详细操作地址：https://help.aliyun.com/document_detail/96402.html?spm=a2c4g.11186623.6.556.60194fedfSkxIB



## 3

## MQ是微服务架构

## 非常重要的部分





------



MQ的诞生把原来的同步架构思维转变到异步架构思维提供一种方法，为大规模，高并发的业务场景的稳定性实现提供了很好的解决思路。



Martin Fowler强调：分布式调用的第一原则就是不要分布式。这句话看似颇具哲理，然而就企业应用系统而言，只要整个系统在不停地演化，并有多个子系统共同存在时，这条原则就会被迫打破。



Martin Fowler提出的这条原则，一方面是希望设计者能够审慎地对待分布式调用，另一方面却也是分布式系统自身存在的缺陷所致。



所以微服务并不是万能药，适合的架构才是最好的架构。
