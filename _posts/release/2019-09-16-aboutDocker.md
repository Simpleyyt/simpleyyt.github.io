---
layout: post  
title: [Docker]关于 Docker 入门，这一篇就够了
tagline: by 郑璐璐
categories: Docker  
tags: 
    - 郑璐璐

---

关于 Docker 的一些概念和操作，我争取这一篇博客说完。
下面正文开始。
<!--more-->
# Docker 镜像与容器
说到 Docker ，你会常遇到两个内容： image 和 container (即镜像和容器)
关于镜像和容器，你可以这样来理解：镜像是构建 Docker 的基石，用户基于镜像来运行自己的容器。或者说，镜像是 Docker 生命周期中的构建或打包阶段，而容器则是启动或是执行阶段。
好吧，说的再明白点儿，就是有了镜像，才有的容器。容器是在镜像的基础上，才有的。
# Docker 安装
以前写过，这里就不赘述了，直接上链接：
需要科学上网的安装方法:[[Docker]CentOS7下Docker安装教程](https://blog.csdn.net/zll_0405/article/details/84890806)
不需要科学上网的安装方法:[[Docker]CentOS7通过rpm包安装Docker](https://blog.csdn.net/zll_0405/article/details/85092766)
# Docker 相关命令
## Docker 操作相关命令：
```
systemctl start docker		启动 docker
systemctl status docker		查看 docker 状态
systemctl stop docker		停止 docker
systemctl enable docker		开机自启
docker info 			查看 docker 概要信息
docker --help			查看 docker 帮助文档
```
## 镜像相关命令：
查看镜像命令： 
```
docker images
```
搜索镜像： 
```
docker search 镜像名称
```
拉取镜像:[[Docker]Docker拉取，上传镜像到Harbor仓库](https://blog.csdn.net/zll_0405/article/details/85124404)
删除镜像:[[Docker]如何批量删除镜像](https://blog.csdn.net/zll_0405/article/details/85217839)
## <font face='华文中宋' size=3 >容器相关命令：
<strong>查看容器:</strong>
查看正在运行的容器：
 ```
docker ps
```
查看所有容器：
```
docker ps -a
```
查看最后一次运行的容器： 
```
docker ps -l
```
查看停止的容器： 
```
docker ps -f status=exited
```
<strong>创建容器:</strong>
```
docker run
```
可以在 run 后面加参数。其中：
```
-i   表示运行容器
-t   表示容器启动后进入其命令行
--name  为创建的容器命名
-v     表示目录映射关系(前者是宿主机目录，后者是映射到宿主机上的目录)
-d     在 run 后面加上 -d 参数，则会创建一个守护式容器在后台运行
-p     表示端口映射，前者是宿主机端口，后者是容器内的映射端口
```
交互式方式创建容器 
```
docker run -it --name=容器名称 镜像名称：标签 /bin/bash
```
守护式方式创建容器 
```
docker run -di --name=容器名称 镜像名称：标签
```
登录守护式容器方式 
```
docker exec -it 容器名称(或容器 ID) /bin/bash
```
<strong>启动容器:</strong>
```
docker start 容器名称(或容器 ID)
```
<strong>停止容器:</strong>
```
docker stop 容器名称(或容器 ID)
```
## 文件拷贝：
将文件拷贝到容器内 
```
docker cp 需要拷贝的文件或目录  容器名称：容器目录
```
将文件从容器内拷贝出来 
```
docker cp 容器名称：容器目录	需要拷贝的文件或目录
```
## 目录挂载：
在创建容器时，将宿主机的目录与容器内的目录进行映射，这样可以通过修改宿主机某个目录的文件从而去影响容器
创建容器 添加 -v 参数 后边为 宿主机目录：容器目录，完整命令： 
```
docker run  -v 宿主机目录：容器目录
```
如果共享的是多级目录，可能会出现权限不足的情况
可以通过添加参数 --privileged=true 来解决，因为 CentOS7 中安全模块将 selinux 权限禁掉了，添加此参数，可以将问题解决。
## 查看容器 IP：
```
docker inspect 容器名称(容器 ID )
```
也可以直接输出 IP 地址：
```
docker inspect --format='{{NetworkSetting。IPAddress}}' 容器名称(容器 ID)
```
## 删除容器
```
docker rm 容器名称(容器 ID)
```
# 常见的应用部署
## MySQL 部署：
1 ，拉取镜像： 
```
docker pull centos/mysql-57-centos7
```
2 ，创建容器： 
```
docker run -di --name=mysql -p 3306：3306 -e MYSQL_ROOT_PASSWORD=root centos/mysql-57-centos7
```
其中： -p 代表端口映射，格式为 宿主机映射端口：容器运行端口
-e 代表添加环境变量 
MYSQL_ROOT_PASSWORD 是 root 用户的登录密码
3 ，进入 mysql 容器： 
```
docker exec -it mysql /bin/bash
```
4 ，登录 mysql ：
```
mysql -u root -p
```

## tomcat 部署：
1 ，拉取镜像 
```
docker pull tomcat：7-jre7
```
2 ，创建容器	
```
docker run -di --name=mytomcat -p 9000：8080 -v /usr/local/webapps:/usr/local/webapps tomcat：7-jre7
```

## Nginx 部署：
1 ，拉取镜像 
```
docker pull nginx
```
2 ，创建 nginx 容器 
```
docker run -di --name=mynginx -p 80：80 nginx
```

## Redis 部署：
1 ，拉取镜像 
```
docker pull redis
```
2 ，创建 redis 容器 
```
docker run -di --name=myredis -p 6379：6379 redis
```
# 迁移与备份
容器保存为镜像： 
```
docker commit 容器名称 镜像名称
```
例： 
```
docker commit mynginx mynginx_i
```
将镜像保存为 tar 文件，例：
```
docker save -o mynginx。tar mynginx_i
```
镜像恢复与迁移： -i 输入的文件，例： 
```
docker load -i mynginx。tar
```

# Dockerfile
Dockerfile 是由一系列命令和参数构成的脚本，基于基础镜像，最终创建一个新的镜像，常用命令有：
```
FROM image_name：tag  定义了使用哪儿个基础镜像启动构建流程
MAINTAINER user_name	声明镜像的创建者
ENV key value		设置环境变量(可以写多条)
RUN command 		是 Dockerfile 的核心部分(可以写多条)
ADD source_dir/file dest_dir/file  将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压
COPY source_dir/file dest_dir/file   和 ADD 相似，但是如果有压缩文件并不能解压
WORDIR path_dir 设置工作目录
```
需要注意一下，如果要使用 Dockerfile 文件，名字必须为「Dockerfile」,否则里面的命令不会有效。
# 镜像上传下载到镜像仓库
以前写过博客，感觉还是比较详细的:[[Docker]Docker拉取，上传镜像到Harbor仓库](https://blog.csdn.net/zll_0405/article/details/85124404)(在上面应该也看到过了，再放一次)

关于 Docker 入门，我只能帮你到这儿了~