---
layout: post
categories: Flink
title: Flink 基础学习(十) 使用 Influxdb 和 Grafana 监控 Flink
tags:
  - 惊奇
---


# 1 前言

一个项目的良好运行，依赖于系统监控和及时报警，对比于登陆到服务器进行查看服务器负载，使用图形化界面会更好的进行查看，于是这里记录一下搭建监控的步骤。

<!--more-->

# 2 相关软件安装和启动

## 2.1 Influxdb

> InfluxDB是一个用于存储和分析时间序列数据的开源数据库。

该数据库用来存储我们 `Flink` 运行时的指标信息 `metrics`，具体关于 `Influxdb` 的概念，可以参考中文文档：[InfluxDB中文文档](https://jasper-zhang1.gitbooks.io/influxdb/content/)

在 `mac` 下进行安装比较简单，使用 `Homebrew` 就能进行简单的单机安装和使用

简单记录如下：

```shell
// 安装
$ brew install influxdb
// 启动
$ brew services start influxdb
# 进入 Influxdb
$ influxdb
```

**建数据库和建立授权的用户信息：**

以下创建的信息将会在待会的 `Flink` 中用到，所以可以参考后面的配置，随意替换成自己想要填写的

```shell
# 进入到 influxdb
$ influxdb
# 创建一个授权的用户和对应的密码
> create user "jingqi" with password '123456' with all privileges
# 创建一个待会用来存放数据的数据库
> create database test
# 使用上面创建的数据库
> use test
```

---
## 2.2 Grafana

> The analytics platform for all your metrics
> 您所有指标的分析平台。从官网介绍中可以看出，Grafana 是个优秀的可视化监控工具。

同样，安装起来也很简单，参考官网即可：[Grafana :Installing on macOS](https://grafana.com/docs/grafana/latest/installation/mac/)

简单记录如下：

```shell
$ brew install grafana
$ brew services start grafana
```

通过上述命令安装后，默认监听本地的 3000 端口，待会配置好 `Flink` 参数后，再一起来看监控 `UI`。

---
# 3 修改 Flink 配置文件

定位到该目录：`${FLINK_HOME}/conf`，找到 `flink-conf.yaml` 文件，在里面添加以下 `influxdb` 相关的配置：

```yaml
#===================
# Metrics Config 
#===================
metrics.reporter.influxdb.class: org.apache.flink.metrics.influxdb.InfluxdbReporter
metrics.reporter.influxdb.host: localhost
metrics.reporter.influxdb.port: 8086
metrics.reporter.influxdb.db: test
metrics.reporter.influxdb.username: jingqi
metrics.reporter.influxdb.password: 123456
```

解释下上面配置的对应含义：

- **class：固定值，表示使用 InfluxdbReporter 驱动**
- **host：需要监听的 InfluxDB 的 host 地址**
- **port：这个是 InfluxDB 的端⼝号，默认是 8086**
- **db：表示你要将 metrics 数据存⼊入到 InfluxDB 的哪个数据库**
- **username：InfluxDB 的验证用户名**
- **password：InfluxDB 的密码**

参考 [metrics 文档](https://ci.apache.org/projects/flink/flink-docs-stable/monitoring/metrics.html)，里面提到关于依赖包的配置：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/monitoring/influxdb_lib_copy.png)

所以，如果你跟我一样，使用的是上面 `brew` 默认安装流程，还需要将 `${FLINK_HOME}/opt/` 目录下的 `flink-metrics-influxdb-1.9.0.jar` 放入到 `${FLINK_HOME}/lib` 目录中，不然你将会看到下面的报错信息：

```xml
java.lang.ClassNotFoundException: org.apache.flink.metrics.influxdb.InfluxdbReporter
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:264)
```

---
## 3.1 启动 Flink

启动 `Influxdb` 数据库和修改 `flink-conf.yaml` 配置文件后，我们就可以启动 `Flink`，来验证一下时序数据库中存了什么数据。

```shell
# 启动 flink
$ cd ${FLINK_HOME}/bin
$ ./start-cluster.sh
# 进入 influxdb
$ influxdb
> use test
> show measurements
...
> select * from jobmanager_Status_JVM_CPU_Load;
...
```

**measurements** 有点像 `MySQL` 中的 `tables` 概念，然后单独查询某张表的内容，可以看到每个 `field` 下对应着各自的 `value`：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/monitoring/influxdb_measurements.png)

上图展示的是，在 `Flink` 成功启动后，通过前面放入的 `Reporter` 驱动，往 `Influxdb` 源源不断写入数据。

---
# 4 Grafana 配置

在游览器中打开 `http://localhost:3000`，使用默认登录账号密码 `admin/admin` 就能进入到 `Grafana` 的监控界面，先将 `Influxdb` 数据源加上后，再点击 + 号就能开始我们的监控配置了：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/monitoring/influxdb_grafana_add_source.png)

---
## 4.1 添加 Influxdb 数据源

点击上面的大红框，像个数据库的标记，然后选择 `Influxdb` 的选项，接着在里面设置前面我们设置过的数据库用户名密码:

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/monitoring/influxdb_datasource.png)

上图中，三个红框是必填的，填写好之后，点击左下角的保存按钮就成功添加了 `Influxdb` 数据源了。

---
## 4.2 添加指标

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/monitoring/grafana_add_query.png)

监控大盘 `dashboard` 是由多个面板 `panel` 组成的，所以在创建大盘后，就要开始添加面板，先后点击上面两个红框中的选项。

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/monitoring/grafana_query_detail.png)

进入 `panel` 配置后，上图左侧的菜单按钮，可以设置查询语句、可视化界面的设置以及通用设置，**然后具体的查询语句 `Query`，可以参考上图展示的，通过下拉选择进行配置查询条件。**

接着单个面板就配置完成了，可以继续上述步骤，不断添加面板来进行监控不同指标的数据。由于应用刚启动，采集的数据不多，所以需要等待一段时间后，数据库中的数据才会多起来，然后整体的监控界面就会丰富起来~

---
# 5 总结

1. 安装和启动 `Influxdb` 和 `grafana`
2. 填写 `Flink` 配置文件，启动
3. 在 `Grafana` 配置 `Influxdb` 数据源信息
4. `Add Query` 添加监控的指标

**本次简单记录了搭建 `Flink` 监控的流程，对于其中的指标具体含义还不太清晰，抱着先会用，再去深入学习的态度，在这里继续挖下一个坑~**

如有其它学习建议或文章不对之处，请与我讨论吧~

---
# 6 项目地址

[https://github.com/Vip-Augus/flink-learning-note](https://github.com/Vip-Augus/flink-learning-note)

```sh
git clone https://github.com/Vip-Augus/flink-learning-note
```

---
# 7 参考

1. [使用InflubDB和Grafana监控Flink](https://www.cnblogs.com/createweb/p/11636762.html)
2. [Influxdb 介绍与使用](https://segmentfault.com/a/1190000015721780)
3. [InfluxDB中文文档](https://jasper-zhang1.gitbooks.io/influxdb/content/)
4. [Grafana :Installing on macOS](https://grafana.com/docs/grafana/latest/installation/mac/)











