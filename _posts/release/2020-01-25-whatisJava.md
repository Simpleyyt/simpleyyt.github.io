---
layout: post
title:  Java平台概述
categories: Java
tags:
	- cxuan

---

阿粉？阿粉？阿粉？阿粉在哪里，项目经理今天发现阿粉没来，一时间很生气，心里盘算回来一定要让阿粉知道自己不是好惹的？可是阿粉去了哪里呢？阿粉受不鸟这个公司了，太 TM XXX了，阿粉出来面试了！！！阿粉心想一定要找到一个好工作！！！

<!--more-->

## JRE 和 JDK

Oracle 在 JavaSE 系列上提供了两种主要的软件产品他们分别是 `JRE` 和 `JDK`

### Java SE Runtime Environment (JRE)

首先 `JRE` 是 **Java Runtime Environment**，是 Java `运行时环境`。实际上，JRE 是一种旨在运行在其他软件的软件。作为 Java 的运行时环境，JRE 包括了基本的 Java 类库，Java 类加载器和 Java 虚拟机。

在这个 Java 系统中

* 类加载器负责正确加载类并将它们与核心 Java 类库进行连接。
* JVM 负责确保 Java 应用程序在设备或者云环境中有足够的空间资源来运行 Java 程序。
* JRE 主要的作用是为其他组建提供了一个`容器（Container）`，负责协调安排他们的活动。

### Java SE Development Kit (JDK)

JDK 是 Java 完整的 SDK，它包括任何 JRE 中拥有的东西，另外，它还包括 JRE 所没有的命令行工具例如 Java 编译器 `javac`， `javadoc` 和 `jdb`，它能够用来创建新的项目和编译程序，只靠 JRE 是无法创建新项目的

通常，如果只关心在计算机上运行 Java 程序，则只会安装 JRE，这就是你所需要的。另一方面，如果你打算进行一些 Java 编程，则需要安装JDK。

有时，即时你不打算在计算机上进行任何 Java 开发，你仍然要安装 JDK。比如说，如果你想用 JSP 开发一个 web 应用程序，从技术上来讲，你是在应用服务器中运行 Java 程序，那你为什么需要安装 JDK 呢？因为应用程序会将 JSP 转换为 Java Servlet，并且需要使用 JDK 来编译 servlet 等。

## Java 编程语言

Java 编程语言作为当今异常火爆的编程语言，它崛起的因素是什么呢？

我想或许可以从下面这几个切入点进行了解

* `简单`，Java 会让你的工作变得更加轻松，使你把关注点放在主要业务逻辑上，而不必关心指针、运算符重载、内存回收等与主要业务无关的功能。
* `便携性`，Java 是平台无关性的，这意味着在一个平台上编写的任何应用程序都可以轻松移植到另一个平台上。
* `安全性`， 编译后会将所有的代码转换为字节码，人类无法读取。它使开发无病毒，无篡改的系统/应用成为可能。
* `动态性`，它具有适应不断变化的环境的能力，它能够支持动态内存分配，从而减少了内存浪费，提高了应用程序的性能。

* `分布式`，Java 提供的功能有助于创建分布式应用。使用`远程方法调用（RMI）`，程序可以通过网络调用另一个程序的方法并获取输出。您可以通过从互联网上的任何计算机上调用方法来访问文件。这是革命性的一个特点，对于当今的互联网来说太重要了。

* `健壮性`，Java 有强大的内存管理功能，在编译和运行时检查代码，它有助于消除错误。

* `高性能`，Java 最黑的科技就是字节码编程，Java 代码编译成的字节码可以轻松转换为本地机器代码。通过 JIT 即时编译器来实现高性能。
* `解释性`，Java 被编译成字节码，由 Java 运行时环境解释。

* `多线程性`，Java支持多个执行线程（也称为轻量级进程），包括一组同步原语。这使得使用线程编程更加容易，Java 通过管程模型来实现线程安全性。

## Java 虚拟机

`JVM ` 有两个主要的功能： 允许 Java 程序在任何设备或者操作系统上运行，也就是 `compile once,run anywhere`，以及管理和优化动态内存分配。 Java 诞生在 1995 年（我诞生的这一年果然不同寻常），所有的计算机程序都会运行在一个特定的操作系统上，程序运行内存由开发人员手动管理，所以给 JVM 的开发带来了启示作用

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/01.png)



* 技术方面的启示： JVM 是一个特殊的软件程序，它能够执行代码和提供运行时代码环境
* 日常运行的启示： JVM 是我们运行 Java 程序的方式，我们配置 Java 环境变量，然后在执行期间依靠它来管理程序资源。

Java 虚拟机可以移植到不同的平台，以提供与硬件和操作系统无关的功能。

## Java 基础库

提供 Java 平台基本功能的类和接口（基于 JDK 1.8）

**Java.applet**

Java Applet 就是用 Java 语言编写的一些小应用程序，它们可以直接嵌入到网页中，并能够产生特殊的效果（小程序入门可以参考 https://blog.csdn.net/qq_26024867/article/details/82558385）例如下面

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/02.png)



Java.applet 现在已经几乎不再使用，为什么不再使用了呢？你这个问题让鸭鸭陷入了沉思，为什么不用了？想不出来，求助百度和谷歌吧，我认为这个回答还是比较可靠的

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/03.png)



**Java.awt**

java.awt 是一个软件包，包含用于创建用户界面和绘制图形图像的所有分类。Java.awt 包我们也几乎不再使用。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/04.png)

**Java.beans**

bean 这个词想必我们不是刚学 Java 的时候听到的吧，应该是接触 Spring 才认识到 bean 这个词吧，bean 在 Java 中就是 java 的类，或者说，就是 Java 语言的组件，充分体现了 Java 语言平台独立和面向对象编程的优势。

所有的 Swing 和 AWT 类都是 JavaBean。 GUI 组件是理想的 JavaBean。Java.beans 包括属性、事件、方法和持久化组件。

**Java.io**

Java 的核心库 `java.io` 提供了全面的 IO 接口。包括：文件读写、标准设备输出等。Java 中 IO 是以流为基础进行输入输出的，所有数据被串行化写入输出流，或者从输入流读入。

Java 流的分类

按流向可以分为

* 输入流：程序可以从外部读取数据。
* 输出流：程序能向其中写入数据的流。

按数据传输单位分

* 字节流：以字节为单位传输的流。
* 字符流：以字符为单位传输的流。

按照功能分

* 节点流：用于直接操作目标设备的流。
* 过滤流：是对一个已存在的流的链接和封装，通过对数据进行处理为程序提供功能强大、灵活的读写功能。

下面是一个 IO 流的最全分类

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/09.png)



**Java.lang**

`java.lang` 包是 java 语言的核心，它提供了 java 中的基础类。包括基本 Object 类、Class 类、String 类、基本类型的包装类、基本的数学类等等最基本的类。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/05.png)

**Java.math**

java.math 包提供用于执行任意精度整数算法 (BigInteger) 和任意精度小数算法 (BigDecimal) 的类。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/06.png)

**Java.net**

java.net 软件包包含类和接口，这些类和接口为 Java 中的网络提供了强大的基础结构。这些包中的许多类提供 Java 中的 socket 通信。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/07.png)

**Java.nio**

java.nio全称 `java non-blocking IO`，是指 jdk1.4 及以上版本里提供的新 api（New IO） ，为所有的原始类型（boolean类型除外）提供缓存支持的数据容器，使用它可以提供非阻塞式的高伸缩性网络。

**Java.rmi**

rmi 的全称是 `Remote Method Invocation`，远程方法调用，是在 JDK1.2 中实现的，它大大增强了 Java 分布式开发的能力

**Java.security**

java.security 是 Java 中为安全框架提供的类和接口。

**Java.sql**

提供用于使用 Java 编程语言访问和处理存储在数据源（通常是关系数据库）中的数据的API。

**Java.text**

提供以与自然语言无关的方式来处理文本、日期、数字和消息的类和接口。

**Java.time**

提供了用于日期时间处理的 API

**Java.util**

Java.util 也是 Java 核心 API 中非常重要的接口，它包含集合框架、遗留的 collection 类、事件模型、日期和时间设施、国际化和各种实用工具类（字符串标记生成器、随机数生成器和位数组、日期Date类、堆栈Stack类、向量Vector类等）。集合类、时间处理模式、日期时间工具等各类常用工具包

## Java 应用

我们知道 Java 从诞生起一直流行到现在，那么 Java 这门编程语言能够做什么呢？Java 可以在不同的领域中使用，下面是它的应用领域

* 银行业务：处理交易管理。
* 零售：你在商店/餐厅看到的计费应用程序完全用 Java 编写。
* 信息技术：Java 旨在解决实现依赖性。
* Android：应用程序用 Java 编写或使用Java API。
* 金融服务：用于服务器端应用程序。
* 股票市场：编写关于应投资哪家公司的算法。
* 大数据：Hadoop MapReduce 框架是使用 Java 编写的。
* 科学与研究社区：处理大量数据。

![](http://www.justdojava.com/assets/images/2019/java/image-cxuan/java/interview01/08.png)



文章参考

https://stackoverflow.com/questions/1906445/what-is-the-difference-between-jdk-and-jre

https://docs.oracle.com/javase/specs/jls/se8/jls8.pdf

https://www.javaworld.com/article/3272244/what-is-the-jvm-introducing-the-java-virtual-machine.html

https://baike.baidu.com/item/java.applet/5179357?fr=aladdin

https://baike.baidu.com/item/java.io/5179754?fr=aladdin

https://www.edureka.co/blog/what-is-java/