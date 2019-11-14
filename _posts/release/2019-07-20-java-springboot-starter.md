---
layout: post
category: SpingBoot
title: SpringBoot 精髓之 SpringBoot-starter
tagline: by 子悠
tags: 
  - 子悠

---

### 背景

在互联网发达的今天，容器化和微服务化是一种潮流，已经不是趋势了，而是潮流。不管是出去面试还是自己日常项目开发，容器化可能还没普及，但是微服务化是不能缺少的。在微服务如此盛行的天下，Spring Clound 已经很流行了，作为 SpringCloud 的基石 SpringBoot 自然也是不容忽视。关于 SpringBoot 我们 Java 极客技术团队专门为知识星球的用户制作了一套视频教程，视频已经发布了几章了，还在持续更新中，欢迎大家到知识星球中学习，进入知识星球后请记得发帖要机器码哦。这篇文章我们先了解一下SpringBoot Starter，SpringBoot 的 Starter 我们可以说是天天都在用，但是到底什么是 Starter，如何自己编写一个 Starter 呢？这篇文章我们来一探究竟。
<!--more-->

### SpringBoot-Starter

#### 什么是 Starter

我们先看下官方是咱们定义 Starter 的，如下

> Starters are a set of convenient dependency descriptors that you can include in your application.

意思是说 Starts 是一组可以让你很方便在应用增加的依赖关系描述符的集合。或者可以这样理解，平时我们开发的时候很多情况下都会有一个模块依赖另一个模块，这个时候我们一般都是采用 maven 的模块依赖，进行模块的依赖，但是这种情况下我们完全可以采用 Starter 的方式，将需要被依赖的模块用 Starter 的方式去开发，最后直接引入一个 Starter 也可以达到这样的效果。

#### 命名规则

讲起来可以比较抽象，我们这里以一个例子来试一下效果。在实战之前我们需要先了解的 Starter 的命名规则，在官方文档上很清楚的写了如下内容：

> Do not start your module names with `spring-boot` . Let’s assume that you are creating a starter for "acme", name the auto-configure module `acme-spring-boot-autoconfigure` and the starter `acme-spring-boot-starter`. If you only have one module combining the two, use `acme-spring-boot-starter`.

由于 SpringBoot 官方本身就会提供很多 Starter，为了区别哪些 Starter 是官方，哪些是私人的或者第三方的，所以 SpringBoot 官方提出，第三方在建立自己的 Starter 的时候命名规则统一用`xxx-spring-boot-starter`，而官方提供的 Starter 统一命名方式为`spring-boot-starter-xxx`。

#### 使用

假设我们现在有一个需求是将对象转换成 JSON，并在字符串前面加上一个名称，前轴我们支持通过配置文件配置的方式。(当然这个需求只是一个假设，我们用来测试功能用，实际的开发当然不会只是这么简单)。

1. 我们先创建一个 Starter，名字叫`myjson-spring-boot-starter`，并加入自动装配和 fastjson 的依赖，如下

![image-20190720182519601](http://justdojava.com/assets/images/2019/java/image_ziyou/starter1.png)

2. 然后我们创建一个 Service，并增加一个`public String objToJson(Object object)` 方法，直接调用`fastjson` 的方法

```java

import com.alibaba.fastjson.JSON;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author ziyou<br>
 * <b>Date：</b>2019-07-20 18:16<br>
 * <b>Desc：</b>无<br>
 */
public class MyJsonService {

    private String name;

    /**
     * 使用 fastjson 将对象转换为 json 字符串输出
     *
     * @param object 传入的对象
     * @return 输出的字符串
     */
    public String objToJson(Object object) {
        return getName() + JSON.toJSONString(object);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

3. 编写配置类

```java

import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author ziyou<br>
 * <b>Date：</b>2019-07-20 18:36<br>
 * <b>Desc：</b>无<br>
 */
@ConfigurationProperties(prefix = "ziyou.json")
public class MyJsonProperties {

    public static final String DEFAULT_NAME = "ziyou";

    private String name = DEFAULT_NAME;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}


```



4. 创建自动化配置类

```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author ziyou<br>
 * <b>Date：</b>2019-07-20 18:39<br>
 * <b>Desc：</b>无<br>
 */
@Configuration
@ConditionalOnClass({MyJsonService.class})
@EnableConfigurationProperties(MyJsonProperties.class)
public class MyJsonAutoConfiguration {

    /**
     * 注入属性类
     */
    @Autowired
    private MyJsonProperties myJsonProperties;

    /**
     * 当当前上下文中没有 MyJsonService 类时创建类
     *
     * @return 返回创建的实例
     */
    @Bean
    @ConditionalOnMissingBean(MyJsonService.class)
    public MyJsonService myJsonService() {
        MyJsonService myJsonService = new MyJsonService();
        myJsonService.setName(myJsonProperties.getName());
        return myJsonService;
    }
}


```

5. 然后我们再创建 `resource/META-INF/spring.factories` 文件，增加如下内容，将自动装配的类配置上

```properties

org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.ziyou.starter.MyJsonAutoConfiguration

```

6. 然后我们通过运行`mvn install`命令，将这个项目打包成 jar 部署到本地仓库中，提供让另一个服务调用。

7. 创建一个新的 SpringBoot web 项目`test-myjson-spring-boot-starter`，提供一个接口去访问。

```
package com.ziyou.test.controller;

import com.ziyou.starter.MyJsonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author ziyou<br>
 * <b>Date：</b>2019-07-20 19:04<br>
 * <b>Desc：</b>无<br>
 */
@RestController
public class MyJsonController {

    @Autowired
    private MyJsonService myJsonService;

    @RequestMapping(value = "tojson")
    public String getStr() {
        User user = new User();
        user.setName("dsfsf");
        user.setAge(18);
        return myJsonService.objToJson(user);
    }
}


```

8. application.properties 中配置。

```properties

server.port=8089
ziyou.json.name=java-geek-teck

```

9. pom.xml 中配置刚刚编写的 Starter。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.ziyou</groupId>
    <artifactId>test-myjson-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.ziyou</groupId>
            <artifactId>myjson-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>

</project>
```

10. 启动测试项目，打开浏览器访问接口可以看到如下效果。

![image-20190720221614055](http://justdojava.com/assets/images/2019/java/image_ziyou/starter2.png)

11. 分析：从结果中我们可以看到在 Starter 中定义的`MyJsonService`已经被成功的调用和执行。



### 小结

今天简单的跟大家分享了一下 SpringBoot Starter 的实现原理和简单应用，希望能帮助到大家。写这篇文章主要是因为这个问题我当时面试的时候被问到过，但是当时自己没有了解过，现在写出来一个是自己加深记忆，一个是帮助大家，希望大家如果在面试中被问到可以很好的表现。另外我们《 Java 极客技术》知识星球最近在分享我们团队录制的 SpringBoot 的视频，作为一部分内容的补充，欢迎大家进到星球共同学习。

对于今天的文章你有什么想要说的或者分享的吗？欢迎到 Java 极客技术的知识星球中跟我们一起交流。这里有着一批热爱技术的人在一起成长，欢迎你的加入。此外我们还有很多优质的电子书送给大家，欢迎关注公众号，在后台回复关键字：电子书，java， go，kafka。更多更好的内容我们在一直持续的更新，欢迎到星球跟我们一起进步。

