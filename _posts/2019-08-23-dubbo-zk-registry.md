---
layout: post
category: Java
title: 不知道如何实现服务的动态发现？快来看看 Dubbo 是如何做到的
tagline: by 小黑
tags: 
  - Dubbo,Zookeeper
published: true
---


上篇文章[如果有人问你 Dubbo 中注册中心工作原理，就把这篇文章给他](http://www.justdojava.com/2019/08/17/dubbo-registry-learn/)大致了解了注册中心作用以及 Dubbo Registry 模块源码，这篇文章将深入 Dubbo ZooKeeper 模块，去了解如何实现服务动态的发现。

<!--more-->

> ps: 以下将 ZooKeeper  缩写为 zk。

## dubbo zk 数据结构

在 [ZooKeeper 基本概念分享](http://www.justdojava.com/2019/08/10/zk-baisic-knowledge/)一文讲道，ZK 内部是一种树形层次结构，节点存在多种类型。而 Dubbo 只会创建持久节点和临时节点。

若服务提供者服务接口为 `com.service.FooService`，将会在 ZK 中创建创建如下路径 `/dubbo/com.service.FooService/providers/providerURL`。

服务路径分为四层，根节点默认为 dubbo，可以在 dubbo-registry 设置 group 属性改变该值。

>**ps: 若无注册中心隔离需求，不要随便修改。**

第二层节点为服务节点全名称，如  `com.service.FooService`。

第三层节点为服务目录，如 providers。另外还存在其他目录节点，分别为 consumers（消费者目录），configurators（配置目录），routers（路由目录）。下面服务订阅主要针对这一层节点。

第四个节点为具体服务节点，节点名为具体的 URL 字符串，如 `dubbo://2.0.1.13:12345/com.dubbo.example.DemoService?xx=xx` ，该节点默认为临时节点。
dubbo ZK 树形内部结构示例为：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/image-5ae692eb.png)

ZK 内部服务具体示例如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/Snipaste20190811170204-8ccbf03e.png)

## `RegistryFactory` 实现

Dubbo 可以在配置文件中指定使用注册中心，可以使用 `dubbo.registry.protocol` 指定具体注册中心类型，也可以设置 `dubbo.registry.address` 指定。注册中心相关实现将会使用 `RegistryFactory` 工厂类创建。

*`RegistryFactory ` 接口源码如下:*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/RegistryFactory-558fb540.png)

`RegistryFactory` 接口方法使用 `@Adaptive` 注解，这里将会使用 Dubbo SPI 机制，自动生成代码的一些实现逻辑。这里将会根据 URL 中 `protocol` 属性，去调用最终实现子类。

*`RegistryFactory` 实现子类如图所示：*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/RegistryFactory-b88babc7.png)

`AbstractRegistryFactory` 将会实现接口的 `getRegistry` 方法，主要完成加锁，并调用抽象模板方法 `createRegistry` 创建具体注册中心实现类，并将其缓存在内存中。

*`AbstractRegistryFactory#getRegistry` 源码如下所示：*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/getRegistry-9462b1e0.png)

注册中心实例将会通过具体工厂类创建，这里我们看下 `ZookeeperRegistryFactory` 源码：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/ZookeeperRegistryFactory-b95ee536.png)

ps：Dubbo SPI 机制还具有 IOC 特性，这里的`ZookeeperTransporter` 注入可以参考：[Dubbo 扩展点加载](http://dubbo.apache.org/zh-cn/docs/dev/SPI.html)

## zk 模块源码解析

讲完注册中心实例创建过程，下面深入 `ZookeeperRegistry` 实现源码。

`ZookeeperRegistry` 继承 `FailbackRegistry`抽象类，所以其需要实现其父类抽象模板方法，下面主要了解 `doRegister` 与 `doSubscribe`源码 。

### `doRegister`

服务提供者需要将服务注册到注册中心，注册的目的是为了让消费者感知到服务的存在，从而发起远程调用，另一方面也让服务治理中心感知新的服务提供者上线。zk 模块服务注册代码比较简单，直接使用 zk 客户端在注册中心创建节点。

*`ZookeeperRegistry#doRegister` 实现源码如下：*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/doRegister-d41ddbb5.png)

`zkClient.create` 方法需要传入两个参数。

```java
void create(String path, boolean ephemeral);
```

第一个参数为节点路径，将会通过 `toUrlPath` 将 URL 实例转化成 ZK 中路径格式，转化结果如下：

```

## 转化前 URL 如下：

dubbo://10.20.82.31:12345/com.dubbo.example.DemoService

## 调用  `toUrlPath`  转换之后
/dubbo/com.dubbo.example.DemoService/providers/dubbo%3A%2F%2F10.20.82.31%3A12345%2Fcom.dubbo.example.DemoService

```

第二个参数主要决定 ZK 节点类型主要取自 URL 实例对象中 `dynamic` 参数值，若不存在，默认为 `true`，也就是默认将会创建临时节点。

`zkClient.create` 方法里将会递归调用，首先父节点是否存在，不存在就会创建，直到最后一个节点跳出递归方法。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/AbstractZookeeperClientcreate-d2310cd3.png)

最后 `createEphemeral` 与 `createPersistent` 实际创建节点操作将会交给 ZK 客户端类，这里实现比较简单，可以自行参考源码。

> ps: dubbo 在 2.6.1 起将 zk  客户端默认使用 Curator，之前版本使用 zkclient。dubbo 2.7.1 开始去除 zkclient 实现，也就是说只能使用 Curator 。

### 为何 dubbo 服务提供者节点使用 zk 临时节点

zk 临时节点将会在 zk 客户端断开后，自动删除。dubbo 服务提供者正常下线，其会主动删除 zk 服务节点。

如果服务异常宕机，zk 服务节点就不能正常删除，这就导致失效的服务一直存在 ZK 上，消费者还会调用该失效节点，导致消费者报错。通过 zk 临时节点特性，让 zk 服务端主动删除失效节点，从而下线失效服务。

## `doSubscribe`: 服务动态发现的原理

**订阅基本原理**

服务订阅通常有 pull 和 push 两种方式。pull 模式需要客户端定时向注册中心拉取配置，而 push 模式采用注册中心主动推送数据给客户端。

dubbo zk 注册中心采用是事件通知与客户端拉取方式。服务第一次订阅的时候将会拉取对应目录下全量数据，然后在订阅的节点注册一个 watcher。一旦目录节点下发生任何数据变化，zk 将会通过 watcher 通知客户端。客户端接到通知，将会重新拉取该目录下全量数据，并重新注册 watcher。利用这个模式，dubbo 服务就可以就做到服务的动态发现。

**源码解析**

讲完订阅的基本原理，接着深入源码。

`doSubscribe` 方法需要传入两个参数，一个为 URL 实例，另一个为 `NotifyListener`，变更事件的监听器。 方法内部会根据 URL 接口类型分成两部分逻辑，全量订阅服务与部分类别订阅服务。

*`doSubscribe` 方法整体源码逻辑：*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/doSubscribe-1d1bcd2f.png)

服务治理中心(dubbo-admin)，需要订阅 service 全量接口，用以感知每个服务的状态，所以订阅之前将会把 service 设置成 *，处理所有service。

服务消费者或服务提供者将会走部分类别订阅服务，下面我们以消费者视角，深入后续源码。

文章刚开头讲道了 zk 目录节点存在四种类型，这里将会根据 根据 URL 中 `category`值，决定订阅节点路径。

服务提供者 URL 中 `category`值默认为 `configurators`，而消费者 URL 中`category`值默认为 `providers,configurators,routers`。如果 `category`类别值为 `*`，将会订阅四种类别路径，否则将会只订阅 `providers`类型的路径。

*`toCategoriesPath`  源码如下：*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/toCategoriesPath-866ae26e.png)

接着循环路径数组，循环内将会缓存节点监听器，用以提高性能。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/类别订阅服务-f462befb.png)

最终将会在 ZK 目录节点上注册 watcher，并获取目录节点下所有子节点数据。

`ZookeeperClient#addChildListener 源码如下`

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/订阅-b9b2a82b.png)

这里 watcher 使用 Curator 接口 `CuratorWatcher`，一旦 ZK 节点发生会变化，将会回调 `CuratorWatcher#process` 方法。

*`CuratorWatcher#process` 方法源码如下：*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/CuratorWatcherprocess-85cbd2f3.png)

*消费者订阅时序图如下：*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/dubbo订阅1-6a4ab156.png)

### listener 关系图

订阅方法中我们碰到了多个 `listener`类，刚开始理解时候可能有点乱。可以参考下面关系图理清楚这其中的关系。

`listener` 关系图如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/dubbodoSubscribelistener关系图1-614939d8.png)

回调关系如图所示：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/listener回调关系3-0457ed49.png)

### ZK 模块订阅存在问题

ZK 第一次订阅将会获得目录节点下所有子节点，后续任意子节点变更，将会通过 watcher 进制回调通知。回调通知将会再次全量拉取节点目录下所有子节点。这样全量拉取将会有个局限，当服务节点较多时将会对网络造成很大的压力。

Dubbo 2.7 之后版本引入元数据中心解决该问题，详情可参考，[阿里技术专家详解 Dubbo 实践，演进及未来规划](https://mp.weixin.qq.com/s/9rVGHYfeE8yM2qkSVd2yEQ)。

*引用文中一种解决方案如下图:*

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/e2a0160ed30947eb93f321877b1005cf-85a96178.png)

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190823/ef9d4e895a454669ab3a5d878c76b5ab-2350cd57.png)

## 总结

本文主要介绍了 dubbo zk 的数据结构，其次深入研究 `ZookeeperRegistry` 相关实现源码。通过了解服务注册以及订阅原理，了解 Dubbo 服务动态发现实现方式。

