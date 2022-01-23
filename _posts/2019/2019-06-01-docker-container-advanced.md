---
layout: post
title:  Docker 容器高级操作[Docker 系列-3]
tagline: by 江南一点雨
categories: Docker系列
tags: 
    - 江南一点雨
---

上篇文章向读者介绍了一个 Nginx 的例子，对于 Nginx 这样一个容器而言，当它启动成功后，我们不可避免的需要对 Nginx 进行的配置进行修改，那么这个修改要如何完成呢？且看下文。

<!--more-->

# 依附容器

`docker attach`

依附容器这个主要是针对交互型容器而言的，该命令有一定的局限性，可以作为了解即可，真正工作中使用较少。  

要是用 `docker attach` 命令，首先要确保容器已经启动，然后使用该命令才能进入到容器中。具体操作步骤如下：  

- 创建一个容器，然后启动：  

![4-1](/assets/images/2019/java/image_javaboy/0530/4-1.png)  

- 不要关闭当前窗口，再打开一个新的终端，执行 `docker attach ubuntu` :  

![4-2](/assets/images/2019/java/image_javaboy/0530/4-2.png)  

此时就能进入到容器的命令行进行操作了。  

如果容器已经关闭或者容器是一个后台容器，则该命令就无用武之地了。

由上面的操作大家可以看到，这个命令的局限性很大，使用场景也不多，因此大家作为一个了解即可。

# 容器内执行命令

如果容器在后台启动，则可以使用 `docker exec` 在容器内执行命令。不同于 `docker attach` ，使用 `docker exec` 即使用户从终端退出，容器也不会停止运行，而使用 `docker attach` 时，如果用户从终端退出，则容器会停止运行。如下图：  

![4-3](/assets/images/2019/java/image_javaboy/0530/4-3.png)  

基于这样的特性， 我们以后在操作容器内部时，基本上都是通过 `docker exec` 命令来实现。

# 查看容器信息

容器创建成功后，用户可以通过 `docker inspect` 命令查看容器的详细信息，这些详细信息包括容器的 id 、容器名、环境变量、运行命令、主机配置、网络配置以及数据卷配置等信息。执行部分结果如下图：  

![5-1](/assets/images/2019/java/image_javaboy/0530/5-1.png)  

使用 format 参数可以只查看用户关心的数据，例如：  

- 查看容器运行状态

![5-2](/assets/images/2019/java/image_javaboy/0530/5-2.png)  

- 查看容器ip地址

![5-3](/assets/images/2019/java/image_javaboy/0530/5-3.png)  

- 查看容器名、容器id

![5-4](/assets/images/2019/java/image_javaboy/0530/5-4.png)  

- 查看容器主机信息

![5-5](/assets/images/2019/java/image_javaboy/0530/5-5.png)  

容器的详细信息，在我们后边配置容器共享目录、容器网络时候非常有用，这个我们到后面再来详细介绍。


# 查看容器进程

使用 `docker top` 命令可以查看容器中正在运行的进程，首先确保容器已经启动，然后执行 `docker top` 命令，如下：  

![5-6](/assets/images/2019/java/image_javaboy/0530/5-6.png)  

# 查看容器日志

交互型容器查看日志很方便，因为日志就直接在控制台打印出来了，但是对于后台型容器，如果要查看日志，则可以使用docker提供的 `docker logs` 命令来查看，如下：  

![5-7](/assets/images/2019/java/image_javaboy/0530/5-7.png)  

如下图，首先启动一个不停打日志的容器，然后利用 `docker logs` 命令查看日志，但是默认情况下只能查看到历史日志，无法查看实时日志，使用 `-f` 参数后，就可以查看实时日志了。  

使用 `--tail` 参数可以精确控制日志的输出行数， `-t` 参数则可以显示日志的输出时间。  

![5-8](/assets/images/2019/java/image_javaboy/0530/5-8.png)  

该命令在执行的过程中，首先输出最近的三行日志，同时由于添加了 `-f` 参数，因此，还会有其他日志持续输出。同时，因为添加了 `-t` 参数，时间随同日志一起打印出来了。  

docker 的一大优势就是可移植性，容器因此 docker 容器可以随意的进行导入导出操作。

# 容器导出

既然是容器，我们当然希望 Docker 也能够像 VMWare 那样方便的在不同系统之间拷贝，不过 Docker 并不像 VMWare
 导出容器那样方便（事实上，VMWare 中不存在容器导出操作，直接拷贝安装目录即可），在 Docker 中，使用 export 命令可以导出容器，具体操作如下：  

- 创建一个容器，进行基本的配置操作

本案例中我首先创建一个 nginx 容器，然后启动，启动成功后，将本地一个 index.html 文件上传到容器中，修改 nginx 首页的显示内容。具体操作步骤如下：  

```
docker run -itd --name nginx -p 80:80 nginx
vi ./blog/docker/index.html
docker cp ./blog/docker/index.html nginx:/usr/share/nginx/html/
```  

首先运行一个名为 nginx 的容器，然后在宿主机中编辑一个 index.html 文件，编辑完成后，将该文件上传到容器中。然后在浏览器中输入 http://localhost:80 可以看到如下结果：  

![6-1](/assets/images/2019/java/image_javaboy/0530/6-1.png)  

容器已经修改成功了。

接下来通过 export 命令将容器导出，如下：  

![6-2](/assets/images/2019/java/image_javaboy/0530/6-2.png)  

该命令将容器导入到 docker 目录下。导出成功之后，我们就可以随意传播这个导出文件了，可以发送给其他小伙伴去使用了，相对于 VMWare 中庞大的文件，这个导出文件非常小。一般可能只有几百兆，当然也看具体情况。

# 容器导入

其他小伙伴拿到这个文件，通过执行如下命令可以导入容器（如果自己重新导入，需要记得将 docker 中和 nginx 相关的容器和镜像删除）：  

![6-3](/assets/images/2019/java/image_javaboy/0530/6-3.png)  

容器导入成功后，就可以使用 `docker run` 命令运行了。运行成功之后，我们发现自己定制的 index.html 页面依然存在，说明这是我们自己的那个容器。

参考资料：

[1] 曾金龙，肖新华，刘清.Docker开发实践[M].北京：人民邮电出版社，2015.