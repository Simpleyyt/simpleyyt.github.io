---
layout: post
title: kafka的序列化和反序列化
tagline: by 小马
categories: kafka
tags: 
    - 小马
---

 
## 简介

kafka内部发送和接收消息的时候，使用的是byte[]字节数组的方式(RPC底层也是用这种通讯格式)。但是我们在应用层其实可以使用更多的数据类型，比如int，short，
long，String等，这归功于kafka的序列化和反序列化机制。

<!--more-->
 

## 基本原理分析

在之前的一篇文章[springboot集成kafka示例](http://www.machengyu.net/arch/2019/07/29/kafka-springboot.html)中，我使用的是kafka原生的StringSerializer序列化方式，

```xml

spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

```

源码如下：

```java
public class StringSerializer implements Serializer<String> {
    private String encoding = "UTF8";

    public StringSerializer() {
    }

    public void configure(Map<String, ?> configs, boolean isKey) {
        String propertyName = isKey ? "key.serializer.encoding" : "value.serializer.encoding";
        Object encodingValue = configs.get(propertyName);
        if (encodingValue == null) {
            encodingValue = configs.get("serializer.encoding");
        }

        if (encodingValue instanceof String) {
            this.encoding = (String)encodingValue;
        }

    }

    public byte[] serialize(String topic, String data) {
        try {
            return data == null ? null : data.getBytes(this.encoding);
        } catch (UnsupportedEncodingException var4) {
            throw new SerializationException("Error when serializing string to byte[] due to unsupported encoding " + this.encoding);
        }
    }

    public void close() {
    }
}

```

其实很简单，configure方法设置序列化(serialize方法)需要使用的编码，如果没有设置就使用UTF8格式。这个方法是在生成producer实例的时候被调用的。serialize方法使用的就是String的getBytes把String类型的消息转化为byte字节数组。 

反序列呢？聪明如你应该能想到，使用new String就可以解决了。源码如下，

 ```java
 
@Override
    public String deserialize(String topic, byte[] data) {
        try {
            if (data == null)
                return null;
            else
                return new String(data, encoding);
        } catch (UnsupportedEncodingException e) {
            throw new SerializationException("Error when deserializing byte[] to string due to unsupported encoding " + encoding);
        }
    }
--------------------- 

```

是不是简单到爆呢？

其它的内置序列化组件，像Double, Integer,Long这些原理都类似，就不一一分析了。

## 自定义序列化组件

有时候内置的组件不能满足我们的需要。比如我有个自定义的对象要作为kafka的消息进行收发（把对象转化为json字符串通过String的方式也是一种思路），希望能有一个针对我这个对象自定义的序列化和反序列化组件。

我们先定义一个消息对象，

```java
@Data
@ToString
public class Person {
    private int id;
    private String name;
    private int age;

}
```

然后自定义自己的序列化和反序列化实现类，

```java
@Slf4j
public class PersonDeserializer implements Deserializer <Person> {

    @Override
    public void configure(Map<String, ?> map, boolean b) {

    }

    @Override
    public Person deserialize(String s, byte[] bytes) {
        log.info("自定义的反序列化-deserialize");
        return JSON.parseObject(bytes, Person.class);
    }

    @Override
    public void close() {

    }
}
```

```java
@Slf4j
public class PersonSerializer implements Serializer<Person> {
    private static Gson gson;
    static {
        gson = new GsonBuilder().create();
    }

    @Override
    public void configure(Map<String, ?> map, boolean b) {
        log.info("自定义的序列化组件--configure");
    }

    @Override
    public byte[] serialize(String s, Person person) {
        log.info("自定义的序列化组件--serialize");
        return JSON.toJSONBytes(person);
    }

    @Override
    public void close() {
        log.info("自定义的序列化组件--close");
    }
}

```

代码一看就明白，其实核心就是利用fastjson的toJSONBytes把对象转化为byte数组。

然后我们在配置里指定使用我们自己的序列化和反序列化实现类，

```xml
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=com.ponymaggie.github.kafka.serializer.PersonSerializer


spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=com.ponymaggie.github.kafka.serializer.PersonDeserializer
```


## 测试

本地kafka和zk环境搭建可以参考我之前的一篇文章：

http://www.machengyu.net/arch/2019/07/29/kafka-springboot.html

启动springboot项目，通过日志可以看出消息的收发都是正常的。

```text
2019-08-15 20:06:34.251  INFO 16676 --- [           main] c.p.github.kafka.producer.KafkaSender    : +++++++++++++++++++++  message = {"id":1000,"name":"小明","age":30}

2019-08-15 20:06:34.648  INFO 16676 --- [ntainer#0-0-C-1] c.p.github.kafka.consumer.KafkaReceiver  : ----------------- record =ConsumerRecord(topic = malu, partition = 0, offset = 0, CreateTime = 1565870794430, serialized key size = -1, serialized value size = 36, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = Person(id=1000, name=小明, age=30))
```

源码地址:

https://github.com/pony-maggie/springboot-kafka-serialization-demo


 









