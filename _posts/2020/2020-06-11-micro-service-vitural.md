---
layout: post
categories: 虚拟机
title: 手把手教你，本地搭建虚拟机部署微服务
tags: 
  - 炸鸡可乐
---

关于虚拟机这块以前玩的也很多，但很少总结，容易遗忘，今天索性一条龙总结搞定！

<!--more-->

### 一、介绍
平时我们开发好的项目，通常都是在本地进行测试，然后把项目war包交给运维或者通过`jenkins`等构建工具发布到对应的服务器资源上。

对于生产环境，我们可能会使用云厂商的服务器资源，当然如果公司有自己的机房那就更好了。

但是**对于测试环境，尤其是小企业**，单独购买一台云服务器资源用来测试比较昂贵，我们一般都会将一台本地电脑使用虚拟软件分割出几个单独的资源环境，以达到节约资源、省钱的目的。

最近刚好在折腾虚拟机安装和配置的问题，老实说遇到不少坑，主要想法是将本地的几个微服务部署到虚拟机中去，然后进行测试，看服务是否都能正常跑通？

本次采用的是`VMware`软件，选择的是试用版！

安装这步就不介绍了哈，比较简单，大家可以自行百度！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/01.jpg)

虚拟机软件安装完成之后，为了跟真实的生产环境一致，本次选择的系统镜像是`Centos 7.8`，可以直接访问阿里云的镜像站点`http://mirrors.aliyun.com/centos/7.8.2003/isos/x86_64/`，下载速度会非常快，选择`CentOS-7-x86_64-Minimal-2003.iso`即可。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/02.jpg)

### 二、安装镜像

* 下载好了之后，打开`VMware`软件，点击创建新的虚拟机。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/03.jpg)

* 选择推荐的配置即可

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/04.jpg)

* 选择下载的系统镜像

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/05.jpg)

然后点击下一步，直到完成，等待虚拟机创建并安装成功！

安装过程都是傻瓜式的操作，我在本机上安装了三台，比较简单！

**重点的地方在于网络环境配置，下面我们一起来看看**。
### 三、网络介绍
虚拟机安装完成之后，需要进行相应的网络配置才能上网，`VMware`为我们提供了两种网络配置方案，**一种是：桥接模式，另一种是：NAT 模式**。

#### 3.1、桥接模式（推荐）
桥接模式，简单的说，就是在一个局域网内创立了一个单独的主机，**他可以访问这个局域网内的所有的主机**，但是需要手动配置子网掩码、网关、DNS等，并且他是和真实主机在同一个网段，这个模式里，虚拟机和宿主机可以互相ping通。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/06.png)

#### 3.3、NAT模式
NAT模式，简单的说，虚拟机通过主机的网络来访问外网，虚拟网络想访问外网，就必须通过宿主机的IP地址，主机和虚拟机对外的都是一个IP地址，**因此局域网内的其它机器无法连接到虚拟机**。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/07.png)

### 四、环境配置

>了解了网络配置介绍之后，可以很明显的得出，**我们需要的是整个局域网内的机器都可以访问虚拟机，因此虚拟机需要配置桥接模式进行上网**。

* 点击编辑，选择虚拟网络编辑器

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/08.jpg)

* 点击更改设置

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/09.jpg)

* 选中 **VMnet0**，选择**桥接模式**，并选择对应的**主机网卡**

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/10.jpg)

* **获取主机网卡信息非常关键**，如果不知道选哪一个，可以通过任务管理器查看

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/11.jpg)

* 虚拟网络编辑器配置完成之后，点击单个虚拟机进行网络设置

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/12.jpg)

* 选择桥接模式，连接网络

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/13.jpg)

* 在主机命令控制台上输入`ipconfig /all`获取主机的子网掩码、网关、DNS等信息，便于后续虚拟机进行配置

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/14.jpg)

* 最后登录终端虚拟机进行网络配置

```
#编辑虚拟机中对应网卡的信息（centos7）
vi /etc/sysconfig/network-scripts/ifcfg-ens33

#如果是centos6，编辑文件如下
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

* 在文件末尾添加如下信息，默认为动态获取IP

```
ONBOOT=yes     #开启自动启用网络连接
NETMASK=255.255.252.0   #设置子网掩码（主机中的子网掩码）
GATEWAY=197.168.24.1    #设置网关（主机中的网关）
DNS1=197.168.12.2           #设置主DNS（主机中的DNS服务器）
```

* 当然还可以配置静态IP地址，修改`BOOTPROTO`参数

```
BOOTPROTO=static  #启用静态IP地址，默认为dhcp，表示动态
```

* 设置静态IP地址，与主机IP处于同一网段

```
IPADDR=197.168.24.201    #设置静态IP地址
ONBOOT=yes       #开启自动启用网络连接
NETMASK=255.255.252.0   #设置子网掩码（主机中的子网掩码）
GATEWAY=197.168.24.1    #设置网关（主机中的网关）
DNS1=197.168.12.2           #设置主DNS（主机中的DNS服务器）
```

* 保存成功之后，重启网卡

```
systemctl restart network
```

* 最后测试一下是否可以上网，如果有返回信息，即可上网

```
ping www.baidu.com
```

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/15.jpg)


* 输入`ip addr`查看网络

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/16.jpg)

* 还可以通过`ifconfig`命令，如果出现找不到命令，可以通过如下命令进行安装

```
#安装net-tools
yum install net-tools
```

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/17.jpg)

### 五、项目部署
> 网络配置完成之后，就可以安装服务、部署项目了。

* 输入如下命令，安装 JDK

```
yum -y install java-1.8.0-openjdk
```

* 输入`java -version`查询是否安装成功

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/18.jpg)

* 使用`winScp`工具将`jar`或者`war`包上传到服务器目录

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/micro-service-vitural/19.jpg)

* 使用`xshell`等命令工具远程登录服务器，输入命令启动服务即可

```
#启动某jar服务，将日志打印到service.log文件中
nohup java -jar service.jar > service.log 2>&1 &
```

* 如果出现远程无法访问，查看防火墙是否开启，如果开启将其关闭

```
#查看防火墙是否开启
systemctl status firewalld.service

#关闭防火墙
systemctl stop firewalld.service

#禁止开机自动启动防火墙
systemctl disable firewalld.service
```
### 六、总结
整篇内容比较多，都是自己亲测的，尤其是网络配置部分坑特别多，在配置网络的时候，一定要查询主机是哪个网卡在上网，然后配置桥接模式的时候选择该网卡类型！

如果有表达不对的地方，望网友批评指出！
### 七、参考
1、[秋夜雨巷 - VMWare虚拟机网络配置](https://www.cnblogs.com/aeolian/p/8882790.html)