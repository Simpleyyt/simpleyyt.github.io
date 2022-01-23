---
layout: post
categories: java
title: SpringBoot Schedule应用实践
tagline: by 炸鸡可乐
tags: 
  - 炸鸡可乐
---

3分钟带你搞定SpringBoot中Schedule

<!--more-->
### 一、摘要
阅读完本文大概需要3分钟，本文主要分享内容如下：

* SpringBoot Schedule  实践介绍

### 二、介绍

在实际的业务开发过程中，我们经常会需要定时任务来帮助我们完成一些工作，例如每天早上 6 点生成销售报表、每晚 23 点清理脏数据等等。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-schedule/01.jpg)

如果你当前使用的是 SpringBoot 来开发项目，那么完成这些任务会非常容易！

SpringBoot 默认已经帮我们完成了相关定时任务组件的配置，我们只需要添加相应的注解`@Scheduled`就可以实现任务调度！

### 三、SpringBoot Schedule 应用实践
#### 3.1、pom 包配置
`pom`包里面只需要引入`Spring Boot Starter`包即可！
```xml
<dependencies>
    <!--spring boot核心-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!--spring boot 测试-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```
#### 3.2、启动类启用定时调度
在启动类上面加上`@EnableScheduling`即可开启定时
```java
@SpringBootApplication
@EnableScheduling
public class ScheduleApplication {

    public static void main(String[] args) {
        SpringApplication.run(ScheduleApplication.class, args);
    }
}
```
#### 3.3、创建定时任务
`Spring Scheduler`支持四种形式的任务调度！

* **fixedRate**：固定速率执行，例如每5秒执行一次
* **fixedDelay**：固定延迟执行，例如距离上一次调用成功后2秒执行
* **initialDelay**：初始延迟任务，例如任务开启过5秒后再执行，之后以固定频率或者间隔执行
* **cron**：使用 Cron 表达式执行定时任务

##### 3.3.1、固定速率执行
你可以通过使用`fixedRate`参数以固定时间间隔来执行任务，示例如下：
```java
@Component
public class SchedulerTask {

    private static final Logger log = LoggerFactory.getLogger(SchedulerTask.class);
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    /**
	 * fixedRate：固定速率执行。每5秒执行一次。
	 */
	@Scheduled(fixedRate = 5000)
	public void runWithFixedRate() {
	    log.info("Fixed Rate Task，Current Thread : {}，The time is now : {}", Thread.currentThread().getName(), dateFormat.format(new Date()));
	}
}
```
运行`ScheduleApplication`主程序，即可看到控制台输出效果：
```
Fixed Rate Task，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 11:46:00
Fixed Rate Task，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 11:46:10
...
```
##### 3.3.2、固定延迟执行
你可以通过使用`fixedDelay`参数来设置上一次任务调用完成与下一次任务调用开始之间的延迟时间，示例如下：
```java
@Component
public class SchedulerTask {

    private static final Logger log = LoggerFactory.getLogger(SchedulerTask.class);
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    /**
     * fixedDelay：固定延迟执行。距离上一次调用成功后2秒后再执行。
     */
    @Scheduled(fixedDelay = 2000)
    public void runWithFixedDelay() {
        log.info("Fixed Delay Task，Current Thread : {}，The time is now : {}", Thread.currentThread().getName(), dateFormat.format(new Date()));
    }
}
```
控制台输出效果：
```
Fixed Delay Task，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 11:46:00
Fixed Delay Task，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 11:46:02
...
```
##### 3.3.3、初始延迟任务
你可以通过使用`initialDelay`参数与`fixedRate`或者`fixedDelay`搭配使用来实现初始延迟任务调度。
```java
@Component
public class SchedulerTask {

    private static final Logger log = LoggerFactory.getLogger(SchedulerTask.class);
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

	/**
     * initialDelay:初始延迟。任务的第一次执行将延迟5秒，然后将以5秒的固定间隔执行。
     */
    @Scheduled(initialDelay = 5000, fixedRate = 5000)
    public void reportCurrentTimeWithInitialDelay() {
        log.info("Fixed Rate Task with Initial Delay，Current Thread : {}，The time is now : {}", Thread.currentThread().getName(), dateFormat.format(new Date()));
    }
}
```
控制台输出效果：
```
Fixed Rate Task with Initial Delay，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 11:46:05
Fixed Rate Task with Initial Delay，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 11:46:10
...
```
##### 3.3.4、使用 Cron 表达式
`Spring Scheduler`同样支持`Cron`表达式，如果以上简单参数都不能满足现有的需求，可以使用 cron 表达式来定时执行任务。

关于`cron`表达式的具体用法，可以点击参考这里： [https://cron.qqe2.com/](https://cron.qqe2.com/)
```java
@Component
public class SchedulerTask {

    private static final Logger log = LoggerFactory.getLogger(SchedulerTask.class);
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    /**
     * cron：使用Cron表达式。每6秒中执行一次
     */
    @Scheduled(cron = "*/6 * * * * ?")
    public void reportCurrentTimeWithCronExpression() {
        log.info("Cron Expression，Current Thread : {}，The time is now : {}", Thread.currentThread().getName(), dateFormat.format(new Date()));
    }
}
```
控制台输出效果：
```
Cron Expression，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 11:46:06
Cron Expression，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 11:46:12
...
```
#### 3.4、异步执行定时任务
在介绍**异步执行定时任务**之前，我们先看一个例子！

在下面的示例中，我们创建了一个每隔2秒执行一次的定时任务，在任务里面大概需要花费 3 秒钟，猜猜执行结果如何？
```java
@Component
public class AsyncScheduledTask {

    private static final Logger log = LoggerFactory.getLogger(AsyncScheduledTask.class);
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    @Scheduled(fixedRate = 2000)
    public void runWithFixedDelay() {
        try {
            TimeUnit.SECONDS.sleep(3);
            log.info("Fixed Delay Task, Current Thread : {} : The time is now {}", Thread.currentThread().getName(), dateFormat.format(new Date()));
        } catch (InterruptedException e) {
            log.error("错误信息",e);
        }
    }
}
```
控制台输入结果：
```
Fixed Delay Task, Current Thread : scheduling-1 : The time is now 2020-12-15 17:55:26
Fixed Delay Task, Current Thread : scheduling-1 : The time is now 2020-12-15 17:55:31
Fixed Delay Task, Current Thread : scheduling-1 : The time is now 2020-12-15 17:55:36
Fixed Delay Task, Current Thread : scheduling-1 : The time is now 2020-12-15 17:55:41
...
```
很清晰的看到，任务调度频率变成了每隔5秒调度一次！

这是为啥呢？

从`Current Thread : scheduling-1`输出结果可以很看到，任务执行都是同一个线程！默认的情况下，`@Scheduled`任务都在 Spring 创建的大小为 1 的默认线程池中执行！

更直观的结果是，任务都是串行执行！

下面，我们将其改成异步线程来执行，看看效果如何？

```java
@Component
@EnableAsync
public class AsyncScheduledTask {

    private static final Logger log = LoggerFactory.getLogger(AsyncScheduledTask.class);
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");


    @Async
    @Scheduled(fixedDelay = 2000)
    public void runWithFixedDelay() {
        try {
            TimeUnit.SECONDS.sleep(3);
            log.info("Fixed Delay Task, Current Thread : {} : The time is now {}", Thread.currentThread().getName(), dateFormat.format(new Date()));
        } catch (InterruptedException e) {
            log.error("错误信息",e);
        }
    }
}
```
控制台输出结果：
```
Fixed Delay Task, Current Thread : SimpleAsyncTaskExecutor-1 : The time is now 2020-12-15 18:55:26
Fixed Delay Task, Current Thread : SimpleAsyncTaskExecutor-2 : The time is now 2020-12-15 18:55:28
Fixed Delay Task, Current Thread : SimpleAsyncTaskExecutor-3 : The time is now 2020-12-15 18:55:30
...
```
任务的执行频率不受方法内的时间影响，以并行方式执行！

#### 3.5、自定义任务线程池
虽然默认的情况下，`@Scheduled`任务都在 Spring 创建的大小为 1 的默认线程池中执行，但是我们也可以自定义线程池，只需要实现`SchedulingConfigurer`类即可！

自定义线程池示例如下：
```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
        //线程池大小为10
        threadPoolTaskScheduler.setPoolSize(10);
        //设置线程名称前缀
        threadPoolTaskScheduler.setThreadNamePrefix("scheduled-thread-");
        //设置线程池关闭的时候等待所有任务都完成再继续销毁其他的Bean
        threadPoolTaskScheduler.setWaitForTasksToCompleteOnShutdown(true);
        //设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是阻塞住
        threadPoolTaskScheduler.setAwaitTerminationSeconds(60);
        //这里采用了CallerRunsPolicy策略，当线程池没有处理能力的时候，该策略会直接在 execute 方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务
        threadPoolTaskScheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        threadPoolTaskScheduler.initialize();

        scheduledTaskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
    }
}
```
我们启动服务，看看`cron`任务示例调度效果：
```
Cron Expression，Current Thread : scheduled-thread-1，The time is now : 2020-12-15 20:46:00
Cron Expression，Current Thread : scheduled-thread-2，The time is now : 2020-12-15 20:46:06
Cron Expression，Current Thread : scheduled-thread-3，The time is now : 2020-12-15 20:46:12
Cron Expression，Current Thread : scheduled-thread-4，The time is now : 2020-12-15 20:46:18
....
```
当前线程名称已经被改成自定义`scheduled-thread`的前缀！
### 四、小结
本文主要围绕`Spring scheduled`应用实践进行分享，如果是单体应用，使用`SpringBoot`内置的`@scheduled`注解可以解决大部分业务需求，上手非常容易！


### 五、参考
1、[SpringBoot @Schedule使用注意与原理](https://springboot.io/t/topic/2758)

