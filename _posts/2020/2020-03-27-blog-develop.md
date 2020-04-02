---
layout: post
categories: 学习
title: 从零搭建一个属于自己的个人网站
tags: 
  - 炸鸡可乐
---

很多小伙伴私信我，问我怎么弄一个个人博客系统，之前其实也聊过，不过没关系，今天我们再来详细的说一说。

<!--more-->

对于已经上线的项目，我们知道后期的迭代主要集中在线上发布这个环节，那么对于一个从零开发完的项目，到上线要经过哪些流程呢？

在这里，我把它分为如下三个步骤：

* **申购域名**
* **域名解析**
* **项目部署**

### 申购域名
在互联网中，域名又称网域，是由一串用点分隔的字符组成的互联网上某一台计算机或计算机组的名称，用于在数据传输时标识计算机的电子方位。

关于域名的来源，最早可以追溯到`ARPANET`时代。当时，网络上的每台计算机都使用`IP`数字地址的简单做法在网站中寻找另一台计算机，即通过主机文件（即我们俗称的`Hosts`）进行解析，`Hosts`文件内包含对应计算机的`IP`地址。

随着计算机数量的快速增长，使得主机文件被频繁更新。1983年，保罗·莫卡派乔斯发明了域名解析服务和域名系统，随后它们被引入`ARPANET`（阿帕网：美国高级研究计划署的简称，它是全球互联网的始祖）中。

关于域名，可以理解为是一个 IP 地址的代称，更具体一点可以理解为家庭的门牌号，例如腾讯（`www.qq.com`）、百度（`www.baidu.com`）、淘宝（`www.taobao.com`）、京东（`www.jd.com`）等等，在互联网上直接输入域名即可实现线上浏览访问，目的是为了便于记忆！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/8fe43529aec84f10b9b3b2e0aaf4b4fc.png)

**那么，如何申请一个属于自己的域名呢？**

以前主要是通过万维网来进行购买，现在因为市场已经放开了，阿里云、腾讯云、华为云、百度云等云服务器网站都可以购买！

比如小编我的域名，选择的是在阿里云上购买，在域名注册栏目下，输入自己想购买的域名，例如：`wangwang`。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/d7025e5a411d4b7e8d9d99e6ee16c4b5.jpg)

很遗憾，好的域名基本都被注册完了～

可不要小看这个域名注册，早期很多熟悉域名这块市场的人，早早的把那些热门的域名通过低价给注册了，等到那些有需求的人想注册购买的时候，通过高价拍卖的方式赚取利润。

例如，我们熟悉的`qq.com`，早在1995年被一个叫罗伯特·亨茨曼软件工程师给注册了，后来出价**200万美金**在域名交易市场上出售，可惜很长一段时间都无人问津。

也许是无人问津的缘故，罗伯特·亨茨曼似乎降低了对这个域名所能带来金钱的心理预期。

2003年，处于域名纠纷的腾讯注意到这个域名之后，与罗伯特·亨茨曼进行多次沟通，最终定价10万美元，加律师费1万，总计11万美元，买下`qq.com`这个域名。

11万美元，在2003年，对于中国人来说还真不是一个小数目！

如果你想买一个域名，晚注册不如早注册，当然注册也有一些小技巧，比如我们常用的`货比三家`，这个时候就派上用场了，如果你是一个新手用户，可以先在阿里云、腾讯云、华为云、百度云等网站上**查看一下是否有优惠券**，**然后对比购买价格**，还有就是**做活动的时候购买最划算**，**付款的时候可以省下不少哦**～

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/857cecd285964cd9816304b2088acfc4.jpg)

### 域名解析
域名注册完成之后，就需要进行解析了，在解析之前，我们需要一台服务器，如何购买服务器呢？

有两种方式，第一种方式就是在各大云厂商网站上购买，还是一样，用上我们的`货比三家`套路，**进行价格、服务器配置对比，找出性价比最高的一款**！

![阿里云服务器](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/0eff0058c8994dcdaed74257d7bff956.jpg)

![腾讯云服务器](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/4b5c71608315425780fc17e1370792ec.jpg)

配置不同，价格也不一样，根据自己的需要购买，**对于新手，推荐不必买太贵的，可以购买一款一年100元以下的服务器进行上手！**

这种方式购买的服务器有一个好处，就是可以进行线上维护，而且服务器提供独立公网IP，当服务器性能不够的适合，可以在线升级配置，服务器出问题了，还可以直接联系客服提供支持或者**申请退货**！

第二种方式就是搭建自己的服务器机房，这个方式适合中、大型企业，服务器购买基本是企业批量进行采购！

![服务器](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/6c2d6db768ec456597706c5a2bc31f9b.jpg)

采购完成之后，还需要**购买公网独立ip**，据说一个电信版的公网独立ip，一年费用就高达好几万，当然，机房还需要安装空调等散热设备，以及一些运维人员，进行安装调试，一年的费用开销比较大，显然不适合小企业！

购买完服务器之后，就可以进行域名解析了！怎么操作呢？

例如小编我的域名是在阿里云上购买的，可以去我的控制台中的域名菜单下，点击解析即可进行操作！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/61406865ef374a0bbd93ebf9e1064aad.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/91bd113b89c042c89ab52540c77bb046.jpg)

选择记录类型为`A`，主机记录可以为`www`或者`@`，记录值就是你购买的服务器的公网独立`IP`，点击确认即可完成操作，域名解析这个步骤就完成！其他的云服务器网站操作也类似！

**需要注意的是**：如果你购买的是海外的服务器，例如服务器地点在中国香港、新加坡、美国等，都属于海外版的服务器，这类服务器是不需要进行备案的；如果你购买的服务器地点在国内，是需要进行备案的！

如果不备案，通过域名是无法正常访问服务器IP的！如何进行备案呢？

**到自己购买的云服务器网站进行备案**，例如阿里云、腾讯云，其他的云厂商我没有试用过，只需要上传一些信息，例如域名、服务器IP、相关证件，全程线上操作，通过审核之后，15天之内基本就可以拿到备案号！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/a8dbf959350b44808f83e76fd54be7b0.jpg)

**如果你的业务是在国内，例如需要进行微信对接，那么推荐进行网站备案**；如果你的网站没啥业务或者在国外，可以购买国外的服务器，无需备案，但是国外的服务器`IP`经常会被国内的电信给封掉，有些时候可能会导致国内无法正常访问网站，这一点需要注意一下！
### 项目部署
域名注册并解析完成之后，就可以开始部署我们的项目了，例如我们熟悉的 JavaWeb 项目，因为小编我购买的是`CentOS`，部署起来也很简单！

首先使用客户端登录服务器，例如：windows 操作系统可以使用 shell，mac 操作系统可以使用 item2。
#### 安装JDK
登录之后，输入如下命令安装`JDK`！
```java
yum -y install java-1.8.0-openjdk
```
查看`JDK`安装情况
```java
java -version
```
![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/a0b4f1a42e1b47a3b4f3f3228f871239.jpg)
#### 安装Tomcat
`JDK`安装完成之后，接着再安装`tomcat`，直接访问`tomcat`官网（`http://tomcat.apache.org/`），下载对应的安装包，本次案例选择的是`apache-tomcat-8.5.45.tar.gz`版本，适用于`Linux`操作系统。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/d9a2a5c89f8645eba391ccd3a14ee5bb.jpg)

将下载的文件上传到对应的服务器文件夹中，之后解压文件夹
```
tar -zxvf apache-tomcat-8.5.40.tar.gz
```
或者，通过如下命令，直接在服务器上直接下载文件。
```java
wget https://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.53/bin/apache-tomcat-8.5.53.tar.gz
```
如果出现`wget`找不到，输入`yum install wget`命令进行安装即可！

解压完成之后，进入`apache-tomcat-8.5.53`根目录，修改`conf/server.xml`文件，修改端口号！
```xml
<!--将HTTP服务端口修改为80-->
<Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```
将 HTTP 服务端口修改为`80`之后，`cd`进入`bin`目录下，输入如下命令启动服务器：
```java
sh startup.sh
```
#### 线上访问
`tomcat`启动之后，通过域名即可实现访问服务器资源！

![通过域名访问的页面](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/f29ce871b0884aa887db150a25211c0a.jpg)

出现这个页面，表示已经部署成功了，这个时候，把自己的项目`war`包上传到`tomcat`目录下的`webapp`文件夹中，系统就发布成功了！

如果出现外部无法访问，查看防火墙是否启动，如果启动，将端口开放；如果使用的是云服务器，到控制台中的安全组，放行端口即可！
#### 博客模版
关于博客系统，其实网上有很多开源模版，例如 Jekyll，Jekyll 是一个简单的博客形态的静态站点生产机器，访问地址：`http://jekyllthemes.org/`，从中选择一个自己喜欢的模版，然后进行下载下来！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/e33b94987a564cb28160637c091b733e.jpg)

下载完成之后，还需要在服务器安装 Jekyll 运行环境，静态网站才能运行起来，关于安装就不过多介绍了，网上有很多教程，启动 Jekyll 服务之后，可以通过`http://localhost:4000`访问博客静态页面了，接着安装`nginx`，通过代理连接到 Jekyll 服务上，即可实现在浏览器上用域名访问博客系统。
#### 零费用搭建博客系统
**当然，你还可以不用花一分钱，来搭建一个博客系统**，直接在 github 上创建一个`你的用户名.github.io`这样格式的仓库名称，例如：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/3fd843a94aae4cbab3f17cc60bd05a18.jpg)

然后将上面下载的模版，提交到这个仓库中，同时修改`config.yml`文件，根据自己的需要将模版中的信息换成自己的信息即可！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/e757e54c76de4037a3fba7df4a34199c.jpg)

最后，直接访问`http://你的用户名.github.io`，结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-blog-develop/cd673792d05344948da4ac5508b66cc5.jpg)

需要注意的是，文章采用`markdown`编写，不过语法也比较简单，模版上各种样例都有！
### 总结
本篇主要介绍新系统上线的过程，不知道小伙伴们有没有 GET 到呢？
