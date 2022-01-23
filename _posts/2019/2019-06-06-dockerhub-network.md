---
layout: post
title:  DockerHub 与容器网络[Docker 系列-5]
tagline: by 江南一点雨
categories: Docker系列
tags: 
    - 江南一点雨
---

# DockerHub

DockerHub 类似于 GitHub 提供的代码托管服务，Docker Hub 提供了镜像托管服务，Docker Hub 地址为 [https://hub.docker.com/](https://hub.docker.com/)。

<!--more-->

利用 Docker Hub 读者可以搜索、创建、分享和管理镜像。Docker Hub 上的镜像分为两大类，一类是官方镜像，例如我们之前用到的 nginx、mysql 等，还有一类是普通的用户镜像，普通用户镜像由用户自己上传。对于国内用户，如果觉得 Docker Hub 访问速度过慢，可以使用国内一些公司提供的镜像，例如网易：

- [https://c.163yun.com/hub#/m/home/](https://c.163yun.com/hub#/m/home/) 

本文使用官方的 Docker Hub 来演示，读者有兴趣可以尝试网易的镜像站。

首先读者打开 Docker Hub ，注册一个账号，这个比较简单，我就不赘述了。

账号注册成功之后，在客户端命令行可以登录我们刚刚注册的账号，如下：  

![11-1](/assets/images/2019/java/image_javaboy/0530/11-1.png)

看到 Login Succeeded 表示登录成功！  

登录成功之后，接下来就可以使用 push 命令上传我们自制的镜像了。注意，自制的镜像要能够上传，命名必须满足规范，即 `namespace/name` 格式，其中 namespace 必须是用户名，以前文我们创建的 Dockerfile 为例，这里重新构建一个本地镜像并上传到 Docker Hub ，如下：  

![11-2](/assets/images/2019/java/image_javaboy/0530/11-2.png)  

首先调用 docker build 命令重新构建一个本地镜像，构建成功后，通过 docker images 命令可以看到本地已经有一个名为 wongsung/nginx 的镜像，接下来通过 docker push 命令将该镜像上传至服务端。上传成功后，用户登录Docker Hub ，就可以看到刚刚的镜像已经上传成功了，如下：  

![11-3](/assets/images/2019/java/image_javaboy/0530/11-3.png)  

看到这个表示镜像已经上传成功了，接下来，别人就可以通过如下命令下载我刚刚上传的镜像：  

```
docker pull wongsung/nginx
```  

pull下来之后，就可以直接根据该镜像创建容器了。具体的创建过程读者可以参考我们本系列前面的文章。

# 自动化构建

自动化构建，就是使用 Docker Hub 连接一个包含 Dockerfile 文件的 GitHub 仓库或者 BitBucket 仓库， Docker Hub 则会自动构建镜像，通过这种方式构建出来的镜像会被标记为 Automated Build ，也称之为受信构建 （Trusted Build） ，这种构建方式构建出来的镜像，其他人在使用时可以自由的查看 Dockerfile 内容，知道该镜像是怎么来的，同时，由于构建过程是自动的，所以能够确保仓库中的镜像都是最新的。具体构建步骤如下：  

- 添加仓库

首先登录到 Docker Hub，点击右上角的 Create，然后选择 Create Automated Build ，如下图：  

![12-1](/assets/images/2019/java/image_javaboy/0530/12-1.png)  

则新进入到的页面，选择 Link Account 按钮，然后，选择连接 GitHub ，在连接方式选择页面，我们选择第一种连接方式，如下：

![12-2](/assets/images/2019/java/image_javaboy/0530/12-2.png)  

选择完成后，按照引导登录 GitHub ，完成授权操作，授权完成后的页面如下：  

![12-3](/assets/images/2019/java/image_javaboy/0530/12-3.png)

- 构建镜像

授权完成后，再次点击右上角的 Create 按钮，选择 Create Automated Build ，在打开的页面中选择 GitHub ，如下两张图：  

![12-4](/assets/images/2019/java/image_javaboy/0530/12-4.png)  
![12-5](/assets/images/2019/java/image_javaboy/0530/12-5.png) 

这里展示了刚刚关联的 GitHub 上的仓库，只有一个 docker ，然后点击进去，如下：  

![12-6](/assets/images/2019/java/image_javaboy/0530/12-6.png)  

填入镜像的名字以及描述，然后点击 Create 按钮，创建结果如下：  

![12-7](/assets/images/2019/java/image_javaboy/0530/12-7.png)  

如此之后，我们的镜像就算构建成功了，一旦 GitHub 仓库中的 Dockerfile 文件有更新， Docker Hub 上的镜像构建就会被自动触发，不用人工干预，从而保证镜像始终都是最新的。


接下来，用户可以通过如下命令获取镜像：  

```
docker pull wongsung/nginx2
```  

获取到镜像之后，再运行即可。

有没有觉得很神奇！镜像更新只要更新自己的 GitHub 即可。镜像就会自动更新！事实上，我们使用的大部分 镜像都是这样生成的。

# 构建自己的 DockerHub

前面我们使用的 Docker Hub 是由 Docker 官方提供的，我们也可以搭建自己的 Docker Hub ，搭建方式也很容器，因为 Docker 官方已经将 Docker 注册服务器做成镜像了，我们直接 pull 下来运行即可，没有没很酷！。具体步骤如下：  

- 拉取镜像

运行如下命令拉取registry官方镜像：  

```
docker pull registry
```  

- 运行

接下来运行如下命令将registry运行起来，如下：  

```
docker run -itd --name registry -p 5000:5000 2e2f252f3c88
```  

运行成功后，我们就可以将自己的镜像提交到registry上了，如下：  


![13-1](/assets/images/2019/java/image_javaboy/0530/13-1.png)  

这里需要注意的是，本地镜像的命名按照 `registryHost:registryPort/imageName:tag ` 的格式命名。  

容器运行在宿主机上，如果外网能够访问容器，才能够使用它提供的服务。本文就来了解下容器中的网络知识。  

#  暴露网络端口

在前面的文章中，我们已经有用过暴露网络端口相关的命令了，即 `-p` 参数，实际上，Docker 中涉及暴露网络端口的参数有两个，分别是 `-p` 和 `-P` 。下面分别来介绍。  

- -P

使用 `-P`，Docker 会在宿主机上随机为应用分配一个未被使用的端口，并将其映射到容器开放的端口，以 Nginx 为例，如下：  

![14-1](/assets/images/2019/java/image_javaboy/0530/14-1.png)  

可以看到，Docker 为应用分配了一个随机端口 32768 ，使用该端口即可访问容器中的 nginx（http://lcalhost:32768）。 

- -p

`-p` 参数则有几种不同的用法：  

- hostPort:containerPort

这种用法是将宿主机端口和容器端口绑定起来，如下用法：  

![14-2](/assets/images/2019/java/image_javaboy/0530/14-2.png)  

如上命令表示将宿主机的80端口映射到容器的80上，第一个 80 是宿主机的 80 ，第二个 80 是容器的 80 。

- ip:hostPort:containerPort

这种是将指定的 ip 地址的端口和容器的端口进行映射。如下：  

![14-3](/assets/images/2019/java/image_javaboy/0530/14-3.png)  

将 192.168.0.195 地址的80端口映射到容器的80端口上。  

- ip::containerPort

这种是将指定 ip 地址的随机端口映射到容器的开放端口上，如下：  

![14-4](/assets/images/2019/java/image_javaboy/0530/14-4.png)  

# 总结

本文主要向大家介绍了 DockerHub 和容器网络，DockerHub 是我们容器的集散中心，网络则使我们的容器有办法对外提供服务。

参考资料：

[1] 曾金龙，肖新华，刘清.Docker开发实践[M].北京：人民邮电出版社，2015.