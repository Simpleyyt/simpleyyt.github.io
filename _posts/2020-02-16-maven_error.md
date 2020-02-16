---
layout: post
categories: Java
title: 程序员需要了解依赖冲突的原因以及解决办法
tagline: by 小黑
tags: 
  - 小黑
---
![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200216/Maven.png)

## 0x00. 前言

依赖冲突是日常开发中经常碰到的过程，如果运气好，并不会有什么问题。偏偏阿粉有点背，碰到好几次生产问题，排查一整晚，最后发现却是依赖冲突的引起的问题。

没碰到过这个问题同学可能没什么感觉，阿粉举两个最近碰到例子，让大家感受一些。

**例子 1：**

我们公司有个古老的业务基础包 A。B，C 业务依赖这个包。某个团队拷贝 A 的部分代码进行重构，**类名与路径完全一样**，然后重新打包成 D 发布。

一次业务改动，B 业务也引入了 D 包，测试环境运行的时候，一切 OK，但是在生产运行时，却抛出 `NoSuchMethodError`。

问题原因在于 B 业务依赖 A，D。而 A,D 存在两个同包同名类，运行的时候，具体加载谁，不同环境还真不一样。

**例子 2：**

A 业务使用 `Dubbo` 进行 `RPC` 调用， `Dubbo` 需要依赖 `javassist`。当前依赖关系为：

```log
A------->Dubbo------->javassist-3.18.1.GA
```

某次改动中引入另外一个第三方开源包，其依赖  `javassist-3.15.0-GA` 。生产发布的时候，将 `javassist-3.15.0-GA` 打包到应用中，由于生产环节为 JDK1.8，从而导致运行直接失败。

除了上述问题，依赖冲突还可能导致应用抛出  `ClassNotFoundException`，`NoClassDefFoundError` 等错误。

抛出错误这种情况还算好，还比较容易定位问题。怕就怕，不同版本同一个类内部逻辑不同，从而导致业务异常。这种问题，真的很让人抓狂，让人头秃。

![(http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200216/0082zybply1gbyaxes5fpj306n06n0sp.jpg)

仔细分析依赖冲突，主要可以分为两类：

- 项目同一依赖应用，存在多版本，每个版本同一个类，可能存在差异。
- 项目不同依赖应用，存在包名，类名完全一样的类。

下面我们分析一下依赖冲突产生的原因。

## 0x01. 依赖冲突原因

### 1.1 依赖机制

`Maven` 依赖分为两种情况，直接依赖与间接依赖，这个比较好理解，大家直接看图就好。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200216/0082zybply1gbxet3p0aoj30ma0dfq3y.jpg)



### 1.2 仲裁机制

如果 A 应用间接依赖多个 C 应用，且版本都不一样，Maven 将会通过仲裁机制选择：

- 优先按照依赖管理<dependencyManagement>元素中指定的版本声明进行仲裁时，下面的两个原则都无效了
- 短路径优先
- 若路径相同，将看 pom 中声明的顺序。

第一条原则，我们下面再说。

第二条原则，如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200216/0082zybply1gbyeh1ouvqj31ee0p2twm.jpg)

A 间接依赖两个版本 E，这种情况下，由于 A 到 E-1.0 路径最短，所以 A 中将会使用 E-1.0。

如果路径恰好一样，那么这种情况下 `Maven` 只能根据 `pom` 中的顺序，选择最先声明的，这也是个无奈的选择。

### 1.3 scope 属性

Maven 项目可以分为三个阶段：编译阶段，测试阶段，运行阶段了。通过 `scope` 属性，我们可以决定依赖应用是否参与以上阶段，也将会影响依赖传递。

`Maven` 提供 6 种 `scope` ：

- `compile`
- `provided`
- `runtime`
- `test`
- `system`
- `import`

**compile**

`compile` 是  `Maven` 默认属性，将会使依赖包参与项目的编译，测试，运行阶段。当然，项目打包之后将会包含该依赖。

**provided**

`provided` 意味着依赖仅参与项目编译，测试的阶段。若有如下依赖关系：

```log
A----->B----->C
```

C 的 `scope` 为`provided`，C 将会参与 B 的编译，测试阶段，但是 C 不会传递给 A。如果 A 运行过程需要 C，需要自己直接引入 C 依赖。典型如 `Servlet API`，因为 `Tomcat` 等容器内部会提供。

**runtime**

`runtime` 代表依赖不再参与项目编译阶段，只参与测试，运行阶段。

若依赖不参与编译阶段，这种情况 IDE 中是无法导入相应的类的。若存在依赖类，编译过程中将会报错。

典型的例子是 `JDBC` 驱动包，如 `mysql` :

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>6.0.6</version>
    <scope>runtime</scope>
</dependency>
```

> 知识点：这个好处在于，只能使用 `JDBC` 标准接口，这样就不会与特定的数据库绑定。后续若切换数据库，只需要更换 `pom`，然后修改相应的参数即可。

**test**

`test` 仅参与测试阶段的工作，典型的例子为 `junit`：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

**system**

`system` 与 `provided` 范围一致，只不过 `system` 需要使用 **systemPath** 属性指定本地路径，而 `provided` 将会从 `Maven` 仓库拉取。

**import**

`import`  比较特殊，不会参与以上阶段运行。其只能在 `dependencyManagement`下使用，且 `type` 需要为 `pom`。典型的例子为 Spring-boot 依赖。

```xml
    <dependencyManagement>
        <dependencies>
            <!-- Spring Boot -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.6.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

> 知识点：通过这种方式，解决单继承问题，也可以更好将依赖分类。

另外 `Maven scope` 将会影响依赖传递。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200216/0082zybply1gbyg096fimj326a0bi14h.jpg)

> 如果依赖关系为： **A--->B--->C**,A 依赖 B，B 依赖 C。最左列代表 B 的 `scope` 属性，第一行代表 C 的 `scope` 属性

如上所示，当 C 的 `scope` 为 **provided/test**, C 只在 B 中起作用，不会通过间接依赖传递给 A。

当且仅当 B 的 `scope` 为 `compile`，且 C `scope` 为 `runtime` ，A 将会间接依赖 C，且 `scope` 为 `runtime`。其他情况下，C 的 scope 将会与 B 的 scope 一致。

## 0x02. 解决冲突的方法

### 2.1 使用 Maven 属性控制依赖传递

依赖冲突时，根据错误日志，定位到冲突类，定位相应 `jar` 包，最后通过 **excludes** 排除相应的包。

另外可以结合 **IDEA  Maven Helper** 插件，主动检查冲突依赖，提前排除。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200216/0082zybply1gbyk50a2n9j31y40mke0j.jpg)

通过插件，我们可以清晰看到冲突包，以及依赖路径，还有相应的 `Scope`。

除了排除依赖，我们可以通过合理的设置 `scope` 属性，不让依赖传播下去。比如说，A 需要是使用 `Spring-beans` 包中某些类。如果其他项目铁定会使用 Spring，那么我们可以将 A 中 `Spring-beans` `scope` 设置为 `provided`，让其他项目自己选择引入 `Spring-beans` 的版本。

> 这个适合公共基础包，其他包不要随便使用`provided`，若使用一定要写清楚，使用过程中需要引入的依赖。

以上方法虽然治标，但是不治本。如果想依赖冲突不发生，我们需要提前建立一定的规范，团队一起遵守，才能有效避免该类问题。

1. 应用项目中使用  `dependencyManagement` 统一管理基础依赖，定义统一的版本，如常用中间包，工具包，日志包。
2. 二方包中不要引入无关的依赖，做到尽量少的依赖。团队开发中，比较常见情况是二方包继承公共的父 pom，从而导致继承许多无相关的依赖，这种情况可以单独管理。
3. 二方包做好向下兼容，不要随意改动现有类名，方法名，字段名。
4. 项目应用上线之前，将 `snapshot` 替换成正式版本。虽然 `snapshot` 修改起来很方便，但是正因为这个特性，可以被随便修改。如果某次生产打包发布不注意，就会引入。
5. 二方包不要使用同一个包名，类名。一般来说，团队开发中，包名，类名一样概率比较小。这种比较容易出现在一些重构项目，复制原来类，重构打包发布。对于情况下可以修改包名。如 `cmomon-lang3` 是 `common-lang` 升级版， `cmomon-lang3` 包名为 org.apache.commons.lang3,而 `common-lang` 包名为 `org.apache.commons.lang`

## 0x03. 总结

如果我们把 `NPE` 问题当做新手村普通怪物，那么依赖冲突问题就是人马这种精英怪。刚开始遇到，我们会被虐的比较惨。只有我们不断升级，学习掌握技巧，然后才能可以从容不迫解决。

> ps:塞尔达中，你们第一次遇见人马，打了几次？阿粉记得那天整整从晚上九点打到凌晨两点，就是打不过啊~

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200216/Maven.png)

## 0x04. 帮助文档

[Maven Dependency Scopes](https://howtodoinjava.com/maven/maven-dependency-scopes/)

[Maven optional关键字透彻图解](https://juejin.im/post/5dc0c36be51d456e35627114)

[这篇文章写的很好，大家可以看下。，重新看待Jar包冲突问题及解决方案](https://www.jianshu.com/p/100439269148)

[包管理原则](https://juejin.im/post/5e4784536fb9a07c9070d142)