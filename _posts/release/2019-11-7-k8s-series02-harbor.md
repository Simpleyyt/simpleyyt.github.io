---
layout: post
categories: k8s
title: CentOS7 下搭建 Harbor 仓库以及登录
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---

手把手教会你在 CentOS7 环境下搭建 Harbor 仓库，以及使用 Docker 以 HTTP 方式登录 Harbor 仓库。
<!--more-->
# CentOS7 环境下搭建 Harbor 仓库
## 环境依赖
Harbor 仓库需要环境:Python 2.7 或以上版本，Docker 1.10 或以上，Docker Compose 1.6.0 或以上。
CentOS7 自带 Python ，所以不需要安装。
关于 Docker 安装网上有很多成熟的教程，就不在赘述。
所以接下来说一说 docker-compose 。
部署 docker-compose (这里是以 1.16.1 版本为例，具体版本可以根据自己需要进行下载):
```
curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

提权:
```
chmod +x /usr/local/bin/docker-compose
```

验证docker-compose是否部署成功:
```
docker-compose --version
```

![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-001.jpg)
如上图，可以看到，我们已经成功部署 docker-compose 。

## 在线安装 Harbor 及其相关配置
为了方便寻找 Harbor ，将它安装在 usr/local/src 目录下，所以需要进入该目录:
```
cd /usr/local/src
```

下载相关gz包:
```
链接地址： https://github.com/vmware/harbor/releases
根据自己的需要，下载即可。本篇文章以下载 v1.3.0 为例，下载命令:
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.3.0.tgz
下载完成之后，进行解压:
tar -zxvf harbor-offline-installer-v1.3.0.tgz
```

耐心等待解压完成即可。解压完成之后，进行以下操作:
```
进入 harbor 目录: cd harbor
修改配置文件: vi harbor.cfg
配置文件中有 hostname : hostname = 192.168.243.138
#设置访问地址，可用 ip ，域名，不能使用 127.0.0.1 或 localhost ，在此设为 192.168.243.138
#如果设置为域名，记得在自己的 hosts 文件中做相应修改
#在此只是示例，具体可根据自己需要
 harbor.cfg 详细配置可参考:https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md#configuring-harbor
修改相关内容之后，运行: ./prepare  进行更新参数操作
```

但是需要注意，这个脚本有个坑 .hostname =reg.xx.com 默认的不能有，注释掉也不行。
不要问我为什么知道这个，耗在这里耗了将近半个小时。
配置文件中有关于 Harbor 的默认密码:
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-002.jpg)
修改配置文件之后，即可启动，一条命令即可:
```
./install.sh
```

如下图， Harbor 正在启动:
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-003.jpg)
如下图所示时，表示 Harbor 安装成功
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-004.jpg)
此时，我们可以通过访问刚才设置的 ip 地址，访问到 Harbor 界面
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-005.jpg)
输入默认账号: admin ，密码: Harbor12345 ，可以看到管理界面:
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-006.jpg)
在这个过程中，常用的命令就是停止和安装命令:
```
docker-compose down -v   停止
docker-compose up -d    启动
```

## 可能出现的错误
1，无法访问此页:造成的原因，可能没有把防火墙关闭，导致不能访问
一条命令即可:
```
临时关闭防火墙： systemctl stop firewalld 
永久关闭防火墙： systemctl disable firewalld.service
```

但是一般不建议把防火墙关掉。先写在这里，我后续再研究研究，看看都用到了哪儿些端口，等回来再更新

2，查看日志时，发现错误: failed to connect to tcp://postgresql:5432
解决办法:
```
停止并删除 docker 容器: docker-compose down -v
启动所有 docker 容器: docker-compose up -d
```

3，在停止并删除docker容器时，发现错误:ERROR: network harbor_harbor has active endpoints
解决办法:
```
重启 Docker service:service docker restart
```

4，ERROR:Failed to Setup IP tables: Unable to enable SKIP DNAT rule: (iptables failed: iptables --wait -t nat -I DOCKER -i br-2add1a39bc5d -j RETURN: iptables: No chain/target/match by that name。
(exit status 1)
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-007.jpg)
出现这个问题的原因是因为，我是后来才关闭的防火墙，这个时候需要重启一下 docker 才生效。
```
service docker restart 
```

重启docker之后，再运行命令:
```
./install.sh
```

问题解决。
以上是 CentOS7 下搭建 Harbor 仓库的过程，接下来说说使用 Docker 以 HTTP 方式登录 Harbor 仓库
# Docker 登录 Harbor 仓库( HTTP 方式)
Docker 登录到 Harbor 仓库时，不管是使用 http 协议还是使用 https 协议，都需要修改一些配置。
来介绍一下，在使用 http 协议时，需要进行什么哪些配置。
首先，确定自己的 Harbor 仓库使用的是 http 协议，在 harbor.cfg 文件中就可以看到:
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-008.jpg)
查找 docker 的服务文件，使用命令:
```
systemctl status docker
```

可以看到 docker 的服务文件在 /etc/systemd/system 目录下。
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-009.jpg)
接下来我们需要去编辑 docker.service 文件，并进行一些修改，在 ExecStart 处，添加 –insecure-registry 参数
```
--insecure-registry=reg.zll.com( Harbor地址， harbor.cfg 文件中的 hostname 项)
```

修改完成如下图:
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-010.jpg)
重新加载 service 文件，重启 docker 服务:
```
systemctl daemon-reload
systemctl restart docker
```

在图中可以看到， Harbor 仓库我是使用的域名，所以还需要在 hosts 文件中做一些配置，如果使用的是 ip 地址，则此步骤可以忽略
```
编辑 hosts 文件: vi /etc/hosts
将 Harbor 地址写入到 hosts 文件中: 192.168.243.138 reg.zll.com
#以我这次的配置为例，具体可以灵活变动
```

此时，相关步骤便结束了，我们可以在 Docker 客户端使用命令进行登录
```
docker login [ ip 地址或域名]( Harbor 地址， harbor.cfg 文件中的 hostname 项)
//根据提示分别输入用户名和密码
```

可以看到，此时 Docker 可以登录到 Harbor 仓库上面了。
![](http://www.justdojava.com/assets/images/2019/java/image-zll/k8sSeries/k8s-series02-011.jpg)
因为使用的是 http 协议登陆的，所以会有一个警告，对于实验环境来说，是可以忽略的。

## 可能遇到的问题:
Error response from daemon: Get http://reg.zll.com/v2/: dial tcp 192.168.243.138:80: connect: connection refused
原因是因为在修改了 hosts 文件之后，没有重新载入 docker ，再运行一下命令即可:
```
systemctl daemon-reload
systemctl restart docker
```

关于 Docker 登录 Harbor 仓库( HTTP 方式)到此便结束了
以上，感谢您的阅读~

