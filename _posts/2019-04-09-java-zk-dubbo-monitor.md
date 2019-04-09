---
layout: post
category: java
title: 基于ZK的 Dubbo-admin 与 Dubbo-monitor 搭建
tagline: by 子悠
tags: 
  - java
published: true
---

### 背景
最近项目中使用了 dubbo 在实现服务注册和发现，需要实现对服务提供者和调用者的监控，之前有研究过基于 redis作为注册中心的监控平台，不过本文基于 zk 作为注册中心，进行 dubbo-admin 和 dubbo-monitor 搭建。另外项目基于 dubbo 2.6.4版本，所以该监控版本调整为 dubbo2.6.4。
 
 <!--more-->
 
### 步骤
1. [GitHub](https://github.com/apache/incubator-dubbo-ops/tree/master)
2. 官方组件目前在重构，采用前后分离技术，尚未完成。本文采用的还是 master 分支的老版本 dubbo-admin
3. `git clone https://github.com/apache/incubator-dubbo-ops`
4. 将项目根目录下的 pom.xml文件中的 dubbo 版本调整为2.6.4 `<dubbo_all_version>2.6.4</dubbo_all_version>`
5. 将 dubbo-admin 项目下的 pom.xml文件中的 dubbo版本进行调整，并且增加 netty 依赖

```

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.6.4</version>
</dependency>
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.30.Final</version>
</dependency>  


```

6. 修改dubbo-admin 项目中 resources 目录下的 applicatio.properties文件

```
server.port=7001
spring.velocity.cache=false
spring.velocity.charset=UTF-8
spring.velocity.layout-url=/templates/default.vm
spring.messages.fallback-to-system-locale=false
spring.messages.basename=i18n/message
# root 用户登录账户和密码
spring.root.password=root
# guest 用户登录账户和密码
spring.guest.password=guest
# zk 注册中心地址，可以配置单个或者多个
dubbo.registry.address=zookeeper://ip1:2181?backup=ip2:2181,ip3:2181
# 配置 zk 中根目录文件夹，不配默认为 dubbo，并非生产者和消费的的分组
dubbo.registry.group=ad-dubbo

```

7. 修改dubbo-admin 项目中 resources 目录下的 dubbo-admin.xml文件，如果没有配置 group 可以不用加 group 配置

```

<dubbo:registry address="${dubbo.registry.address}" group="${dubbo.registry.group}" check="false" file="false"/>

```

8. 此条如果配置了 provider 和 consumer 的分组时采用，如果没有则跳过。在 dubbo-admin 项目中创建`com.alibaba.dubbo.common`包，并新建 `URL.java`类，复制 dubbo2.6.4中的 URL.java，按照图片修改

![](/assets/images/2019/java/image_ziyou/dubbo1.jpg)

> 说明：如果不这样修改，在管理后台会有问题，官方 [issue](https://github.com/apache/incubator-dubbo-ops/issues/61)

![](/assets/images/2019/java/image_ziyou/dubbo2.jpg)

9. 在根目录下执行`mvn clean package`

10. 启动 dubbo-admin，执行`java -jar dubbo-admin/target/dubbo-admin-0.0.1-SNAPSHOT.jar`

11. 打开http://127.0.0.1:7071即可看到界面，登录账户和密码为 applicatio.properties中配置的默认root/root,guest/guest


### dubbo-monitor搭建
1. 修改 dubbo-monitor-simple 项目 resources/conf 目录下的dubbo.properties

 ```
 
dubbo.container=log4j,spring,registry,jetty-monitor
dubbo.application.name=dubbo-admin-monitor
dubbo.application.owner=dubbo
# zk 注册中心地址同 admin 
dubbo.registry.address=zookeeper://1p1:2181?backup=ip2:2181,ip3:2181
dubbo.protocol.port=7070
dubbo.jetty.port=7002
dubbo.monitor.queue=1000
# 统计数据和图表的生产路径，需要手动提前创建
dubbo.jetty.directory=/opt/monitor
# charts 目录，必须放到 dubbo.jetty.directory目录下
dubbo.charts.directory=${dubbo.jetty.directory}/charts
# statistics 目录，必须放到 dubbo.jetty.directory目录下
dubbo.statistics.directory=${dubbo.jetty.directory}/statistics
dubbo.log4j.file=logs/dubbo-monitor-simple.log
dubbo.log4j.level=WARN
# zk 注册中心的根目录，不配置默认为 dubbo
dubbo.registry.group=ad-dubbo
    
 ```
 
2. 在根目录下执行`mvn clean package`，在`dubbo-monitor-simple/target`目录下的`dubbo-monitor-simple-2.0.0-assembly.tar.gz`拷贝到其他目录，或者本目录，解压并启动

```

tar xzvf dubbo-monitor-simple-2.0.0-assembly.tar.gz 
cd dubbo-monitor-simple-2.0.0
./assembly.bin/server.sh start

```
3. 打开http://127.0.0.1:7002

![](/assets/images/2019/java/image_ziyou/dubbo3.jpg)

配置生产者和消费者，进行调用就会显示类似下面的图，这里的图修改图片生成的时间为1分钟

![](/assets/images/2019/java/image_ziyou/dubbo4.jpg)

4. 注意事项
    - `dubbo.monitor.queue`：监测队列大小，默认为100000
    - chart 图片默认五分钟根据统计目录的数据生成一张 png 图片，在SimpleMonitorService.java 中110行，可以修改自己需要的时间间隔，
    ![](/assets/images/2019/java/image_ziyou/dubbo5.jpg)
    - `dubbo.jetty.directory=/opt/monitor`这个路径必须自己手动提前创建，否则无法自动创建统计和图片目录，导致没有图片显示
    - 如果没有出现图片，在生产者和消费者的配置中添加监控地址`dubbo.monitor.address= 172.20.155.60:7070`
    
```

# 配置文件
dubbo.registry.group = ad-dubbo
dubbo.monitor.address= 172.20.155.60:7070

# 基于 SpringBoot 增加配置
@Bean
public MonitorConfig monitorConfig() {
    MonitorConfig config = new MonitorConfig();
    config.setAddress(monitorAddress);
    config.setProtocol("registry");
    return config;
}

```

### 小结
经过几天的时间修改了部分源码搭建起来的监控平台，目前已经在生产上使用了。从目前的服务调用来看，效果还可以。前期搭建的过程中还是踩了不少坑的，由于 dubbo 的版本复杂，网上多种监控平台各有各的特点，
权衡了很久才决定还是用官方的监控平台。官方的监控平台是使用人数最多的，既然是要上生产，就必须要有保障。但是可能由于年代久远，不是很美观，不过能用。网上还有其他版本的监控平台比如开源工具 Dubbokeeper，
以及开源工具 DubboMonitor-x，这几个在之前研究的时候采用 Redis 作为注册中心的时候有研究过，但是后面选择 zk 和官方的监控平台就没有折腾了。

