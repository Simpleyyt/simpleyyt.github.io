---
layout: post
categories: Spring
title: 定时任务莫名停止，Spring 定时任务存在 Bug？？？
tagline: by 小黑
tags: 
  - 小黑
---

Hello~各位读者新年好，我是鸭血粉丝（大家可以称呼我为「阿粉」）。这里阿粉给大家拜个年，祝大家蒸蒸日上烫烫烫，年年有余屯屯屯。
<!--more-->

## 那年那 Bug

春节放假，阿粉坐上高铁回家，路上阿粉突然想到一次生产问题。那是阿粉参加工作第一年，那一年国庆假期，阿粉提前一天请假回家办个护照。那时候阿粉刚开始负责的系统，所以工作日请假，还是有点担心，就怕**问题**看阿粉不在，悄然上门。

哎，真实越怕什么，就来什么。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb6sau53e9j303603cjr6.jpg)

高铁开到一半的时候，同事反馈系统不能获取最新的流水信息（流水信息通过 `Spring` 定时任务定时拉取）。阿粉心里一惊，立刻拔出电脑，连上 **VPN**，准备登上生产机器，查看系统情况。可是，高铁上网络大家也懂，很不稳定，连了好久连不上 **VPN**，只好远程指挥同事看一下系统日志。通过同事反馈的日志，发现拉取流水定时任务没有执行，进一步查看，阿粉发现整个系统其他的定时任务也都停止了。。。

这真是一个奇怪的的问题，这好端端的定时任务怎么会突然停止？

暂时想不到解决办法，只好指挥同事先重启应用。重启之后，暂时解决问题，定时任务重新开始执行，也获取到最新的付款流水信息。

## 问题排查

到家之后，阿粉立刻登上生产机器，查看系统日志。这里阿粉发现重启之前某一定时任务运行到一半，并且在这之后其他定时任务就没有再被执行。

通过系统日志，定位到了有问题的代码。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb7i09cbp4j318i0lq78t.jpg)

这里采用**重试补偿策略**，防止查询流水信息因为网络等问题发生偶发的失败。这个策略面对偶发的失败没什么问题，但是如果查询银行流水服务一直失败，这段代码就会陷入死循环。恰巧那段时间网络出现一些问题，导致这里查询一直处于失败。

增加最大重试次数，修复该 `Bug`。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb7igk1gpsj318i0mqaep.jpg)

修复之后，立刻将最新版本代码部署到生产系统，暂时解决了这个问题。

> 知识点：面对一些失败，可以采用重试补偿策略，重新执行，最大可能保证执行成功，但是这里切记设置合适的的**重大的次数**。

## 深入排查

虽然问题解决了，但是阿粉心里还是存在一个疑惑，为何一个定时任务发生了阻塞，就会影响执行其他定时任务。阿粉最初的理解是不同的定时任务应该互相隔离，互不影响才对，真难到是 `Spring` 定时任务的 **Bug** 吗？

想到这里，阿粉决定写一个 **Demo**，复现问题，然后深入源码排查。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb7pfukmduj30u00ugdrz.jpg)

启动程序，日志输出如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb7pmjuahmj32cw05sac6.jpg)

从日志可以看到，`fixDelayMethod` 方法执行之后进入休眠，直到休眠结束，`cronMethod` 定时任务才有机会被执行。另外从上面可以看到，上述两个定时任务都由 `pool-1-thread-1`线程执行。从这点可以看出 `Spring` 定时任务将会交给线程池执行。

> 知识点: 线程池中线程默认命名策略为 pool-%poolNumber-thread-%num。

如果线程池只有一个工作线程，该线程一旦被长时间阻塞，堆积的其他任务就没有机会被执行。

那么是不是这个问题导致的 `Sping` 定时任务停止执行？我们继续往下排查。

图上日志绿色部分， `ScheduledAnnotationBeanPostProcessor` 输出一个重要信息：

```log
No TaskScheduler/ScheduledExecutorService bean found for scheduled processing
```

查看 [`Spring` 文档](https://docs.spring.io/spring/docs/4.3.26.RELEASE/spring-framework-reference/htmlsingle/#scheduling-task-scheduler-implementations)，`Spring` 内部将会通过调用 `TaskScheduler` 执行定时任务，而另一个 `ScheduledExecutorService` 为 `JDK` 提供执行定时任务的执行器。记住这两者

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb8rv9d35pj31ic0hmgoh.jpg)

通过这段日志，使用 IDEA 的强大的**关键字搜索功能**，定位到 `ScheduledAnnotationBeanPostProcessor#finishRegistration` 方法。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb7rg2fug4j30u012vkde.jpg)

这个方法比较长，大家重点关注图中标示的几处。

`Spring` 启动之后将会扫描所有 `Bean` 中带有 `@Scheduled` 注解的方法，然后封装成 `Task` 子类放置到 `ScheduledTaskRegistrar`。

> 这段代码位于 `ScheduledAnnotationBeanPostProcessor#processScheduled`，感兴趣的可以翻阅查看

如果此时 `ScheduledTaskRegistrar` 不存在定时任务或者 `ScheduledTaskRegistrar` 中的 `TaskScheduler`不存在，`finishRegistration`将会多次调用 `ScheduledAnnotationBeanPostProcessor#resolveSchedulerBean` 方法用以查找 `TaskScheduler/ScheduledExecutorService`。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb8se85rkpj31po0qg12a.jpg)

接下去将会把获取到 `Bean` 通过 `setScheduler` 注入到 `ScheduledTaskRegistrar` 中。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb8smi1pphj31l00jmdma.jpg)

如果获取的为 `ScheduledExecutorService` 类型，将会将其封装到 `taskScheduler`中。

最后还没找到，将会输出最刚开始见到的日志。然后 `Spirng` 将会在 `ScheduledTaskRegistrar#afterPropertiesSet` 创建一个单线程的定时任务执行器 `ScheduledExecutorService`，注入到 `ConcurrentTaskScheduler`中，然后通过 `taskScheduler` 执行定时任务。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb7rpnzcjxj30vf0u0ahh.jpg)

![image-20200125144040781](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb8swduasij31fk0u048r.jpg) 

交给`TaskScheduler` 的定时任务最后实际上还是通过 `ScheduledExecutorService`执行。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb8sz5wx5gj327a0c6aep.jpg)

这里可以得出一个**结论**：

`Spring` 定时任务实际上通过 `JDK` 提供的 `ScheduledExecutorService`执行。默认情况下，Spring 将会生成一个单线程`ScheduledExecutorService`执行定时任务。所以一旦某一个定时任务长时间阻塞这个执行线程，其他定时任务都将被影响，没有机会被执行线程执行。

Spring 这种默认配置，在需要执行多个定时任务的情况，可能会是一个坑。我们可以通过改变配置，使 Spring 采用多线程执行定时任务。

## 自定义配置

Spring 可以通过多种方式改变默认配置。

### xml 配置

通过 `xml` 配置 `TaskScheduler` 线程数。

```xml
<task:annotation-driven  scheduler="myScheduler"/>
<task:scheduler id="myScheduler" pool-size="10"/>
```

通过上面的配置，Spring 将会使用 `TaskScheduler` 子类 `ThreadPoolTaskScheduler`，内部线程数为 `pool-size` 数量，这个线程数将会直接设置 `ScheduledExecutorService` 线程数量。

### 注解配置

在上面问题排查中，我们知道 `Spring` 将会查找 `TaskScheduler/ScheduledExecutorService`,若存在将会使用。所以这里我们可以生成这些类的 `Bean`。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200125/006tNbRwly1gb8tuvqie0j313e0rqdma.jpg)

> 以上方式二选一即可

### SpringBoot 配置

上面两种配置适用于普通 `Spring`，比较繁琐。相比而言 `SpringBoot` 配置将会非常简单,只需要在启动配置文件加入如下配置即可。

```properties
spring.task.scheduling.pool.size=10
spring.task.scheduling.thread-name-prefix=task-test
```

## 技术总结

下面开始技术总结：

1. `Spring` 定时任务执行原理实际使用的是 `JDK` 自带的 `ScheduledExecutorService`
2. `Spring` 默认配置下，将会使用具有单线程的 `ScheduledExecutorService`
3. 单线程执行定时任务，如果某一个定时任务执行时间较长，将会影响其他定时任务执行
4. 如果存在多个定时任务，为了保证定时任务执行时间的准确性，可以修改默认配置，使其使用多线程执行定时任务
5. 面对偶发的失败，我们可以采用重试补偿策略，不过这里切记设置合适的最大重试次数

## 随便聊聊

对于常用的开源框架，我们不仅要掌握怎么用，还要熟悉相关的配置，最后还应该去了解其内部的使用的原理。这样出了问题，我们也能很快定位问题，找到问题的实际原因。

## 帮助文档

[Spring scheduling-task-scheduler ](https://docs.spring.io/spring/docs/4.3.26.RELEASE/spring-framework-reference/htmlsingle/#scheduling-task-scheduler-implementations)
