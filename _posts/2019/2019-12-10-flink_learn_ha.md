---
layout: post
categories: Flink
title: Flink 基础学习(八) 手把手教你搭建伪集群 HA
tags:
  - 惊奇
---

# 1 前言

前面理论性的知识是不是有点太“干货”，所以来点实战性的内容吧，这次记录了如何搭建高可用的 `Flink` 集群。

在正式配置前，来讲下**为何要配置高可用（`High Availability`）**

目前越来越多公司的线上应用，都采用的是分布式架构（一主多从），从而避免单点故障引起的服务不可用。

**而在 `Flink` 中，同样也有集群保障服务的高可用，任何时候都有一个主 `JobManager` 和多个备 `JobManager`，当前主节点宕机后，立刻会有备 `JobManager` 当选成为主 `JobManager` ，接管集群，让每个作业继续正常进行。**

出于这样的考虑，在 `Flink` 应用上线前，需要有这样一个集群来进行保障，下面就来看下该如何进行配置吧。

<!--more-->

---

# 2 前置工作

## 2.1 机器准备

在本次搭建前，我通过了本地单机模拟，监听了 8081 和 8082 端口，启动了两个 `JobManager` 和三个 `TaskManager`。

| 序号 | IP        | 启动进程                | UI PORT |
| ---- | --------- | ----------------------- | ------- |
| 1    | 127.0.0.1 | JobManager、TaskManager | 8081    |
| 2    | 127.0.0.1 | JobManager、TaskManager | 8082    |
| 3    | 127.0.0.1 | TaskManager             | /       |

一般来说，`JobManager` 任务调度器所在的机器配置可以低一些，而 `TaskManager` 任务执行者所在的机器配置会高一些，所以基于上述机器的配置，请各位使用时进行合理分配。

### 2.1.2 机器之间 SSH 免密登录

**本质上，我们只需要修改一份配置，然后将修改后的 `Flink` 包通过 `SCP` 命令传送到其它机器上。**

接着在单台服务器上启动，它会根据 `masters` 和 `slaves` 文件找到对应的服务器，登录上去，判断启动 `JobManager` 、`TaskManager` 或者两者都启动。

**总结一下机器之间 SSH 免密登录的要点：**

**1. 每台机器生成公私钥：`ssh-keygen` 命令**

**2. 将本机生成的公钥(`id_rsa.pub`)拷贝到其它机器的认证列表中(`authorized_keys`)**

   使用该命令 `ssh-copy-id -i id_rsa.pub root@118.25.xxx.xxx`

**3. 重复上述命令，将本机的公钥添加到集群中的每一台服务器上**

**4. 免密登录所在的服务器，就是待会我们启动 `./start_cluster.sh` 脚本所在的服务器哟**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/ssh_without_code.png)

参考上面的图片，将公钥拷贝到远程服务器后，我们就能通过 `ssh root@xxx.xxx.xx.xx`，免密码登录到远程服务器啦。

---

## 2.2 安装 Zookeeper

为了启用 `JobManager` 高可用这个功能，需要在 `flink_conf.yaml` 中将 **高可用模式** 设置为 `zookeeper`

`Flink` 利用 `Zookeeper` 管理 `JobManager` 和 `TaskManager`，进行分布式协调（熟悉 `Dubbo` 系的小伙伴，应该对我们的 `Zookeeper` 注册中心感到熟悉）。

`Zookeeper` 是独立于 `Flink` 的一项服务，该服务通过节点选举和轻量级一致状态存储提供高度可靠的分布式协调。

待会再具体讲配置文件的参数，这里是提醒大家一定要安装和启动 `Zk` 服务

具体可以看下 [Zookeeper 单机环境和集群环境搭建](https://github.com/heibaiying/BigData-Notes/blob/master/notes/installation/Zookeeper单机环境和集群环境搭建.md)

---

## 2.3 安装 Hadoop

在高可用模式下，需要使用 **分布式文件系统** 来持久化存储 `JobManager` 的元数据（`meta data`），常用的是 `HDFS`，于是 `Hadoop` 也得提前安装，在 `Hadoop` 配置上，我花了半天的功夫，才将它从 0 搭建了起来，这里得好好记录一下。

提前说明一下，下面出现域名的地方，我基本都以 `IP` 地址的形式展示出来，避免大家忘了在 `/etc/hosts` 文件中进行 `DNS` 解析，以及设定的文件目录是 `/tmp`，也是避免启动前忘了给文件夹赋予读写权限。

**（懂我意思了吧，要修改成域名和文件目录地址，记得添加 `DNS` 解析和  `chmod` 赋予权限哟）**

进入 `${HADOOP_HOME}/etc/hadoop` 目录下，修改以下配置文件

**1. core-site.xml**

```xml
<configuration>
    <property>
        <!--指定 namenode 的 hdfs 协议文件系统的通信地址-->
        <name>fs.defaultFS</name>
        <value>hdfs://127.0.0.1:9000</value>
    </property>
    <property>
        <!--指定 hadoop 集群存储临时文件的目录-->
        <name>hadoop.tmp.dir</name>
        <value>/tmp/hadoop/tmp</value>
    </property>
</configuration>
```

**2. hdfs-site.xml**

```xml
<configuration>
    <property>
      <!--namenode 节点数据（即元数据）的存放位置，可以指定多个目录实现容错，多个目录用逗号分隔-->
    	<name>dfs.namenode.name.dir</name>
    	<value>/tmp/hadoop/namenode/data</value>
		</property>
		<property>
      <!--datanode 节点数据（即数据块）的存放位置-->
    	<name>dfs.datanode.data.dir</name>
    	<value>/tmp/hadoop/datanode/data</value>
		</property>
    <property>
      <!-- 手动设置 DFS 监听端口号 -->
  		<name>dfs.http.address</name>
 	 		<value>127.0.0.1:50070</value>
		</property>
</configuration>
```

**3. yarn-site.xml**

```xml
<configuration>
    <property>
        <!--配置 NodeManager 上运行的附属服务。需要配置成 mapreduce_shuffle 后才可以在 Yarn 上运行 MapReduce 程序。-->
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <!--resourcemanager 的主机名-->
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
</configuration>
```

**4. mapred-site.xml**

```xml
<configuration>
    <property>
        <!--指定 mapreduce 作业运行在 yarn 上-->
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

单机启动，定位到 `${HADOOP_HOME}/sbin` 目录，输入以下命令 `./start-all.sh` 启动 `Hadoop`

除了通过 `${HADOOP_HOME}/logs` 目录下的日志查看，还可以通过以下三个端口检测 `Hadoop` 是否成功启动：

- **Resource Manager: http://localhost:50070**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/hadoop_resource_manager_overview.png)

- **JobTracker: http://localhost:8088**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/hadoop_job_tracker_overview.png)

- **Specific Node Information: http://localhost:8042**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/hadoop_node_overview.png)



## 2.4 安装 Hadoop 踩坑记

- **localhost port 22 : Connection refused**

提示这个错误是因为 `Hadoop` 启动时需要远程登录 `ssh localhost`，需要进入 `/root/.shh` 目录，将公钥加入到认证列表中：

```shell
$ cat id_rsa.pub >> authorized_keys
```

如果是 `Mac` 用户，你还需要在小齿轮设置中的【共享】模块勾选【远程登录】这一个选项


- **hadoop无法访问50070端口**

查看日志将会看到如下提示信息是 

```log
java.net.ConnectException: Call From yejingqidembp.lan/192.168.199.232 to localhost:9000 failed on connection exception: java.net.ConnectException: Connection refused;
```

**实际原因是：`Resource Manager` 没有正常启动**

解决方案：

1. 在 `hdfs-site.xml` 中添加 `namenode` 初始化默认端口

```xml
<!-- dfs ip 地址请按实际使用时设置 -->
<property>
  <name>dfs.http.address</name>
  <value>127.0.0.1:50070</value>
</property>
```

2. 重新格式化 `namenode`，然后再启动 `hadoop`

```shell
  $ cd ${HADOOP_HOME}
  $ ./bin/hdfs namenode -format
  $ ./sbin/start-all.sh
```

经过上面的操作，你就能够成功启动单点 `Hadoop`，如果想要启动集群版的，请参考这篇文章：[Hadoop 集群环境搭建](https://github.com/heibaiying/BigData-Notes/blob/master/notes/installation/Hadoop%E9%9B%86%E7%BE%A4%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.md)



---

# 3 Flink Standalone Cluster HA

使用 `Zookeeper` 作为注册中心，进行资源的分布式协调，实现集群高可用

## 3.1 Flink 配置

**首先 `Flink` 的配置文件路径在 `${FLINK_HOME}/conf`，推荐各位使用 `VSCode` 这个编辑器打开，可以在左侧查看文件目录树的结构，界面好看，功能强大**（这大概是我抛弃了 `Sublime` 的原因哈哈哈）

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/flink_ha_config_overview.png)

上图是本次要修改的三个核心配置文件，关于高可用 `Zookeeper` 的设定和主从节点的配置。


- **flink-conf.yaml**

```yaml
# High Availability
# 高可用模式设置成 zk
high-availability: zookeeper
# 持久化存储 JobManager 元数据的地址
high-availability.storageDir: hdfs://127.0.0.1:9000/flink/ha/
# zk 仲裁集群地址，实际使用时建议三台以上
high-availability.zookeeper.quorum: 127.0.0.1:2181
```

- **master**

配置了两个 `JobManager`，由于是本地演示，所以监听了 8081 和 8082 两个端口，启动两个 `WEBUI:PORT`
```shell
127.0.0.1:8081
127.0.0.1:8082
```

- **slaves**

配置了三个 `TaskManager`

```shell
127.0.0.1
127.0.0.1
127.0.0.1
```


**其它更多可选配置，可以参考 [Flink 官方配置介绍](https://ci.apache.org/projects/flink/flink-docs-stable/ops/config.html)，关于 `JobManager` 堆大小分配和 `Slot` 分配等配置，本次使用的是默认的，使用的是 1G 和 1 Slot**

## 3.2 分发到其它服务器

前面也提到过，集群中的配置，只需要在一台服务器上修改好，打通机器间的免密登录，在分发出去的机器上启动集群就可以了，所以来看下如何将文件夹分发到其它服务器吧

```shell
$ scp -r /usr/local/flink-1.9.1 root@127.0.0.2:/usr/local
$ scp -r /usr/local/flink-1.9.1 root@127.0.0.3:/usr/local
```

假设我们有三台服务器，127.0.0.1 上是我们修改过的包，通过上面的 `scp` 命令就能将文件夹发送到集群中的服务器。

**不过本次演示是单机，不需要分发，我们可以直接进行下一步，但在多机器的情况下，保持路径一致，分发后才进行下一步。**

## 3.3 启动集群

在启动前，请确保 `Zookeeper` 和 `Hadoop` 都已成功启动，进入 `${FLINK_HOME}/bin` 目录

```shell
$ ./start-cluster.sh
Starting HA cluster with 2 masters.
Starting standalonesession daemon on host yejingqideMBP.lan.
[INFO] 1 instance(s) of standalonesession are already running on yejingqideMBP.lan.
Starting standalonesession daemon on host yejingqideMBP.lan.
Starting taskexecutor daemon on host yejingqideMBP.lan.
[INFO] 1 instance(s) of taskexecutor are already running on yejingqideMBP.lan.
Starting taskexecutor daemon on host yejingqideMBP.lan.
[INFO] 1 instance(s) of taskexecutor are already running on yejingqideMBP.lan.
Starting taskexecutor daemon on host yejingqideMBP.lan.
```

## 3.4 验证集群成功启动

通过 `JPS` 命令，查看当前运行的 `Java` 进程

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/flink_ha_jps_process.png)

红框中圈出的是属于 `Flink` 的进程，包括两个 `JobManager` 和 三个 `TaskManager`。

同样，我们还可以通过界面 `UI` 查看：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/flink_ha_web_ui_overview.png)

**左边是 `JobManager1`，端口号是 8081，在 `JobManager` 菜单中也能看到我们配置的 `Zookeeper` 信息，右边是 `JobManager2`，端口号是 8082，在 `TaskManager` 菜单可以看到启动了三个 `Worker`，以上结果符合我们在 `master` 和 `slaves` 文件上的配置~**

## 3.5 验证集群功能

前面也提到过，集群帮助我们出现单点故障导致整个服务不可用的情况，**通过查看线程信息和 `JobManager` 菜单下的 `logs` 日志，发现了 8081 是我们的 `Master` 节点。**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/flink_ha_master_node.png)

接着就是关闭 8081 进程  `kill -9 processID` 来一下，刷新 8081 的页面，将会变成灰色不可用，这样我们的主节点就“宕机了”

重新启动 8081，`./jobManager.sh start`，查看输出日志：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/ha/flink_ha_alter_node.png)

可以发现 8081 已经没有 `grant leadership` 的标志，我们的 8082 成功当选 `master` 节点，继续承担主节点的职责，完成资源任务的调度。


## 3.6 Flink 启动踩坑

在启动过程中，发现 `Cluster HA` 并没有成功启动，通过查看 `${FLINK_HOME}/log` 下的日志，发现了这个问题：

```log
- Shutting StandaloneSessionClusterEntrypoint down with application status FAILED. Diagnostics java.io.IOException: Could not create FileSystem for highly available storage (high-availability.storageDir)
	at org.apache.flink.runtime.blob.BlobUtils.createFileSystemBlobStore(BlobUtils.java:119)
...
Caused by: org.apache.flink.core.fs.UnsupportedFileSystemSchemeException: Could not find a file system implementation for scheme 'hdfs'. The scheme is not directly supported by Flink and no Hadoop file system to support this scheme could be loaded.
```

**同样，查阅到相关资料后，发现是默认下载的 `Flink` 缺少 `Hadoop` 的相关依赖，需要去[官网](https://flink.apache.org/downloads.html)下载相应的 `Optional components` 依赖，例如我下载的是这个 [flink-shaded-hadoop-2-uber-2.8.3-7.0.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-shaded-hadoop-2-uber/2.8.3-7.0/flink-shaded-hadoop-2-uber-2.8.3-7.0.jar)**

**下载完成后，将它扔入 `${FLINK_HOME}/lib` 目录下，然后重新启动就可以了。**

---
# 4 总结

**本篇介绍了如何启用 `Flink Standalone Cluster HA`，由于踩坑巨多才成功搭建，这里总结一下要点：**

1. 服务器准备

   将前面出现的 `ip` 地址替换成自己集群中的服务器

2. 免密登录其它服务器

   参考 `ssh-copy-id `命令 和 `authoried_keys` 文件作用

3. 安装 `Zookeeper` 和 `Hadoop`

4. 配置 `Hadoop`

   这一步是坑最多的，`Mac brew` 安装的有点坑，所以十分建议下载官网的包，然后修改下载后的包配置，具体看前面的配置

5. 下载 `Flink`

   同样也是建议下载官网的包，然后记得下载 `Hadoop` 依赖包，扔入 `${FLINK_HOME}/lib` 目录下

6. 配置 `Flink`

   `flink-conf.yaml`、`masters` 和 `slaves`

7. 启动 `Flink` 集群和验证

**好了，以上就是本次的经验总结，介绍了如何搭建伪集群高可用 `HA`，跟上面说的，将出现的 `IP` 换成多台服务器就能够实现真正的高可用部署**

如有其它学习建议或文章不对之处，请与我联系~（可在 `Github` 中提 `Issue` 或掘金中联系）

---
# 5 项目地址

[https://github.com/Vip-Augus/flink-learning-note](https://github.com/Vip-Augus/flink-learning-note)

```sh
git clone https://github.com/Vip-Augus/flink-learning-note
```

---

# 6 参考资料

1、[生产级别flink HA集群搭建和调优](http://moheqionglin.com/site/serialize/02006013006/detail.html)

2、[Linux主机之间ssh免密登录配置](https://my.oschina.net/shipley/blog/1507337)

3、[JobManager High Availability (HA)](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/jobmanager_high_availability.html)

4、[hadoop无法访问50070端口](https://blog.csdn.net/Neone__u/article/details/53741786)

5、[在Mac下安装Hadoop的坑](https://www.jianshu.com/p/d19ce17234b7)

6、[Flink 官方配置介绍](https://ci.apache.org/projects/flink/flink-docs-stable/ops/config.html)











