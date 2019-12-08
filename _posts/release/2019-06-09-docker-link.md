---
layout: post
categories: Docker系列
title: Docker 容器连接[Docker 系列-7]
tagline: by 江南一点雨
tags: 
  - 江南一点雨
---

数据卷容器以及和大家聊过了，本文我们再来看看使用数据卷容器实现数据的备份与恢复，然后再来看看容器的连接操作。

<!--more-->

利用数据卷容器可以实现实现数据的备份和恢复。

# 数据备份与恢复

## 备份

数据的备份操作很容易，执行如下命令：  

```
docker run --volumes-from mydata --name backupcontainer -v $(pwd):/backup/ ubuntu tar cvf /backup/backup.tar /usr/share/nginx/html/
```  

命令解释：

1. 首先使用 `--volumes-from` 连接待备份容器。
2. `-v` 参数用来将当前目录挂载到容器的 `/backup` 目录下。
3. 接下来，将容器中 `/usr/share/nginx/html` 目录下的内容备份到 `/backup` 目录下的 `backup.tar` 文件中，由于已经设置将当前目录映射到容器的 `/backup` 目录，因为备份在容器 `/backup` 目录下的压缩文件在当前目录下可以立马看到。  

执行结果如下：

![19-1](/assets/images/2019/java/image_javaboy/0530/19-1.png)  

备份完成后，在当前目录下就可以看到 `/backup` 文件，打开压缩文件，发现就是 `/usr/share/nginx/html` 目录及内容。

## 恢复

数据的恢复则稍微麻烦一些，操作步骤如下：

### 创建容器

首先创建一个容器，这个容器就是要使用恢复的数据的容器，我这里创建一个 nginx 容器，如下：

```
docker run -itd -p 80:80 -v /usr/share/nginx/html/ --name nginx3 nginx
```  

创建一个名为 nginx3 的容器，并且挂载一个数据卷。

### 恢复

数据恢复需要一个临时容器，如下：
```
docker run --volumes-from nginx3 -v $(pwd):/backup nginx tar xvf /backup/backup.tar
```  

命令解释：  

1. 首先还是使用 `--volumes-from` 参数连接上备份容器，即第一步创建出来的 `nginx3` 。
2. 然后将当前目录映射到容器的 `/backup` 目录下。
3. 然后执行解压操作，将 `backup.tar` 文件解压。解压文件位置描述是一个容器内的地址，但是该地址已经映射到宿主机中的当前目录了，因此这里要解压缩的文件实际上就是宿主机当前目录下的文件。

# 容器连接

一般来说，容器启动后，我们都是通过端口映射来使用容器提供的服务，实际上，端口映射只是使用容器服务的一种方式，除了这种方式外，还可以使用容器连接的方式来使用容器服务。

例如，有两个容器，一个容器运行一个 `SpringBoot` 项目，另一个容器运行着 `mysql` 数据库，可以通过容器连接使 `SpringBoot` 直接访问到 `Mysql` 数据库，而不必通过端口映射来访问 `mysql` 服务。

为了案例简单，我这里举另外一个例子：  

> 有两个容器，一个 `nginx` 容器，另一个 `ubuntu` ，我启动 `nginx` 容器，但是并不分配端口映射，然后再启动 `ubuntu` ，通过容器连接，在 `ubuntu` 中访问 `nginx` 。  


具体操作步骤如下：  

- 首先启动一个 nginx 容器，但是不分配端口，命令如下：  

```
docker run -d --name nginx1 nginx
```  

命令执行结果如下：  

![20-1](/assets/images/2019/java/image_javaboy/0530/20-1.png)  

容器启动成功后，在宿主机中是无法访问的。  

- 启动ubuntu

接下来，启动一个 ubuntu ，并且和 nginx 建立连接，如下：  

```
docker run -dit --name ubuntu --link nginx1:mylink ubuntu bash
```  

这里使用 --link 建立连接，nginx1 是要建立连接的容器，后面的 mylink 则是连接的别名。  

运行成功后，进入到 ubuntu 命令行：  

```
docker exec -it ubuntu bash
```  

然后，有两种方式查看 nginx 的信息：  

**第一种**   

在 ubuntu 控制台直接输入 env ，查看环境变量信息：  

![20-2](/assets/images/2019/java/image_javaboy/0530/20-2.png)  

可以看到 docker 为 nginx 创建了一系列环境变量。每个前缀变量是 MYLINK ，这就是刚刚给连接取得别名。开发者可以使用这些环境变量来配置应用程序连接到 nginx 。该连接是安全、私有的。 访问结果如下：  

![20-3](/assets/images/2019/java/image_javaboy/0530/20-3.png)  

**第二种**   

另一种方式则是查看 ubuntu 的 hosts 文件，如下：  

![20-4](/assets/images/2019/java/image_javaboy/0530/20-4.png)  

可以看到，在 ubuntu 的 hosts 文件中已经给 nginx1 取了几个别名，可以直接使用这些别名来访问 nginx1 。

小贴士：  

> 默认情况下，ubuntu 容器中没有安装 curl 命令，需要手动安装下，安装命令如下：  
> apt-get update  
> apt-get install curl

# 总结

本文主要向大家介绍了 Docker 容器中的数据备份与恢复操作，同时也向大家介绍了容器连接相关的操作，不知道你学会了吗？

参考资料：

[1] 曾金龙，肖新华，刘清.Docker开发实践[M].北京：人民邮电出版社，2015.