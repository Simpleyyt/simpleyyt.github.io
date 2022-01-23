---
layout: post
categories: 分布式
title: 想知道注册中心原理吗？先来学习一下 ZooKeeper
tagline: by 小黑
tags: 
  - 小黑

---

Dubbo 通过注册中心在分布式环境中实现服务的注册与发现，而注册中心通常采用 ZooKeeper，研究注册中心相关源码绕不开 ZooKeeper，所以学习了 ZooKeeper 的基本概念以及相关 API 操作。

<!--more-->

## ZooKeeper 相关概念

### session

客户端与服务端采用 TCP 长连接，服务端在为客户端创建 Session 会分配一个唯一 sessionId。在 Session timeout 时间内，客户端可以向服务端发送请求以及接受 watcher 事件通知。

### 数据结构

Zookeeper 将所有数据存储在内存中，数据模型是一棵树（Znode Tree)，由斜杠（/）的进行分割的路径，就是一个Znode，例如/foo/path1。

![null](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190810/zknamespace.jpg)

### Znode

Znode 将会保存数据内容以及相关属性信息。在 Znode 中使用 Stat 数据结保存相关属性信息。Stat 属性中有三种版本信息，分别为 version：当前节点版本信息，cversion:当前节点子节点版本，aversion 当前节点的 ACL 版本。每次发生改动，版本数值将会单调递增。

更新，删除 Znode 可以传入版本数值，如果版本数值不对，将会导致删除/更新失败，这个特性类似于 CAS 操作。

Znode 有以下几种类型：

1. 永久节点

一旦创建，将会一直存在，除非手动删除。dubbo 目录节点为永久节点。

2. 临时节点

临时节点基于客户端 Session,Session 有效期内将会一直存在，Session 失效，节点将会自动删除。

利用这个机制，Dubbo 服务者创建的节点就是临时节点。如果 Dubbo 服务者程序意外宕机，在 Session 超时之后，也能自动删除服务节点，自动下线有问题的服务。

3 顺序节点

创建顺序节点将会自动在名字后追加整形数字，默认长度为 10 位。顺序节点也分为永久与临时。

利用临时顺序节点，我们可以用来实现分布式锁 [七张图彻底讲清楚ZooKeeper分布式锁的实现原理【石杉的架构笔记】](https://juejin.im/post/5c01532ef265da61362232ed)。

### Watcher 机制

客户端可以在指定节点注册监听器（Watcher），在触发特定事件后，ZooKeeper 服务端会将事件通知到客户端。在 Dubbo 中消费者基于 watcher 机制可以动态感知到新的服务者加入。

ZooKeeper 可以在三种请求中设置监听，分别为：

*   getData()，获取节点数据
*   getChildren() 获取子节点
*   exists() 判断节点是否存在

通知事件类型分为，增删改事件，以及子节点变动事件。

**需要注意的是，watcher 通知过一次之后将会失效，若想继续监听通知，需要重新注册。**

## ZooKeeper 原生 API 操作

ZooKeeper 官方提供 Java API 实现，提供相关操作的方法。

### 创建连接

```java

        ZooKeeper zk=new ZooKeeper("127.0.0.1:2181", 150000, new Watcher() {

            @Override
            public void process(WatchedEvent watchedEvent) {

                System.out.println("已经触发了" + watchedEvent.getType() + "事件"+watchedEvent);

            }

        });
```

创建连接需要传入 ZooKeeper 服务端地址，然后设定 session 超时时间，另外还需要创建一个 Watcher，用于监听连接事件。建立连接之后，就可以使用该客户端操作。

### CURD 操作

```java

        // 创建永久节点，需要传入 ACL 权限列表，以及指定节点类型
        zk.create("/test","test".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL,CreateMode.PERSISTENT);

        // 修改节点值。更新节点值需要传入节点的版本，如果版本与服务端版本不一致，更新失败，类似 CAS 机制。-1 代表不比较节点版本
        zk.setData("/test","test1".getBytes(),-1);
        // 删除节点.删除节点也需要传入节点版本
        zk.delete("/test",-1);
        // 创建临时节点
        zk.create("/ephemeral","ephemeral".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL,CreateMode.EPHEMERAL);

```

ZooKeeper 客户端相关 CRUD 操作如上。可以看到相关操作比较繁琐，需要传入参数较多。

### watcher

```java
 // 在 exists 注册 watcher，创建节点，删除节点，改变节点将会触发回调

        zk.exists("/test", new Watcher() {

            @Override
            public void process(WatchedEvent event) {
                System.out.println("回调实例，类型为："+event.getType());
            }
        });
        // 获取节点数据,可以注册 watcher,删除节点以及改变节点数据可以触发回调
        zk.getData("/test", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("回调实例，类型为："+event.getType());
            }
        },new Stat());

        // 获取子节点，注册 watcher，一级子节点变动后将会触发回调
        zk.getChildren("/test", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("回调实例，类型为："+event.getType());
            }
        });
```
ZooKeeper API 可以为三种操作注册 watcher，一旦相关节点变动将会触发事件通知。



## Curator

从上面代码示例可以看到 ZooKeeper 提供 API 比较复杂且难用。可以使用 Curator 或者 zkclient 这种第三方框架代替原生 API。这类框架封装 ZooKeeper 原生 API，抽象化相关接口，简化操作难度。

dubbo 抽象相关 ZooKeeper 操作，并分别使用  Curator 或者 zkclien 实现。在 dubbo 2.6.1 版本之后将会默认使用 Curator，之前版本默认使用 zkclient 。

下面我们使用 Curator  操作 ZooKeeper 。

### 建立连接

```java
 // 设置重试策略
 RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
 // 默认 session 超时时间 60 s
 CuratorFramework client = CuratorFrameworkFactory.newClient("127.0.0.1:2181", retryPolicy);
```
Curator  创建连接与原生 API 大致相关，不过需要设置重试策略，第一次连接失败，Curator  可以重新尝试连接，直到超过最大连接次数。

### 节点 CURD 操作

```java
        // 创建目录节点

        client.create().forPath("/test", "123456789".getBytes());
        // 创建普通节点
        client.create().forPath("/test/normal", "123121".getBytes());
        // 修改普通节点内容
        client.setData().forPath("/test/normal", "1121231231".getBytes());
        // 删除节点
        client.delete().forPath("/test/normal");

        // 创建临时节点
        client.create().withMode(CreateMode.EPHEMERAL).forPath("/ephemeral");
        // 创建永久顺序节点
        client.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath("/sequential");
        
       // 获取所有子节点
        List<String> nodes = client.getChildren().forPath("/parent");
```

Curator CRUD 操作比较简单，无需设置相关属性参数。

### 设置监听

Curator 相关监听 API 封装 zookeeper 原生API，内部增加重复注册等功能，从而使监听可以重复使用。

Curator 存在三种类型 API。

* `NodeCache`:针对节点增删改操作。
* `PathChildrenCache`:针对节点一级目录下节点增删改监听
* `TreeCache`：结合 `NodeCache` 与 `PathChildrenCache` 操作，不仅可以监听当前节点，还可以监听节点下任意子节点（支持多级）变动。

```java
	//   `NodeCache`使用方式
        NodeCache nodeCache=new NodeCache(client,"/test1",false);
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                System.out.println("当前节点："+nodeCache.getCurrentData());

            }
        });
        nodeCache.start();

	// PathChildrenCache 使用方式
	 PathChildrenCache pathChildrenCache=new PathChildrenCache(client,"/test2",false);
        pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework client, PathChildrenCacheEvent event) throws Exception {
                System.out.println(event);
            }
        });
        pathChildrenCache.start(PathChildrenCache.StartMode.BUILD_INITIAL_CACHE);
        System.out.println("注册watcher成功...");      
	// TreeCache 使用方式
        TreeCache treeCache=new TreeCache(client,"/tree");
        treeCache.getListenable().addListener(new TreeCacheListener() {
            @Override
            public void childEvent(CuratorFramework client, TreeCacheEvent event) throws Exception {
                                System.out.println(event);
            }
        });
        treeCache.start();
        System.out.println("注册watcher成功...");
 
```

