---
layout: post
category: http
title: Http 和 Socket 到底是哪门子亲戚？
tagline: by 江南一点雨
tags: 
  - http
  - socket
published: true

---

一些刚入门的小伙伴可能会用 Socket，也会用 OkHttp 或者 HttpUrlConnection 等一些 HTTP 客户端工具，这两个东西看着有点像可是又不太一样，到底是哪里不一样呢？好像又说不出来，那么今天我希望能够帮助大家理解这两个东西。

<!--more-->

# Http 与 Socket

我们先来看一张经典图：  

![p260](http://www.justdojava.com/assets/images/2019/java/image_javaboy/http/3.jpg)  

HTTP(HyperText Transfer Protocol) 即超文本传输协议，它是基于 TCP/IP 协议之上的应用层协议，TCP/IP 属于传输层协议，主要用来解决数据如何在网络中进行传输，而 HTTP 属于应用层协议，主要用来解决数据如何包装，在实际开发中，有的公司会在 C/S 结构的项目中使用自定义协议，一般自定义协议就是指自定义应用层协议。就像我从深圳向广州寄一件快递，HTTP 协议负责物品如何包装以及到达目的地之后如何拆箱，而 TCP/IP 协议就是快递公司，负责将东西从深圳运送到广州，可能中途还会经过 N 个中转站，这些都由 TCP/IP 协议去负责。  

我们在做数据传输的时候，甚至可以只使用 TCP/IP 协议，但是这样会没有应用层，没有应用层，我们就不能有效识别出数据内容，所以我们还是需要应用层协议，根据实际需求，我们可以选择不同的应用层协议，比如 HTTP、FTP 等。  

Socket 则是对 TCP/IP 协议的封装，它就是一个调用接口，通过调用 Socket，我们就可以使用 TCP/IP 协议，TCP/IP 协议只是一个协议栈，想要让程序员能够使用它，就必须提供可以供程序员使用的接口，这个接口就是 Socket ，在我们充分了解了 HTTP 协议的数据格式之后，我们也可以利用 Socket 来模拟 HTTP 请求。  

网上有一个形象的描述，说 HTTP 就是一部轿车，提供了数据的封装形式，Socket 则是发动机，提供了基本的网络通信能力。  

好了，不知道小伙伴现在有没有搞清这两个之间的关系呢？

搞清楚这个问题之后，我们再来顺便聊一聊 Http 的报文结构。

# Http 报文

## 请求报文

HTTP 的请求信息由四部分组成，分别是请求行、请求头、空行和请求数据，如下：  

![p261](http://www.justdojava.com/assets/images/2019/java/image_javaboy/http/4.jpg)  

1. 请求行主要包含了三部分信息，请求方法、请求 URI 以及 HTTP 的版本  
2. 请求头中主要包含了请求的各种条件  
3. 空行 CR+LF  
4. 请求数据  

## 响应报文

HTTP 响应报文也由四部分组成，分别是状态行、响应头、空行以及响应正文，如下：  

![p262](http://www.justdojava.com/assets/images/2019/java/image_javaboy/http/5.jpg)  

1. 状态行包含三部分内容，分别是 HTTP 版本、状态码和原因短语  
2. 响应头信息  
3. 空行  
4. 响应数据  

## HTTP 请求方法

请求方法除了常见的 GET、POST 之外，在移动互联网时代，PUT、DELETE 等方法也得以大展拳脚，HTTP 中的主要方法如下：  

![p264](http://www.justdojava.com/assets/images/2019/java/image_javaboy/http/6.jpg)  

---以上表格来自《网络是怎样连接的》一书

## HTTP 头信息

无论是请求报文还是响应报文，都涉及到 HTTP 头，HTTP 头信息一般来说可以分为四大类，分别是通用头、请求头、响应头和实体头，如下：  

![p263](http://www.justdojava.com/assets/images/2019/java/image_javaboy/http/7.jpg)  

---以上表格来自《网络是怎样连接的》一书

OK，搞清楚了HTTP的数据格式，接下来我们就可以用Socket模拟一个HTTP请求了。敬请关注下篇文章。