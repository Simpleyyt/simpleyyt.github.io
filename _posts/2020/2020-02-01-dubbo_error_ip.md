---
layout: post
categories: Dubbo
title: 程序员鸭血粉丝又踩坑了，一次关于 Dubbo 服务 IP 注册错误的踩坑经历
tagline: by 小黑
tags: 
  - 小黑
---

Hello~各位读者新年好！我是鸭血粉丝，一位经常踩坑的程序员，所谓 `bug` 如风，常伴吾身，hasaki~

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200201/006tNbRwly1gbg1pren64j306y07tdho.jpg)

这不最近又遇到个问题，Dubbo 服务 IP 注册错误，好了，下面进入正题。

<!--more-->

## 踩坑

阿粉公司最近新建一个机房，需要将现有系统同步部署到新机房，部署完成之后，两地机房同时对提供服务。系统架构如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200201/006tNbRwly1gbg2sgilo9j30s40xf420.jpg)

这个系统当前对外采用 `Restful` 接口，内部远程采用 `Dubbo`，服务注册中心使用 `zookeeper`。服务当前设定只会调用本机房内服务。

原先服务都在 A 机房，B 机房为新建机房。B 机房部署完成之后，需要测试 B 机房系统可用性。生产测试的发现 B 机房竟然调用 A 机房服务。

> A/B 机房网络互相打通，可以互相访问

通过排查 B 机房服务日志，发现 `Service B` 一个服务节点注册 **IP** 解析错误，将 B 机房机器 **IP** 解析成 A 机房机器 **IP**。

于是当测试流量进入 B 机房时，`openapi`服务通过注册中心获取到错误的 `Service B` 服务地址，从而调用了 A 机房的服务。调用方式简化成如下图。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200201/006tNbRwly1gbg3ha21v4j30xf0gpq49.jpg)

> 知识点：Dubbo 服务提供者启动时将会将服务地址（**IP**+端口）注册到注册中心，消费者启动时将会通过注册中心获取服务提供者地址（**IP**+端口），后续服务调用将会直接通过服务地址直接调用。

## 问题分析

 **Debug** `Dubbo` 源码，定位到 `IP` 解析代码，位于 `ServiceConfig#findConfigedHosts`，源码如下：

> Dubbo 版本为 2.6.7

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200201/006tNbRwly1gbh04re3o7j30u011qu0y.jpg)

这个方法源码比较长，看起来比较费劲，不过好在这个方法注释上已经写明白 `IP` 地址查找顺序。

```java
Register & bind IP address for service provider, can be configured separately. Configuration priority: environment variables -> java system properties -> host property in config file -> /etc/hosts -> default network address -> first available network address
```

解析过程，`Dubbo` 将会过滤无用 **IP**，过滤规则如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200201/006tNbRwly1gbh6j7rloqj31220a87cy.jpg)

下面将结合图示讲解查找顺序，只要其中一步读取 **IP** 符合上述规则，方法就会返回。

第一步将会调用 `ServiceConfig#getValueFromConfig` 从 `environment variables` 或 `java system properties` 配置 **IP** 地址。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200201/006tNbRwly1gbh2v39hxnj317g0b2n70.jpg)

这种方式通过在 `JVM` 启动参数中显示指定 **IP** 。

```
-DDUBBO_IP_TO_BIND=1.2.3.4
```

第二步通过读取 `Dubbo` 配置文件配置变量获取 **IP**。

```xml
<!-- protocol 指定整个 Dubbo 应用服务默认 IP -->
<dubbo:protocol host="1.2.3.4"/>
<!-- provider 指定 Dubbo 应用具体某个服务默认 IP -->
<dubbo:provider host="1.2.3.4"/>
```

第三步通过调用 `InetAddress.getLocalHost().getHostAddress()` 获取本地 **IP**。该方法将会获取机器 `hostname`，然后再在 `/etc/hosts` 配置文件中查找 `hostname` 对应的配置 IP。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200201/006tNbRwly1gbh3leun6gj31c40n6x5f.jpg)

第四步通过 `socket` 连接注册中心从而获取本机 IP。

如果上述几步都不成功，Dubbo 将会轮询本机所有网卡，直到找到合适的 **IP** 地址。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200201/006tNbRwly1gbh39t5x2oj30ys0u07wh.jpg)

**问题原因**

通过排查上述几个规则，最后发现本地 `/etc/hosts` 文件 **IP** 配置错误， `hostname` 配置成了 A 机房的  **IP** 。

## 技术总结

Dubbo 在 **IP** 解析上花费很大功夫，最大程度上帮我们自动获取正确 **IP**。但是现实还是很残酷，真实环境下机器可能存在多网卡，内外网 **IP**，**VPN** ，或者应用采用 **Docker** 部署，这些情况下`Dubbo` 有可能就会获取到错误 **IP**，从而导致消费者调用失败。如果真遇到这种情况，读者首先通过上面顺序排查 **IP** 读取来源，若最后确定 **IP** 读取自网卡 。这种情况下就只能根据下面几种方式显示指定 IP。

配置方式一：

在 `JVM` 启动参数中加入如下配置

```java
-DDUBBO_IP_TO_BIND=1.2.3.4
```

配置方式二：

在 `/etc/hosts` 设置 `hostname` 对应的 **IP**。

配置方式三：

`Dubbo` 配置文件显示指定 IP。

```xml
<!-- protocol 指定整个 Dubbo 应用服务默认 IP -->
<dubbo:protocol host="1.2.3.4"/>
<!-- provider 指定 Dubbo 应用具体某个服务默认 IP -->
<dubbo:provider host="1.2.3.4"/>
```



## 帮助链接

https://dubbo.apache.org/zh-cn/blog/dubbo-network-interfaces.html

