---
layout: post
title: springboot集成kafka示例
tagline: by 小马
category: java
tags: [Kafka,springboot,broker,bootstrap-server]
---

* content
{:toc}


源码地址:

https://github.com/pony-maggie/springboot-kafka-demo

----------------------


## 本地kafka和zk环境

我们需要在本地启动一个单机版的kafka和zookeeper环境。kafka的安装包自带zookeeper，直接启动即可，这个详细过程不是本文的重点，不详细说了。

我的本地环境配置如下：

* win10系统
* kafka_2.12-1.1.1
* zookeeper-3.4.9
* spring boot 2.1.6.RELEASE

启动zk，端口是2181

```
C:\kafka\kafka_2.12-1.1.1
λ .\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
```

启动kafka，端口是

```
C:\kafka\kafka_2.12-1.1.1
λ .\bin\windows\kafka-server-start.bat .\config\server.properties
```

记得查看启动日志确认启动成功才行。

用kafka自带的工具创建一个topic试试：

```
C:\kafka\kafka_2.12-1.1.1
λ .\bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test20190713
Created topic "test20190713".
```

可以看到创建成功了。然后我们查询下kafka的topic，

```
C:\kafka\kafka_2.12-1.1.1
λ .\bin\windows\kafka-topics.bat --list --zookeeper localhost:2181
test20190713
```


然后我们可以用kafka自带的生产者和消费者工具进行测试，进一步验证本地环境。

首先分别启动生产者和消费者，

```
C:\kafka\kafka_2.12-1.1.1
λ .\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic test20190713
>this is a test
>
```

```
C:\kafka\kafka_2.12-1.1.1
λ .\bin\windows\kafka-console-consumer.bat --zookeeper localhost:2181 --topic test20190713
Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].
```

在消费者的窗口输入消息，很快消费者窗口就会显示出该消息了。或者消费者启动也可以用下面的方式：

```
C:\kafka\kafka_2.12-1.1.1
λ .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test20190713
```

原因可以参考：

[Kafka中的broker-list,bootstrap-server以及zookeeper](https://blog.csdn.net/pony_maggie/article/details/95862515)

 下面两个如何配置
 


## 创建demo项目工程

### 依赖

```xml
<dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
```

### 配置

```xml
#============== kafka ===================
# 指定kafka 代理地址，可以多个
spring.kafka.bootstrap-servers=127.0.0.1:9092

#=============== 生产者配置=======================

spring.kafka.producer.retries=0
spring.kafka.producer.batch-size=16384
spring.kafka.producer.buffer-memory=33554432
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

#===============消费者配置=======================
# 指定默认消费者group id
spring.kafka.consumer.group-id=test-consumer-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.enable-auto-commit=true
spring.kafka.consumer.auto-commit-interval=100
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

先来解释下这几个配置，

* bootstrap-servers：连接kafka的地址，多个地址用逗号分隔
* batch-size：当将多个记录被发送到同一个分区时， Producer 将尝试将记录组合到更少的请求中。这有助于提升客户端和服务器端的性能。这个配置控制一个批次的默认大小（以字节为单位）。16384是缺省的配置
* retries：若设置大于0的值，客户端会将发送失败的记录重新发送
* buffer-memory：Producer 用来缓冲等待被发送到服务器的记录的总字节数，33554432是缺省配置
* key-serializer：关键字的序列化类
* value-serializer：值的序列化类


**到这里配置就可以结束了，目前spring-kafka已经和spring boot无缝对接，可以自动加载配置文件进行配置，我们不需要再单独定义配置类。**


### 测试代码

我们先定义一个消息实体，方便消费者和生产者共享。

```java
@Data
public class Message {
    private Long id;    //id
    private String msg; //消息
    private Date sendTime;  //时间戳

}
```

然后是生产者，

```java
@Component
@Slf4j
public class KafkaSender {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    private Gson gson = new GsonBuilder().create();

    //发送消息方法
    public void send() {
        Message message = new Message();
        message.setId(System.currentTimeMillis());
        message.setMsg(UUID.randomUUID().toString());
        message.setSendTime(new Date());
        log.info("+++++++++++++++++++++  message = {}", gson.toJson(message));
        kafkaTemplate.send("malu", gson.toJson(message));
    }
}
```
代码很简单，不做过多解释。

然后是消费者，

```java
@Component
@Slf4j
public class KafkaReceiver {

    @KafkaListener(topics = {"malu"})
    public void listen(ConsumerRecord<?, ?> record) {

        Optional<?> kafkaMessage = Optional.ofNullable(record.value());

        if (kafkaMessage.isPresent()) {

            Object message = kafkaMessage.get();

            log.info("----------------- record =" + record);
            log.info("------------------ message =" + message);
        }

    }
}
```

只需要在监听的方法上通过注解配置一个监听器即可，另外就是指定需要监听的topic。

kafka的消息再接收端会被封装成ConsumerRecord对象返回，它内部的value属性就是实际的消息。


## 测试

启动springboot项目，通过日志可以看出消息的收发都是正常的。

```
2019-07-29 15:02:28.812  INFO 13468 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 2.0.1
2019-07-29 15:02:28.812  INFO 13468 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : fa14705e51bd2ce5
2019-07-29 15:02:28.819  INFO 13468 --- [ad | producer-1] org.apache.kafka.clients.Metadata        : Cluster ID: eGkIiJuNTHGwNkZy31j2NQ

2019-07-29 15:02:31.831  INFO 13468 --- [           main] c.p.github.kafka.producer.KafkaSender    : +++++++++++++++++++++  message = {"id":1564383751831,"msg":"7c4f3344-d366-453f-ba12-4ca091171636","sendTime":"Jul 29, 2019 3:02:31 PM"}
2019-07-29 15:02:31.839  INFO 13468 --- [ntainer#0-0-C-1] c.p.github.kafka.consumer.KafkaReceiver  : ----------------- record =ConsumerRecord(topic = malu, partition = 0, offset = 1, CreateTime = 1564383751831, serialized key size = -1, serialized value size = 102, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = {"id":1564383751831,"msg":"7c4f3344-d366-453f-ba12-4ca091171636","sendTime":"Jul 29, 2019 3:02:31 PM"})
2019-07-29 15:02:31.839  INFO 13468 --- [ntainer#0-0-C-1] c.p.github.kafka.consumer.KafkaReceiver  : ------------------ message ={"id":1564383751831,"msg":"7c4f3344-d366-453f-ba12-4ca091171636","sendTime":"Jul 29, 2019 3:02:31 PM"}
2019-07-29 15:02:34.833  INFO 13468 --- [           main] c.p.github.kafka.producer.KafkaSender    : +++++++++++++++++++++  message = {"id":1564383754833,"msg":"7d515786-09f4-41f0-b512-99e3489c1d82","sendTime":"Jul 29, 2019 3:02:34 PM"}
2019-07-29 15:02:34.848  INFO 13468 --- [ntainer#0-0-C-1] c.p.github.kafka.consumer.KafkaReceiver  : ----------------- record =ConsumerRecord(topic = malu, partition = 0, offset = 2, CreateTime = 1564383754834, serialized key size = -1, serialized value size = 102, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = {"id":1564383754833,"msg":"7d515786-09f4-41f0-b512-99e3489c1d82","sendTime":"Jul 29, 2019 3:02:34 PM"})
2019-07-29 15:02:34.849  INFO 13468 --- [ntainer#0-0-C-1] c.p.github.kafka.consumer.KafkaReceiver  : ------------------ message ={"id":1564383754833,"msg":"7d515786-09f4-41f0-b512-99e3489c1d82","sendTime":"Jul 29, 2019 3:02:34 PM"}
```

我们在代码里创建了一个名为 "malu" 的topic，可以通过命令查询下：

```
C:\kafka\kafka_2.12-1.1.1                                          
λ .\bin\windows\kafka-topics.bat --list --zookeeper localhost:2181
__consumer_offsets                                                 
malu                                                               
test20190713                                                       
```

## 其它说明

如果启动的时候报错，需要考虑springboot和spring-kafka兼容性问题。比如一开始我启动的时候就报错：

```
Error creating bean with name 'org.springframework.boot.autoconfigure.kafka.KafkaAnnotationDrivenConfiguration'
```

后来把spring-kafka的版本升级下就好了。具体的版本对应关系可以看下官方的说明：

https://spring.io/projects/spring-kafka


参考：

http://kafka.apachecn.org/documentation.html

http://www.54tianzhisheng.cn/2018/01/05/SpringBoot-Kafka/


















