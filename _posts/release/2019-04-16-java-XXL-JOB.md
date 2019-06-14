---
layout: post
category: java
title: 你应该知道的任务调度平台 XXL-JOB
tagline: by 子悠
tags: 
  - java
published: true
---

### 背景
日常开发中，我们难免会遇到需要处理一些定时任务，而且这些定时任务还需要灵活的调度，并且在异常的情况下需要做的重试或者报警。这些任务我们希望能灵活配置，并且能及时生效，不需要经常发版本更新代码。
所以我们希望能有一个这样的平台，能满足我们的这些需求。感谢开源社区，已经有了很好的解决方案，就是 XXL-JOB。
本文介绍的版本是基于 XXL-JOB的1.9.0版本，新版本调度中心 Admin 已经切换为 SpringBoot 项目了。
 
 <!--more-->
 
#### 介绍
[XXL-JOB](https://github.com/xuxueli/xxl-job)是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。
[中文文档地址](http://www.xuxueli.com/xxl-job/#/)

XXL-JOB由两个模块组成分为**调度中心**和**执行器**，作者许雪里的开源项目，感谢大佬。

#### 调度中心搭建
从[release](https://github.com/xuxueli/xxl-job/releases)拉取最新代码

根据自己的需要配置xxl-job-admin中xxl-job-admin.properties文件中的数据源信息以及账号密码，以及accessToken和邮件服务器地址等信息
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/xxl-job1.png)

配置log4j.xml中日志的路径
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/xxl-job2.png)

将xxl-job-admin打包成war包，部署到tomcat中即可

#### 执行器配置
新建Springboot项目，pom.xml中引入xxl-job的核心库

```

<dependency>
       <groupId>com.xuxueli</groupId> 
       <artifactId>xxl-job-core</artifactId>
       <version>1.9.0</version>
</dependency>

```

配置文件中配置调度中心的地址和一些具体参数
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/xxl-job3.png)

编写jobHandler，继承IJobHandler实现内部execute方法，具体的业务逻辑就在这个方法里面实现。这种方式是通过 Java 代码来执行定时任务的，除了 JavaBean 方式还支持 Python，nodeJs，Shell 等方式。
我最喜欢的是 Python 方式，因为 Python 在处理简单的定时任务的时候还是比较得心应手的，而且很快速，但是稍微复杂一点的就不方便了，而且 Python 可以直接在WebIDE 里面直接粘贴代码，实现功能，就不用发版本了，但是具体的需要看具体的业务。

```

@Service
@JobHandler(value = "demoHandler")
public class DemoHandler extends IJobHandler {
    @Override
    public ReturnT<String> execute(String s) throws Exception {
        XxlJobLogger.log("日志记录数据...");
        //do something
        return ReturnT.SUCCESS;
    }
}

```

#### 新增任务
配置好调度中心并且也成功启动了执行器后，登录调度中心新增执行器然后就可以配置任务了

新增执行器
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/xxl-job4.png)

新增任务
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/xxl-job5.png)
> 新增任务的时候需要选择上一步创建的执行器，选择运行模式，如果是 JavaBean 方式就配置JobHandler，或者选择 Python 模式等，然后填上必要的一些信息，如Cron以及一些参数
> 具体配置可以参考[三、任务详解](http://www.xuxueli.com/xxl-job/#/?id=%E4%B8%89%E3%80%81%E4%BB%BB%E5%8A%A1%E8%AF%A6%E8%A7%A3)
这里的配置项比较多。

配置完成后可以如下，可以手动执行，暂停，查看日志，如果是Python 模式可以直接点击`GLUE`按钮进去填写代码，相关的代码也有版本回溯，方便回滚，十分方便。
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/xxl-job6.png)


### 小结
整个 XXL—JOB 都是支持分布式部署的，而且执行器也可以配置多个，具体的路由策略也是可以配置的，十分方便。新版本的调度中心 Admin 已经升级为 SpringBoot 项目了，而且项目本身就携带了很多类型的执行器可以使用，基本上实现了开箱即用，只要添加自己的业务逻辑就可以了。

不过整个 XXL-JOB 有个缺点就是没有权限管理，执行器没有实现权限控制，这个目前作者没有实现，应该在后续的版本中会增加，这个在作者的 todo list 里面，但是具体什么时候实现就不知道了。目前我们这边项目已经在线上跑了差不多一年了，还是比较稳定的，而且只是组内使用，没有执行器权限的问题，但是如果要是上升到公司级别，让各个组用的话权限还是一个重要的点。
