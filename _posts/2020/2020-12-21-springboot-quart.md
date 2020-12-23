---
layout: post
categories: java
title: quart 架构与单体应用介绍
tagline: by 炸鸡可乐
tags: 
  - 炸鸡可乐
---

在上篇文章中，我们深入的介绍了单机版本定时任务的实现原理，今天我们继续来点干活，介绍在平时使用最多的一个定时任务框架 Quartz！

<!--more-->
### 一、摘要
阅读完本文大概需要5分钟，本文主要分享内容如下：

* Quartz 架构介绍
* SpringBoot Quartz 应用整合

### 二、关于 Quartz
Quartz 是 OpenSymphony 开源组织在 Job scheduling 领域开源的一个作业调度框架项目，完全由 Java 编写，主要是为了实现在 Java 应用程序中进行作业调度并提供了简单却强大的机制！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-quart/01.png)

Quartz 不仅可以单独使用，还可以与 J2EE 与 J2SE 应用程序相结合使用！

同时，Quartz 允许程序开发人员根据时间的间隔来调度作业！

与 JDK 中调度器不同的是，Quartz 实现了作业和触发器的多对多的关系，还能把多个作业与不同的触发器关联，一次可以调度几十个、上百个甚至上几万个复杂的程序！

### 三、Quartz 架构图
在详细介绍 Quartz 应用之前，我们先来看看它具体的架构图！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-quart/02.png)

从图中可以看出，Quartz 框架主要包括如下几个部分：

* **SchedulerFactory**：任务调度工厂，主要负责管理任务调度器
* **Scheduler**：任务调度控制器，主要是负责任务调度
* **Job**：任务接口，即被调度的任务
* **JobDetail**：Job 的任务描述类，job 执行时会依据此对象的信息反射实例化出 Job 的具体执行对象。
* **Trigger**：任务触发器，主要存放 Job 执行的时间策略。例如多久执行一次，什么时候执行，以什么频率执行等等
* **Calendar**：Trigger 扩展对象，可以排除或者包含某个指定的时间点（如排除法定节假日）
* **JobStore**：存储作业和任务调度期间的状态

可能感觉很抽象，下面我们先来看一个简单的例子，示例代码如下：

```java
/**
 * 实现 Job 任务接口
 */
public class QuartzTest implements Job {
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        System.out.println(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
    }
    public static void main(String[] args) throws SchedulerException {
        // 创建一个Scheduler
        Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
        // 启动Scheduler
        scheduler.start();
        // 新建一个Job, 指定执行类是QuartzTest, 指定一个K/V类型的数据, 指定job的name和group
        JobDetail job = JobBuilder.newJob(QuartzTest.class)
                .usingJobData("jobData", "test")
                .withIdentity("myJob", "myJobGroup")
                .build();
        // 新建一个Trigger, 表示JobDetail的调度计划, 这里的cron表达式是 每1秒执行一次
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("myTrigger", "myTriggerGroup")
                .startNow()
                .withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?"))
                .build();
        // 让scheduler开始调度这个job, 按trigger指定的计划
        scheduler.scheduleJob(job, trigger);
    }
}
```
运行结果如下：
```
2020-11-09 21:38:40
2020-11-09 21:38:45
2020-11-09 21:38:50
2020-11-09 21:38:55
2020-11-09 21:39:00
2020-11-09 21:39:05
2020-11-09 21:39:10
...
```
整个代码虽然简单，但是五脏俱全，在应用方面使用最多的就是`Job`和`Trigger`。

下面，我们就一起从源码层面来看看具体怎么使用！
#### 3.1、Job
打开`Job`源码，里面其实就是一个包含执行方法`void execute(JobExecutionContext context)`的接口，开发者只需实现接口来定义具体任务即可！

```java
public interface Job {

    void execute(JobExecutionContext context) throws JobExecutionException;
}
```

`JobExecutionContext` 类封装了获取上下文的各种信息，`Job`运行时的信息也保存在 `JobDataMap` 实例中！

例如，我想要获取在上文初始化时使用到的`usingJobData("jobData", "test")`参数，可以通过如下方式进行获取！

```java
@Override
public void execute(JobExecutionContext context) throws JobExecutionException {
    //从context中获取instName，groupName以及dataMap
    String jobName = context.getJobDetail().getKey().getName();
    String groupName = context.getJobDetail().getKey().getGroup();
    JobDataMap dataMap = context.getJobDetail().getJobDataMap();
    //从dataMap中获取myDescription，myValue以及myArray
    String value = dataMap.getString("jobData");
    System.out.println("jobName:" + jobName + ",groupName:" + groupName + ",jobData:" + value);
}
```
输出结果：
```java
jobName:myJob,groupName:myJobGroup,jobData:test
```
#### 3.2、Trigger
`Trigger`主要用于描述`Job`执行的时间触发规则，最常用的有`SimpleTrigger`和`CronTrigger`两个实现类型。

* **SimpleTrigger**：主要处理一些简单的调度规则，例如触发一次或者以固定时间间隔周期执行
* **CronTrigger**：调度处理更加灵活，可以通过`Cron`表达式定义出各种复杂时间规则的调度方案，例如每早晨9:00执行，周一、周三、周五下午5:00执行等；

##### 3.2.1、SimpleTrigger
用`SimpleTrigger`实现每2秒钟执行一次任务为例，代码如下：
```java
public static void main(String[] args) throws SchedulerException {
    //构建一个JobDetail实例...

    // 构建一个Trigger，指定Trigger名称和组，规定该Job立即执行，且两秒钟重复执行一次
    SimpleTrigger trigger = TriggerBuilder.newTrigger()
            .startNow() // 执行的时机，立即执行
            .withIdentity("myTrigger", "myTriggerGroup") // 不是必须的
            .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                    .withIntervalInSeconds(2).repeatForever()).build();

    // 让scheduler开始调度这个job, 按trigger指定的计划
    scheduler.scheduleJob(job, trigger);
}
```
运行结果：
```java
2020-12-03 16:55:53
2020-12-03 16:55:55
2020-12-03 16:55:57
......
```
其中最关键的就是`withSchedule()`这个方法，通过`SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(2).repeatForever()`来构建了一个简单的`SimpleTrigger`类型的任务调度规则，从而实现任务调度！

##### 3.2.2、CronTrigger
如开始介绍的例子一样，里面使用正是`CronTrigger`类型的调度规则！
```java
public static void main(String[] args) throws SchedulerException {
    //构建一个JobDetail实例...

    // 新建一个Trigger, 表示JobDetail的调度计划, 这里的cron表达式是 每5秒执行一次
    Trigger trigger = TriggerBuilder.newTrigger()
            .withIdentity("myTrigger", "myTriggerGroup")
            .startNow() // 立即执行
            .withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?"))
            .build();

    // 让scheduler开始调度这个job, 按trigger指定的计划
    scheduler.scheduleJob(job, trigger);
}
```
运行结果：
```java
2020-12-03 17:09:10
2020-12-03 17:09:15
2020-12-03 17:09:20
......
```
`CronTrigger`相比`SimpleTrigger`，在配置调度规则方面，使用`cron`表达式更加灵活！

##### 3.2.3、Cron 表达式详解
Quartz 的 Cron 表达式，具体配置规则可以参考如下：
```
.---------------------- seconds(0 - 59)
|  .------------------- minute (0 - 59)
|  |  .---------------- hour (0 - 23)
|  |  |  .------------- day of month (1 - 31)
|  |  |  |  .---------- month (1 - 12)
|  |  |  |  |  .------- Day-of-Week (1 - 7) 
|  |  |  |  |  |  .---- year (1970 - 2099) ...
|  |  |  |  |  |  |
*  *  *  *  *  ?  *
```

具体样例如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-quart/03.jpg)

在 cron 表达式中不区分大小写，更多配置和使用操作可以参考[这里](https://www.matools.com/cron)。

还可以在线解析`cron`表达式进行测试。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-quart/04.jpg)

#### 3.3、监听器（选用）
quartz 除了提供能正常调度任务的功能之外，还提供了监听器功能！

所谓监听器，其实你可以把它理解为类似`Spring Aop`的功能，可以对全局或者局部实现监听！

监听器应用，在实际项目中并不常用，但是在某些业务场景下，可以发挥一定的作用，例如：你想在任务处理完成之后，去发送邮件或者发短信进行通知，但是你又不想改以前的代码，这个时候就可以在监听器里面完成改项任务！

quartz 监听器主要分三大类：

* SchedulerListener：任务调度监听器
* TriggerListener：任务触发监听器
* JobListener：任务执行监听器

##### 3.3.1、SchedulerListener
`SchedulerListener`监听器，主要对任务调度`Scheduler`生命周期中关键节点进行监听，它只能全局进行监听，简单示例如下：

```java
public class SimpleSchedulerListener implements SchedulerListener {

    @Override
    public void jobScheduled(Trigger trigger) {
        System.out.println("任务被部署时被执行");
    }

    @Override
    public void jobUnscheduled(TriggerKey triggerKey) {
        System.out.println("任务被卸载时被执行");
    }

    @Override
    public void triggerFinalized(Trigger trigger) {
        System.out.println("任务完成了它的使命，光荣退休时被执行");
    }

    @Override
    public void triggerPaused(TriggerKey triggerKey) {
        System.out.println(triggerKey + "（一个触发器）被暂停时被执行");
    }

    @Override
    public void triggersPaused(String triggerGroup) {
        System.out.println(triggerGroup + "所在组的全部触发器被停止时被执行");
    }

    @Override
    public void triggerResumed(TriggerKey triggerKey) {
        System.out.println(triggerKey + "（一个触发器）被恢复时被执行");
    }

    @Override
    public void triggersResumed(String triggerGroup) {
        System.out.println(triggerGroup + "所在组的全部触发器被回复时被执行");
    }

    @Override
    public void jobAdded(JobDetail jobDetail) {
        System.out.println("一个JobDetail被动态添加进来");
    }

    @Override
    public void jobDeleted(JobKey jobKey) {
        System.out.println(jobKey + "被删除时被执行");
    }

    @Override
    public void jobPaused(JobKey jobKey) {
        System.out.println(jobKey + "被暂停时被执行");
    }

    @Override
    public void jobsPaused(String jobGroup) {
        System.out.println(jobGroup + "(一组任务）被暂停时被执行");
    }

    @Override
    public void jobResumed(JobKey jobKey) {
        System.out.println(jobKey + "被恢复时被执行");
    }

    @Override
    public void jobsResumed(String jobGroup) {
        System.out.println(jobGroup + "(一组任务）被恢复时被执行");
    }

    @Override
    public void schedulerError(String msg, SchedulerException cause) {
        System.out.println("出现异常" + msg + "时被执行");
        cause.printStackTrace();
    }

    @Override
    public void schedulerInStandbyMode() {
        System.out.println("scheduler被设为standBy等候模式时被执行");
    }

    @Override
    public void schedulerStarted() {
        System.out.println("scheduler启动时被执行");
    }

    @Override
    public void schedulerStarting() {
        System.out.println("scheduler正在启动时被执行");
    }

    @Override
    public void schedulerShutdown() {
        System.out.println("scheduler关闭时被执行");
    }

    @Override
    public void schedulerShuttingdown() {
        System.out.println("scheduler正在关闭时被执行");
    }

    @Override
    public void schedulingDataCleared() {
        System.out.println("scheduler中所有数据包括jobs, triggers和calendars都被清空时被执行");
    }
}
```
需要在任务调度器启动前，将`SimpleSchedulerListener`注册到`Scheduler`容器中！
```java
// 创建一个Scheduler
Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
//添加SchedulerListener监听器
scheduler.getListenerManager().addSchedulerListener(new SimpleSchedulerListener());
// 启动Scheduler
scheduler.start();
```
运行`main`方法，输出结果如下：
```
scheduler正在启动时被执行
scheduler启动时被执行
一个JobDetail被动态添加进来
任务被部署时被执行
2020-12-10 17:27:10
....
```
##### 3.3.2、TriggerListener
`TriggerListener`，与触发器`Trigger`相关的事件都会被监听，它既可以全局监听，也可以实现局部监听。

所谓局部监听，就是对某个`Trigger`的名称或者组进行监听，简单示例如下：

```java
public class SimpleTriggerListener implements TriggerListener {

    /**
     * Trigger监听器的名称
     * @return
     */
    @Override
    public String getName() {
        return "mySimpleTriggerListener";
    }

    /**
     * Trigger被激发 它关联的job即将被运行
     * @param trigger
     * @param context
     */
    @Override
    public void triggerFired(Trigger trigger, JobExecutionContext context) {
        System.out.println("myTriggerListener.triggerFired()");
    }

    /**
     * Trigger被激发 它关联的job即将被运行, TriggerListener 给了一个选择去否决 Job 的执行,如果返回TRUE 那么任务job会被终止
     * @param trigger
     * @param context
     * @return
     */
    @Override
    public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context) {
        System.out.println("myTriggerListener.vetoJobExecution()");
        return false;
    }

    /**
     * 当Trigger错过被激发时执行,比如当前时间有很多触发器都需要执行，但是线程池中的有效线程都在工作，
     * 那么有的触发器就有可能超时，错过这一轮的触发。
     * @param trigger
     */
    @Override
    public void triggerMisfired(Trigger trigger) {
        System.out.println("myTriggerListener.triggerMisfired()");
    }

    /**
     * 任务完成时触发
     * @param trigger
     * @param context
     * @param triggerInstructionCode
     */
    @Override
    public void triggerComplete(Trigger trigger, JobExecutionContext context, Trigger.CompletedExecutionInstruction triggerInstructionCode) {
        System.out.println("myTriggerListener.triggerComplete()");
    }
}
```
与之类似，需要将`SimpleTriggerListener`注册到`Scheduler`容器中！
```java
// 创建一个Scheduler
Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
// 创建并注册一个全局的Trigger Listener
scheduler.getListenerManager().addTriggerListener(new SimpleTriggerListener(), EverythingMatcher.allTriggers());
        
// 创建并注册一个局部的Trigger Listener
//scheduler.getListenerManager().addTriggerListener(new SimpleTriggerListener(), KeyMatcher.keyEquals(TriggerKey.triggerKey("myTrigger", "myJobTrigger")));
        
// 创建并注册一个特定组的Trigger Listener
//scheduler.getListenerManager().addTriggerListener(new SimpleTriggerListener(), GroupMatcher.groupEquals("myTrigger"));
// 启动Scheduler
scheduler.start();
```
##### 3.3.3、JobListener
`JobListener`，与任务执行`Job`相关的事件都会被监听，和`Trigger`一样，既可以全局监听，也可以实现局部监听。

简单示例如下：

```java
public class SimpleJobListener implements JobListener {

    /**
     * job监听器名称
     * @return
     */
    @Override
    public String getName() {
        return "mySimpleJobListener";
    }

    /**
     * 任务被调度前
     * @param context
     */
    @Override
    public void jobToBeExecuted(JobExecutionContext context) {
        System.out.println("simpleJobListener监听器，准备执行："+context.getJobDetail().getKey());
    }

    /**
     * 任务调度被拒了
     * @param context
     */
    @Override
    public void jobExecutionVetoed(JobExecutionContext context) {
        System.out.println("simpleJobListener监听器，取消执行："+context.getJobDetail().getKey());
    }

    /**
     * 任务被调度后
     * @param context
     * @param jobException
     */
    @Override
    public void jobWasExecuted(JobExecutionContext context, JobExecutionException jobException) {
        System.out.println("simpleJobListener监听器，执行结束："+context.getJobDetail().getKey());
    }
}

```
同样的，将`SimpleJobListener`注册到`Scheduler`容器中，即可实现监听！
```java
// 创建一个Scheduler
Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
// 创建并注册一个全局的Job Listener
scheduler.getListenerManager().addJobListener(new SimpleJobListener(), EverythingMatcher.allJobs());

// 创建并注册一个指定任务的Job Listener
//scheduler.getListenerManager().addJobListener(new SimpleJobListener(), KeyMatcher.keyEquals(JobKey.jobKey("myJob", "myJobGroup")));

// 将同一任务组的任务注册到监听器中
//scheduler.getListenerManager().addJobListener(new SimpleJobListener(), GroupMatcher.jobGroupEquals("myJobGroup"));

// 启动Scheduler
scheduler.start();
```
##### 3.3.4、小结
如果想多个同时监听，将其依次加入即可！

```java
Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();

//添加SchedulerListener监听器
scheduler.getListenerManager().addSchedulerListener(new SimpleSchedulerListener());

// 创建并注册一个全局的Trigger Listener
scheduler.getListenerManager().addTriggerListener(new SimpleTriggerListener(), EverythingMatcher.allTriggers());

// 创建并注册一个全局的Job Listener
scheduler.getListenerManager().addJobListener(new SimpleJobListener(), EverythingMatcher.allJobs());

// 启动Scheduler
scheduler.start();
```

### 四、单体应用介绍
下面我们以每5秒执行一次任务为例，采用`SpringBoot + Quartz`进行技术实现。流程如下！

#### 4.1、引入 quartz 包
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```
#### 4.2、编写具体任务
```java
public class TestTask implements Job {


    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("testTask：" + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
    }
}
```
#### 4.3、编写任务调度配置服务
与上面的参数基本一致
```java
@Configuration
public class TestTaskConfig {

    @Bean
    public JobDetail testQuartz() {
        return JobBuilder.newJob(TestTask.class)
                .usingJobData("jobData", "test")
                .withIdentity("myJob", "myJobGroup")
                .storeDurably()
                .build();
    }

    @Bean
    public Trigger testQuartzTrigger() {
        //5秒执行一次
        return TriggerBuilder.newTrigger()
                .forJob(testQuartz())
                .withIdentity("myTrigger", "myTriggerGroup")
                .startNow()
                .withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?"))
                .build();
    }
}
```
#### 4.4、任务测试
启动应用程序，即可观察到对应的任务运行状况！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-quart/05.jpg)

如果需要创建多个定时任务，配置流程也类似！

只是这种静态配置方式多了会带来一个问题，比如，现在这个任务是每5秒跑一次，我现在想改成1个小时或者凌晨2点跑一次，这个时候就必须要改代码呢，但是我又不想改代码，基于此需求，利用数据库存储，我们可以将其改成动态配置！
#### 4.5、动态配置定时任务
从上面的代码中我们可以分析中，定时任务的有三个核心变量，其他的方法都可以封装成公共的。

* 任务名称：例如`myJob`
* 任务执行类：例如`TestTask.class`
* 任务调度时间：例如`0/5 * * * * ?`

基于此规律，我们可以创建一个定时任务实体类，用于保存定时任务相关信息到数据库当中，然后编写一个定时任务工具库，用于创建、更新、删除、暂停任务操作，通过 restful 接口操作将任务存入数据库并管理定时任务！
#### 4.5.1、引入 Jpa 包
```xml
<!--jpa 支持-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!--mysql 数据源-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```
在`application.properties`中加入数据源配置
```
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
#### 4.5.2、创建任务配置表
```sql
CREATE TABLE `tb_job_task` (
  `id` varchar(50)  NOT NULL COMMENT '任务ID',
  `job_name` varchar(100)  DEFAULT NULL COMMENT '任务名称',
  `job_class` varchar(200)  DEFAULT NULL COMMENT '任务执行类',
  `cron_expression` varchar(50)  DEFAULT NULL COMMENT '任务调度时间表达式',
  `status` int(4) DEFAULT NULL COMMENT '任务状态，0：启动，1：暂停，2：停用',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```
#### 4.5.3、编写任务实体类
```java
@Entity
@Table(name = "tb_job_task")
public class QuartzBean {

    /**
     * 任务ID
     */
    @Id
    private String id;

    /**
     * 任务名称
     */
    @Column(name = "job_name")
    private String jobName;

    /**
     * 任务执行类
     */
    @Column(name = "job_class")
    private String jobClass;

    /**
     * 任务调度时间表达式
     */
    @Column(name = "cron_expression")
    private String cronExpression;

    /**
     * 任务状态，0：启动，1：暂停，2：停用
     */
    @Column(name = "status")
    private Integer status;

    //get、set...
}
```
#### 4.5.4、编写任务操作工具类
```java
public class QuartzUtils {

    private static final Logger log = LoggerFactory.getLogger(QuartzUtils.class);

    /**
     * 创建定时任务 定时任务创建之后默认启动状态
     * @param scheduler   调度器
     * @param quartzBean  定时任务信息类
     * @throws Exception
     */
    public static void createScheduleJob(Scheduler scheduler, QuartzBean quartzBean){
        try {
            //获取到定时任务的执行类  必须是类的绝对路径名称
            //定时任务类需要是job类的具体实现 QuartzJobBean是job的抽象类。
            Class<? extends Job> jobClass = (Class<? extends Job>) Class.forName(quartzBean.getJobClass());
            // 构建定时任务信息
            JobDetail jobDetail = JobBuilder.newJob(jobClass).withIdentity(quartzBean.getJobName()).build();
            // 设置定时任务执行方式
            CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(quartzBean.getCronExpression());
            // 构建触发器trigger
            CronTrigger trigger = TriggerBuilder.newTrigger().withIdentity(quartzBean.getJobName()).withSchedule(scheduleBuilder).build();
            scheduler.scheduleJob(jobDetail, trigger);
        } catch (ClassNotFoundException e) {
            log.error("定时任务类路径出错：请输入类的绝对路径", e);
        } catch (SchedulerException e) {
            log.error("创建定时任务出错", e);
        }
    }

    /**
     * 根据任务名称暂停定时任务
     * @param scheduler  调度器
     * @param jobName    定时任务名称
     * @throws SchedulerException
     */
    public static void pauseScheduleJob(Scheduler scheduler, String jobName){
        JobKey jobKey = JobKey.jobKey(jobName);
        try {
            scheduler.pauseJob(jobKey);
        } catch (SchedulerException e) {
            log.error("暂停定时任务出错", e);
        }
    }

    /**
     * 根据任务名称恢复定时任务
     * @param scheduler  调度器
     * @param jobName    定时任务名称
     * @throws SchedulerException
     */
    public static void resumeScheduleJob(Scheduler scheduler, String jobName) {
        JobKey jobKey = JobKey.jobKey(jobName);
        try {
            scheduler.resumeJob(jobKey);
        } catch (SchedulerException e) {
            log.error("暂停定时任务出错", e);
        }
    }

    /**
     * 根据任务名称立即运行一次定时任务
     * @param scheduler     调度器
     * @param jobName       定时任务名称
     * @throws SchedulerException
     */
    public static void runOnce(Scheduler scheduler, String jobName){
        JobKey jobKey = JobKey.jobKey(jobName);
        try {
            scheduler.triggerJob(jobKey);
        } catch (SchedulerException e) {
            log.error("运行定时任务出错", e);
        }
    }

    /**
     * 更新定时任务
     * @param scheduler   调度器
     * @param quartzBean  定时任务信息类
     * @throws SchedulerException
     */
    public static void updateScheduleJob(Scheduler scheduler, QuartzBean quartzBean)  {
        try {
            //获取到对应任务的触发器
            TriggerKey triggerKey = TriggerKey.triggerKey(quartzBean.getJobName());
            //设置定时任务执行方式
            CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(quartzBean.getCronExpression());
            //重新构建任务的触发器trigger
            CronTrigger trigger = (CronTrigger) scheduler.getTrigger(triggerKey);
            trigger = trigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
            //重置对应的job
            scheduler.rescheduleJob(triggerKey, trigger);
        } catch (SchedulerException e) {
            log.error("更新定时任务出错", e);
        }
    }

    /**
     * 根据定时任务名称从调度器当中删除定时任务
     * @param scheduler 调度器
     * @param jobName   定时任务名称
     * @throws SchedulerException
     */
    public static void deleteScheduleJob(Scheduler scheduler, String jobName) {
        JobKey jobKey = JobKey.jobKey(jobName);
        try {
            scheduler.deleteJob(jobKey);
        } catch (SchedulerException e) {
            log.error("删除定时任务出错", e);
        }
    }
}
```
#### 4.5.5、编写 Jpa 数据操作服务
```java
public interface QuartzBeanRepository extends JpaRepository<QuartzBean,String> {

    /**
     * 修改任务状态
     * @param id
     * @param status
     */
    @Modifying
    @Transactional
    @Query("update QuartzBean m set m.status = ?2 where m.id =?1")
    void updateState(String id, Integer status);

    /**
     * 修改任务调度时间
     * @param id
     * @param cronExpression
     */
    @Modifying
    @Transactional
    @Query("update QuartzBean m set m.cronExpression = ?2 where m.id =?1")
    void updateCron(String id, String cronExpression);
}
```
#### 4.5.6、编写controller服务
```java
@RestController
@RequestMapping("/quartz")
public class QuartzController {

    private static final Logger log = LoggerFactory.getLogger(QuartzController.class);

    @Autowired
    private Scheduler scheduler;

    @Autowired
    private QuartzBeanRepository repository;

    /**
     * 创建任务
     * @param quartzBean
     */
    @RequestMapping("/createJob")
    public HttpStatus createJob(@RequestBody  QuartzBean quartzBean)  {
        log.info("=========开始创建任务=========");
        QuartzUtils.createScheduleJob(scheduler,quartzBean);
        repository.save(quartzBean.setId(UUID.randomUUID().toString()).setStatus(0));
        log.info("=========创建任务成功，信息：{}", JSON.toJSONString(quartzBean));
        return HttpStatus.OK;
    }

    /**
     * 暂停任务
     * @param quartzBean
     */
    @RequestMapping("/pauseJob")
    public HttpStatus pauseJob(@RequestBody QuartzBean quartzBean) {
        log.info("=========开始暂停任务，请求参数：{}=========", JSON.toJSONString(quartzBean));
        QuartzUtils.pauseScheduleJob(scheduler, quartzBean.getJobName());
        repository.updateState(quartzBean.getId(), 1);
        log.info("=========暂停任务成功=========");
        return HttpStatus.OK;
    }

    /**
     * 立即运行一次定时任务
     * @param quartzBean
     */
    @RequestMapping("/runOnce")
    public HttpStatus runOnce(@RequestBody QuartzBean quartzBean) {
        log.info("=========立即运行一次定时任务，请求参数：{}", JSON.toJSONString(quartzBean));
        QuartzUtils.runOnce(scheduler,quartzBean.getJobName());
        log.info("=========立即运行一次定时任务成功=========");
        return HttpStatus.OK;
    }

    /**
     * 恢复定时任务
     * @param quartzBean
     */
    @RequestMapping("/resume")
    public HttpStatus resume(@RequestBody QuartzBean quartzBean) {
        log.info("=========恢复定时任务，请求参数：{}", JSON.toJSONString(quartzBean));
        QuartzUtils.resumeScheduleJob(scheduler,quartzBean.getJobName());
        repository.updateState(quartzBean.getId(), 0);
        log.info("=========恢复定时任务成功=========");
        return HttpStatus.OK;
    }

    /**
     * 更新定时任务
     * @param quartzBean
     */
    @RequestMapping("/update")
    public HttpStatus update(@RequestBody QuartzBean quartzBean)  {
        log.info("=========更新定时任务，请求参数：{}", JSON.toJSONString(quartzBean));
        QuartzUtils.updateScheduleJob(scheduler,quartzBean);
        repository.updateCron(quartzBean.getId(), quartzBean.getCronExpression());
        log.info("=========更新定时任务成功=========");
        return HttpStatus.OK;
    }

    /**
     * 删除定时任务
     * @param quartzBean
     */
    @RequestMapping("/delete")
    public HttpStatus delete(@RequestBody QuartzBean quartzBean)  {
        log.info("=========删除定时任务，请求参数：{}", JSON.toJSONString(quartzBean));
        QuartzUtils.deleteScheduleJob(scheduler,quartzBean.getJobName());
        repository.updateState(quartzBean.getId(), 2);
        log.info("=========删除定时任务成功=========");
        return HttpStatus.OK;
    }

}
```
#### 4.5.7、服务重启补偿
在应用程序正常运行的时候，虽然没问题，但是当我们重启服务的时候，这个时候内存的里面的定时任务其实全部都被销毁，因此在应用程序启动的时候，还需要将正常的任务重新加入到服务中！

```java
@Component
public class TaskConfigApplicationRunner implements ApplicationRunner {

    @Autowired
    private QuartzBeanRepository repository;

    @Autowired
    private Scheduler scheduler;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        List<QuartzBean> list = repository.findAll();
        if(!CollectionUtils.isEmpty(list)){
            for (QuartzBean quartzBean : list) {
                //加载启动类型的定时任务
                if(quartzBean.getStatus().intValue() == 0){
                    QuartzUtils.createScheduleJob(scheduler,quartzBean);
                }
                //加载暂停类型的定时任务
                if(quartzBean.getStatus().intValue() == 1){
                    QuartzUtils.createScheduleJob(scheduler,quartzBean);
                    QuartzUtils.pauseScheduleJob(scheduler, quartzBean.getJobName());
                }
            }
        }
    }
}
```
在服务重启的时候，会重新将有效任务加入quartz 中！
#### 4.5.8、接口服务测试

* 调用`quartz/createJob`接口，创建任务

```
=========开始创建任务=========
=========创建任务成功，信息：{"cronExpression":"0/5 * * * * ?","id":"9280b799-f762-47b5-90e4-4874e1ad3c60","jobClass":"com.example.quartz.config.TestTask","jobName":"myJob","status":0}
testTask：2020-12-07 22:31:05
testTask：2020-12-07 22:31:10
testTask：2020-12-07 22:31:15
...
```

* 调用`quartz/pauseJob`接口，暂停任务

```
=========开始暂停任务，请求参数：{"id":"9280b799-f762-47b5-90e4-4874e1ad3c60","jobName":"myJob"}
=========暂停任务成功=========
```

* 调用`quartz/runOnce`接口，立即运行一次定时任务

```
=========立即运行一次定时任务，请求参数：{"jobName":"myJob"}
=========立即运行一次定时任务=========
testTask：2020-12-07 22:36:30
```

* 调用`quartz/resume`接口，恢复定时任务

```
=========恢复定时任务，请求参数：{"id":"9280b799-f762-47b5-90e4-4874e1ad3c60","jobName":"myJob"}
=========恢复定时任务=========
testTask：2020-12-07 22:38:00
testTask：2020-12-07 22:38:05
testTask：2020-12-07 22:38:10
...
```

* 调用`quartz/update`接口，更新定时任务

```
=========更新定时任务，请求参数：{"cronExpression":"0/10 * * * * ?","id":"9280b799-f762-47b5-90e4-4874e1ad3c60","jobClass":"com.example.quartz.config.TestTask","jobName":"myJob"}
=========更新定时任务=========
testTask：2020-12-07 22:39:10
testTask：2020-12-07 22:39:20
testTask：2020-12-07 22:39:30
...
```

* 调用`quartz/delete`接口，删除定时任务

```
=========删除定时任务，请求参数：{"id":"9280b799-f762-47b5-90e4-4874e1ad3c60","jobName":"myJob"}=========
=========删除定时任务=========
```

#### 4.5.9、添加监听器（选用）
当然，如果你想对某个任务实现监听，只需要添加一个配置类，将其注入即可！

```java
@Configuration
public class TestTaskConfig {

	@Primary
    @Bean
    public Scheduler initScheduler() throws SchedulerException {
        Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
        //添加SchedulerListener监听器
        scheduler.getListenerManager().addSchedulerListener(new SimpleSchedulerListener());

        // 创建并注册一个全局的Trigger Listener
        scheduler.getListenerManager().addTriggerListener(new SimpleTriggerListener(), EverythingMatcher.allTriggers());

        // 创建并注册一个全局的Job Listener
        scheduler.getListenerManager().addJobListener(new SimpleJobListener(), EverythingMatcher.allJobs());
        scheduler.start();
        return scheduler;
    }

}
```
#### 4.5.10、小结
需要注意的是：在 quartz 任务暂停之后再次启动时，会立即执行一次，在更新之后也会立即执行一次任务调度！

### 五、总结
本文主要围绕 quartz 的架构以及`Springboot + quartz`项目整合和应用做了初步的介绍。

可能有的朋友会发问，quartz 已经有对应的任务表，不需要手动创建，只需要配置`quartz.properties`文件即可！

没错，`quartz`官方提供了对应任务表，主要用于分布式架构下的任务处理！

如果只是单体应用，可以参考本文中创建一张单表来存储任务，实现简单！

如果集群环境部署，在下篇文章中，我们会详细的介绍`quartz`分布式架构的开发和应用！

鉴于笔者才疏学浅，如果发现有错误的地方，欢迎网友批评指导！
### 六、参考
1、[简书 - Quartz 教程](https://www.jianshu.com/p/ce4c4400eea2)

2、[知乎 - springboot集成quartz笔记](https://zhuanlan.zhihu.com/p/35605394)

