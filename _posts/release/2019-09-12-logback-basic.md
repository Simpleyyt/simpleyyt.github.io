---
layout: post
title:  一文带你了解logback的一些常用配置
tagline: by 小马
categories: java基础
tags: 
    - 小马
---


##  springProfile标签中定义多个环境

我们可以通过springProfile为不同的环境配置不同的日志输出规则，比如生产环境不开启console，级别为info等 下面来看看示例，

<!--more-->

logback-spring.xml文件，

```xml
<springProfile name="prod">
    <root level="INFO">
        <appender-ref ref="FILE"/>
    </root>
</springProfile>
<springProfile name="dev">  
    <root level="DEBUG">
        <appender-ref ref="CONSOLE"/>
    </root>
</springProfile>
```

然后我们运行应用时，根据指定的spring.profile.active进行选择对应的日志配置，如何来指定运行时激活的profile呢，有很多方式，比如我们可以直接在yml文件中指定，

application.yml文件，

```xml
spring:
  profiles:
    active: prod
```

在程序启动时，可以看到当前激活的profile，

```bash
The following profiles are active: prod
```

还有其它方式，不是本文的重点，就不多说了。

另外， springProfile的name可以包含多个名字，比如
```xml
<springProfile name="dev1,dev2,dev3">  
    <root level="DEBUG">
        <appender-ref ref="CONSOLE"/>
    </root>
</springProfile>
```

## ThresholdFilter 日志过滤

首先要说下日志级别的定义，

* ALL 各级包括自定义级别 
* DEBUG 指定细粒度信息事件是最有用的应用程序调试 
* ERROR 错误事件可能仍然允许应用程序继续运行 
* FATAL 指定非常严重的错误事件，这可能导致应用程序中止 
* INFO 指定能够突出在粗粒度级别的应用程序运行情况的信息的消息 
* OFF 这是最高等级，为了关闭日志记录 
* TRACE 指定细粒度比DEBUG更低的信息事件 
* WARN 指定具有潜在危害的情况

它们关系如下：

ALL < DEBUG < INFO < WARN < ERROR < FATAL < OFF。

ThresholdFilter： 临界值过滤器，过滤掉低于指定临界值的日志。比如，

```xml
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
```

会过滤掉低于WARN的日志。

我们可以通过这种方式对日志进行分类，比如输出正常日志和错误日志，就可以用类型下面这样的配置，

```xml
<!--常规日志-->
    <appender name="fcbox-uniorder" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>/app/applogs/test-system.log</File>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <!--<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%-5level] [TxId : %X{PtxId} , SpanId : %X{PspanId}] [%logger:%L] %msg%n</pattern>-->
            <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %sep[%thread] %sep%p %sep%c %sep[TxId : %X{PtxId} , SpanId : %X{PspanId}] %sep%msgToo%exToo%n</Pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>DEBUG</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>/app/applogs/test-system.%d{yyyy-MM-dd}.log</FileNamePattern>
        </rollingPolicy>
    </appender>

    <!--warn和错误级别日志-->
    <appender name="fcbox-uniorder-error"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>/app/applogs/test-system-error.log</File>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <!--<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%-5level] [TxId : %X{PtxId} , SpanId : %X{PspanId}] [%logger:%L] %msg%n</pattern>-->
            <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %sep[%thread] %sep%p %sep%c.%M \(%F:%L\) %sep[TxId : %X{PtxId} , SpanId : %X{PspanId}] %sep%msgToo%exToo%n</Pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>/app/applogs/test-system-error.%d{yyyy-MM-dd}.log</FileNamePattern>
        </rollingPolicy>
    </appender>
```

## 控制台输出日志

这个比较简单，

```xml
<!--输出到控制台-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--<Encoding>UTF-8</Encoding>-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%-5level] [%logger:%L] %msg%n</pattern>
        </encoder>
    </appender>
```

除了常用的encoder属性，还可以设置target属性，默认是System.out。


##  引入默认配置 

一般springboot项目，我们会在logback.xml中看到这样的配置，

 ```xml
 <include  resource="org/springframework/boot/logging/logback/base.xml"/>
 ```
 
这是springboot默认提供的日志配置，我们可以在springboot jar包下看到它的源码，

```xml
<included>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</included>
```
可以看到引入了defaults.xml，console-appender.xml和file-appdender.xml，有兴趣可以通过源码继续了解。这里就不详细说了。

## springboot项目配置输出sql

springboot项目已经不需要像以前的SSM一样，类似下面这样配置sql了，

```xml
<logger name="org.apache.ibatis" level="INFO"/>
<logger name="java.sql" level="INFO"/>
```

我们只需要把 root level设置成debug就可以看到mybatis的日志了。但是这样配置日志又太多了，也不好追踪问题。所以我们一般只需要配置dao所在的包就可以了，

```xml
<logger name="com.test.dao" level="DEBUG"></logger>
```

root的level还是info，这样既能看到sql，也不会有其它太多日志干扰。