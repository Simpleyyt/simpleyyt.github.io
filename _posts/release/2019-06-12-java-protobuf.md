---
layout: post
category: java
title: SpringBoot 集成 Protocol Buffer案例
tagline: by 子悠
tags: 
  - java
published: true
---

### 背景
最近工作中使用到 Protobuf，发现 Protobuf 的强大和方便之处，今天给大家介绍一个这个强大的工具的使用。毕竟有好东西要一起分享。
<!--more-->

### 什么是 Protobuf
首先我们来看下什么是 protobuf，
官方解释：Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler.
> 意思是: Protocol buffers是Google的语言中立，平台中立，可扩展的机制，用于序列化结构化数据 - 类似XML，但更小，更快，更简单。
> 使用 Protobuf 你可以编写一次结构化数据一次，然后可以使用各种语言工具来生成对应语言的源代码然后简单的读取或者操作数据。

抓重点，**语言中立**， **工具**，**源代码**

### 使用
从上面的介绍中我们可以看到在使用 Protobuf 的使用需要有一个对应语言的工具，通过工具生成对应的源代码，然后在操作相应结构的数据。下面我们依次看下具体是如何使用的。
1. 首先我们这里采用的是 java 语言，所以要先去下载 Java 对应的工具通过链接[https://github.com/protocolbuffers/protobuf/releases/tag/v3.8.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.8.0)下载 Java 的工具，macOS 可以直接使用`brew install protobuf`

2. 编写 `.proto` 文件，在使用 Protobuf 前，我们要编写结构化的数据格式，例如我们这里编写 `com-test-model-User.proto` 文件

```
//指定版本
syntax = "proto2";
//定义包名
package com.test.model;

//定义结构数据
message User {
    //必选字段 第1个属性
    required string name = 1;
    //必选字段 第2个属性
    required int32 age = 2;
    //可选字段 第3个属性
    optional string comment = 3;
}

```
编写完了采用命令`protoc --java_out=. ./com-test-model-User.proto`，就会在当前路径下生成相应的代码结构。
![protobuf](http://www.justdojava.com/assets/images/2019/java/image_ziyou/protobuf1.jpg)

3. 使用案例

```
public class Test {

    public static void main(String[] args) throws IOException {
        UserOuterClass.User.Builder userBuilder = UserOuterClass.User.newBuilder();
        userBuilder.setName("子悠");
        userBuilder.setAge(18);
//        userBuilder.setComment("this is comment");

        System.out.println("\n**********************序列化*****************************");
        byte[] bytes = userBuilder.build().toByteArray();
        System.out.println("bytes length is " + bytes.length);
        for (int i = 0; i < bytes.length; i++) {
            System.out.print(bytes[i] + " ");
        }

        System.out.println("\n**********************反序列化*****************************");
        UserOuterClass.User user = UserOuterClass.User.parseFrom(bytes);
        System.out.println(user.getName());
    }
}



//运行结果

**********************序列化*****************************
bytes length is 10
10 6 -27 -83 -112 -26 -126 -96 16 18 
**********************反序列化*****************************
子悠

Process finished with exit code 0


```

4. 结果分析

可能到现在大家还没有发现什么优秀的地方，那么让我解释下。从运行结果来看，序列化后的是一串数字。很简短的一串数字。我们可以想一下如果这里用的 JSON 格式的序列化的话那么结果应该是`{"name": "子悠", "age": 18}`，如果是 xml 的话，那就会更长，从这里我们就可以看出 protobuf 的序列化的效果是多么的强大，效率是多么的高。我们知道在网络传输的过程中，压缩效率越高传输效率就越高。

```xml
 <user>
    <name>子悠</name>
    <age>18</age>
  </user>
```

5. protobuf 优缺点

- 更加简单
- 数据体积小 3- 10 倍
- 更快的反序列化速度，提高 20 - 100 倍
- 可以自动化生成更易于编码方式使用的数据访问类


### 小结

今天简单给大家介绍了 protobuf 的简单使用，更多的详细使用，以及底层压缩原理，感兴趣的朋友可以自己的研究一下。另外说个题外话 protocol buffers 诞生之初是为了解决服务器端新旧协议(高低版本)兼容性问题，名字也很体贴，“协议缓冲区”。只不过后期慢慢发展成用于传输数据。

有时候就是这样，一个项目或者软件的最终形态并不是当时定义的模样，随着时间的推移产品的方向以及定位都会发生翻天覆地的变化。

