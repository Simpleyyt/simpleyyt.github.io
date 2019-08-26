---
layout: post
title: Spring 系列之定时任务—— Scheduled
tagline: by 子悠
categories: Spring
tags: 
  - Java,Spring
---

**人生有涯，学海无涯**

Spring 的定时任务想必大家多多少少都用过，经过 Spring 团队的封装，大家使用起来非常的方便和简洁，那关于 定时任务的真正使用还有哪些你不知道的事呢？下面我们一起来看一下吧。

<!--more-->

### 官方条件

在使用`Scheduled` 注解前，我们先看下，官方给出了使用条件

> An annotation that marks a method to be scheduled. Exactly one of the cron(), fixedDelay(), or fixedRate() attributes must be specified.
>
> The annotated method must expect no arguments. It will typically have a void return type; if not, the returned value will be ignored when called through the scheduler.
>
> Processing of `@Scheduled` annotations is performed by registering a [`ScheduledAnnotationBeanPostProcessor`](https://docs.spring.io/spring/docs/5.0.4.BUILD-SNAPSHOT/javadoc-api/org/springframework/scheduling/annotation/ScheduledAnnotationBeanPostProcessor.html). This can be done manually or, more conveniently, through the `<task:annotation-driven/>`element or @[`EnableScheduling`](https://docs.spring.io/spring/docs/5.0.4.BUILD-SNAPSHOT/javadoc-api/org/springframework/scheduling/annotation/EnableScheduling.html) annotation.

意思是说，在使用 `Scheduled` 注解的时候，必须设置 `cron`()，`fixedDelay()`， `fixedRate()` 三个中的一个属性。该注解使用的方法不期望接收任何参数，并且要使用 `void` 返回类型，即使设置了返回值，也会在回调的时候被忽略。第三句是说，我们在使用 `Scheduled` 注解的时候需要注册 `ScheduledAnnotationBeanPostProcessor`  ，可以通过手动，或者配置 `<task:annotation-driven/>` 或者在 `Springboot` 项目中，直接在启动类上添加  `@EnableScheduling` 注解都可以。总结一下就是

1. 设置定时策略；
2. 不设置返回类型；
3. 注册 `processor`。

### 使用案例

在平时使用的时候我们基本上只要加一个注解，然后再配置上相应的 `cron` 参数就可以了。如下所示：

![scheduled1](http://www.justdojava.com/assets/images/2019/java/image_ziyou/scheduled1.png)

#### Scheduled 参数配置说明

1. `cron()` 配置

   `cron` 是一个六位或者七位的字符串，每一位都对应的相应的含义，如上`0 0 0 * * ?` 表示的是每天 0 点执行。表达式从左往右依次代表的含义是`Seconds Minutes Hours DayofMonth Month DayofWeek Year` 或 `Seconds Minutes Hours DayofMonth Month DayofWeek` 详细的配置方式，读者感兴趣可以自行研究，而且网上也有根据条件自动生产表达式的，可以参考。

2. `fixedDelay()` 方式，执行完毕后调用

   官方解释 

   > `Execute the annotated method with a fixed period in milliseconds between the end of the last invocation and the start of the next.` 

   意思是说在上一次执行完毕和下一次开始调用的中间，延迟一段时间，以毫秒为单位。

3. `fixedRate()` 方式，开始执行后延迟调用

   官方解释

   > `Execute the annotated method with a fixed period in milliseconds between invocations.` 

   固定一个时间间隔进行前后方法的调用。

这几种的配置方式还是比较简单的，可以根据自己的场景以及需求来设置。可以说都还是比较常用的。

### 注意

这里结合自身项目，跟大家介绍几个在使用的过程中容易遇到的一些问题。

#### 单线程问题

**`Spring` 的任务调用默认使用的是单线程模式**，这个如果大家没有在生产上真正遇到过可能不会发现有问题，毕竟本地和测试环境数据少，不容易出现，但是在真正复杂的生产环境中，还是要十分注意的。

![image-20190826221856024](http://www.justdojava.com/assets/images/2019/java/image_ziyou/schedule02.png)

由于 `Spring` 的默认实现是单线程，所以在生产的时候如果任务一旦多了，而且耗时长了，就会出现很多意想不到的情况，比如会发现某个任务不执行，或者没有按照指定的时间运行。这种情况往往是很恐怖的，如果说对实时性要求不高的话，影响不大，如果实时性要求很高的话，那绝对不能容忍。

这个时候我们需要自定义`taskScheduler` ，通过如下方式

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;

/**
 * <br>
 * <b>功能：</b><br>
 * <b>作者：</b>@author 子悠<br>
 * <b>日期：</b>2019-08-22 17:51<br>
 * <b>详细说明：</b>无<br>
 */
@Configuration
public class ScheduledThreadPoolConfig implements SchedulingConfigurer {

    @Value("${thread.pool.corePoolSize:10}")
    private int corePoolSize;

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        scheduledTaskRegistrar.setScheduler(Executors.newScheduledThreadPool(corePoolSize));
    }
}

```

通过实现 `SchedulingConfigurer` 接口中的 `configureTasks` 方法，来创建一个 `ScheduledTask` 并注入到 `ScheduledTaskRegistrar` 中。

 这样就可以自定义线程数，从而避免上述问题的存在。

#### 多实例问题

另外还有一个问题就是如果一个模块定时任务很多，最好可以封装为一个独立的任务调度模块，对于负责场景的任务调度可以采用一些开源的调度框架，比如 `XXL-JOB`，`Azkaban`。

如果只是一些简单的定时任务，不需要引入开源框架的时候，那么自己开发后在部署的时候需要注意多实例会重复运行的情况，意思是说一个定时任务模块，如果部署了两个实例就会导致任务重复执行。这种时候只能采用单实例部署，单实例部署的时候需要加守护进程和程序监控，避免程序出问题。

如果是在需要高可用，可以自己开发调度模块或者采用开源的框架实现。大部分场景 `XXL-JOB` 应该都可以满足。

### 总结

今天给大家介绍了 `Spring` 框架中的 `Scheduled` 组件的简单使用，以及在使用过程中注意的一些事项，希望对大家有帮助。

**人生有涯，学海无涯**，欢迎大家到我们《Java 极客技术》的知识星球中一起进步，一起成长。
