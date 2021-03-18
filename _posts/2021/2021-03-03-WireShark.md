---
layout:post
title: 手把手教你学会如何使用WireShark进行抓包
catgories: 抓包工具
tags:
    - 懿
---

前段时间，因为同事需要分析数据，所以使用了WireShark，但是呢，小伙子不太知道怎么抓取数据，于是就来询问了一下阿粉，阿粉就手把手的教给他，如何使用WireShark进行抓包分析，在这里也分享给大家。

<--more-->

### 1.什么是WireShark

WireShark实际上是有前身的，他的前身叫做Ethereal，它就是用于一个网络封包分析软件，也就是我们日常生活中俗称的抓包工具，而它最大的特点就是，能够尽可能多的显示出详细的网络封包资料，WireShark使用WinPCAP作为接口，直接与网卡进行数据报文交换。

### 2.WireShark下载安装

先把网址给大家放上‘https://www.wireshark.org/’，下载网址在这里，

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/03-03/1.png)

在这里给大家推荐，最好使用你们系统的对应版本，毕竟64位，就是64位，32位就是32位，不同的版本对应不同系统，不要想着兼容，你要知道，一个诡异问题的出现很可能导致你之前做的准备工作都浪费掉了。

大家如果感觉自己下载的慢的，可以在后台回复，抓包，WireShark，获取百度云的下载地址，

下载完成之后，安装，一路Next即可，但是注意，如果你想要切换安装的路径，那么路径最好都是英文的，哪怕你自己不嫌麻烦整个abc，那也比“抓包”两个字好，因为中文路径也是容易出现问题，而不同意被检测出来的。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/03-03/2.jpg)

等待安装完成，桌面出现logo即可，那么我们就点开来看看怎么使用的吧。

### 3.WireShark的使用

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/03-03/3.jpg)

大家从图中就能看出，阿粉用的是笔记本，因为在图中的折线图中出现了波动，剩下都是虚拟网卡里面的内容，就像刚才阿粉说的，WireShark使用的WinPCAP作为接口，直接与网卡进行数据报文的交换，而大家看折线图上的名称，就有阿粉用来玩虚拟机的几个网卡。

找个WLAN试试看？

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/03-03/4.png)

在WLAN的连接中，接口中的数据就会被WireShark成功捕获到，我们来分析一下这个里面都是有什么东西。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/03-03/5.jpg)

- Display Filter(显示过滤器)，  用于过滤
  
- Packet List Pane(封包列表)， 显示捕获到的封包， 有源地址和目标地址，端口号。 颜色不同，代表
  
- Packet Details Pane(封包详细信息), 显示封包中的字段
  
- Dissector Pane(16进制数据)
  
- Miscellanous(地址栏，杂项)

我们在抓包的时候，肯定不能从这一大堆的封包数据中去寻找内容，肯定要进行过滤，大家可以从过滤器上搜索类型。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/03-03/6.jpg)

- arp 显示所有 ARP 数据包

- dns 显示所有 DNS 数据包

- ftp 显示所有 FTP 数据包

- http 显示所有 HTTP 数据包

- ip 显示所有 IPv4 数据包

- ipv6 显示所有 IPv6 数据包

- tcp 显示所有基于 TCP 的数据包

我们按照HTTP来进行抓包，就像下面这个样子。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/03-03/7.jpg)

我们看一下封包详细信息里面都有哪些内容。

- Frame 物理层的数据帧

- Ethernet II 数据链路层以太网帧头部信息

- Internet Protocol Version 4  网际层 IP 包头部信息

- Transmission Control Protocol 传输层的数据段头部信息

- Hypertext Transfer Protocol  应用层的信息，此处是 HTTP 协议

欧呦，Src很有意思呀，shenzhen_3a,还有目标的Mac地址，不错呦。

### WireShark最需要学会的内容

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2021/03-03/8.jpg)

也就是我们过滤器的规则，因为如果你要是对这个规则设置不好，你抓包的时候就不能精准的找到位置，进行分析。

我们总不能输入个HTTP，然后从里面去找吧，要学会效率工作。

比如我们知道IP地址，那么我们就能：

 ip.src == 192.168.1.8
 
 我们知道端口号是 8080，那么我们就可以：
 
 tcp.port == 8080
 
再比如说我们处理HTTP里面的"Get"请求：

http.request.method=="GET" ，POST方法同样。

我们学习到这里，就已经算是能够进行WireShark的抓包行动了，大家有兴趣的可以用WireShark进行抓包分析，来分析之前阿粉所说的TCP的三次握手和四次挥手来进行校验，

地址送上：[三次握手和四次挥手到底是个什么鬼东西？](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247493536&idx=1&sn=f4763c3f5cb9672b689463791c681d7b&chksm=c2868861f5f1017700711b3e3a59147d39982c1a7ca397fbc222d9c0f325673a0957512728f8&token=757368668&lang=zh_CN#rd)


#### 文献参考
《WireShark官网DOC》