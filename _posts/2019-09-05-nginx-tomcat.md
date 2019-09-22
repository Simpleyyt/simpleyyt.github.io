---
layout: post
title: 手把手教你，使用 Nginx 搭配 Tomcat 实现负载均衡!
tagline: by 炸鸡可乐
categories: Nginx
tags: 
  - nginx
---

说到 Nginx ，相信大家都不会陌生，最常用的莫过于：用它来与 Tomcat 搭配做负载均衡，起到灰度发布的作用，同时保证系统高可用！

<!--more-->
### 01、简介
> Nginx（发音同 engine x）是异步框架的网页服务器，也可以用作反向代理、负载平衡器和 HTTP 缓存。该软件由伊戈尔·赛索耶夫创建并于2004年首次公开发布。 2011年成立同名公司以提供支持。2019年3月11日，Nginx 公司被 F5 Networks 以6.7亿美元收购。

传统模型下，一个项目部署在一台`tomcat`上，这个时候，假如`tomcat`因为服务器资源不够，突然挂机了，那么整个项目就无法使用，给客户造成的损失可想而知！

![](http://www.justdojava.com/assets/images/2019/java/image-jay/542c0dfd9e9247769118ed63b5fb99ea.jpg)

Nginx 就可以避免单台服务如果挂机，依然能保证服务正常使用，当我们把项目 war 包部署到三台服务器上时，即使服务器`A`、服务器`B`都挂了，依然能够通过服务器`C`访问项目资源！

![](http://www.justdojava.com/assets/images/2019/java/image-jay/628d9c0447424795b3d38e91f86123db.jpg)

好了，啥也不说了，直接开始干！
### 02、Nginx 安装
#### 2.1、下载 Nginx 安装包
直接访问 Nginx 官网（`https://nginx.org`），下载对应的安装包，本次案例选择的是`nginx-1.6.3.tar.gz`版本，安装环境是`centos7`。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/3e5842347af94a1f8bd6561a96fe6bc2.jpg)

上传到对应服务器的文件夹或者直接在服务器端使用`wget`命令
```
#下载nginx-1.6.3.tar.gz
wget -c https://nginx.org/download/nginx-1.6.3.tar.gz
```
如果出现如下信息：
```
-bash: wget: command not found
```
提示`wget`命令找不到，使用如下命令，进行安装，之后再次执行上述下载命令
```
yum install wget
```
#### 2.2、安装 Nginx
在按照 Nginx 之前，需要安装相应运行库环境，操作如下

1）安装 gcc 环境
```
yum install gcc-c++
```
2） 安装 PCRE 依赖库
```
yum install -y pcre pcre-devel
```
3）安装 zlib 依赖库
```
yum install -y zlib zlib-devel
```
4） 安装 OpenSSL 安全套接字层密码库
```
yum install -y openssl openssl-devel
```
5）解压 Nginx

安装完以上环境库之后，接着进行解压操作
```
#解压文件夹
tar -zxvf nginx-1.6.3.tar.gz
```
6）执行配置命令

`cd`进入文件夹
```
cd nginx-1.6.3
```
执行配置命令
```
./configure
```
如下图，表示执行配置成功！
![](http://www.justdojava.com/assets/images/2019/java/image-jay/278c473bc07b415bb2646f7401f3a543.jpg)

当然，也可以执行自定义配置文件，例如：
```
./configure \
--prefix=/usr/local/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--pid-path=/usr/local/nginx/conf/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```
注意：临时文件目录指定为`/var/temp/nginx`，需要在`/var`下创建`temp`及`nginx`目录

7）执行编译安装命令
```
make install
```
8）查找安装路径
```
whereis nginx
```
结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/74a7bee90e38484e86ab276e2ef05615.jpg)

9）启动服务

进入 nginx 的目录
```
cd /usr/local/nginx/sbin/
```
![](http://www.justdojava.com/assets/images/2019/java/image-jay/27efe98a3d6a469ead064e8c82cf62d8.jpg)

执行如下命令
```
#启动
./nginx

#停止，此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程
./nginx -s stop

#停止，此方式停止步骤是待nginx进程处理任务完毕进行停止
./nginx -s quit

#重新加载配置文件，Nginx服务不会中断
./nginx -s reload
```
10）修改配置文件

比如，修改端口号，默认端口号为`80`，咱们这里改成`81`；

进入配置文件夹
```
cd /usr/local/nginx/conf
```
备份原始配置文件
```
cp nginx.conf nginx.conf.back
```
编辑`nginx.conf`配置文件
```
vim nginx.conf
```
![](http://www.justdojava.com/assets/images/2019/java/image-jay/2b6d16ad50ac4bb68ac82fa04e330573.jpg)

找到`server`中的`listen`，修改端口号为`81`

![](http://www.justdojava.com/assets/images/2019/java/image-jay/0f23800af2034f7483d8c2d14c5c0a9d.jpg)

启动服务
```
./nginx
```
查看 nginx 进程
```
ps -ef|grep nginx
```
![](http://www.justdojava.com/assets/images/2019/java/image-jay/f1ecd9e7fa8e4e378f5faee5af2f33e9.jpg)

到此，nginx 安装基本完成，直接在浏览器上访问服务器地址`ip:81`，就可以进入页面

![](http://www.justdojava.com/assets/images/2019/java/image-jay/049e5789d5274683b2f4f8929fbf1c51.jpg)
### 03、Tomcat安装
直接访问 tomcat 官网（`http://tomcat.apache.org/`），下载对应的安装包，本次案例选择的是`apache-tomcat-8.5.45.tar.gz`版本，本次安装环境是`centos7`。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/f160521a7de0422b852efef24a367156.jpg)

上传到对应的服务器文件夹中，之后解压文件夹
```
tar -zxvf apache-tomcat-8.5.40.tar.gz
```
重新命名
```
mv apache-tomcat-8.5.40 tomcat-1
```
同样的，再次解压安装包，命名为`tomcat-2`
```
mv apache-tomcat-8.5.40 tomcat-2
```
1）修改 tomcat 端口号

**将 tomcat-1 的 http 端口设置为8080，将 tomcat-2 的 http 端口设置为8081。**

进入`tomcat`的`conf`文件夹，修改`server.xml`
```
vim server.xml
```
修改`SHUTDOWN`、`HTTP/1.1`、`redirectPort`、`AJP/1.3`端口，使其错开，避免重启的时候，报端口被占用问题

tomcat-1 的`SHUTDOWN`、`HTTP/1.1`、`redirectPort`、`AJP/1.3`设置如下：
```
<!--关闭服务端口-->
<Server port="9005" shutdown="SHUTDOWN">
...
<!--HTTP服务端口8080，跳转端口9443-->
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="9443" />
			   
<!--AJP服务端口-->
<Connector port="9009" protocol="AJP/1.3" redirectPort="9443" />
...
</Server>
```
tomcat-2 的`SHUTDOWN`、`HTTP/1.1`、`redirectPort`、`AJP/1.3`设置如下：
```
<!--关闭服务端口-->
<Server port="10005" shutdown="SHUTDOWN">
...
<!--HTTP服务端口8081-->
<Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="10443" />
			   
<!--AJP服务端口-->
<Connector port="10009" protocol="AJP/1.3" redirectPort="10443" />
...
</Server>
```
2）启动服务

分别进入 tomcat-1 、 tomcat-2 的`bin`文件夹，执行脚本，启动服务
```
sh startup.sh
```
查看服务是否启动成功
```
ps -ef|grep tomcat
```
![](http://www.justdojava.com/assets/images/2019/java/image-jay/6320b254528b4abbb5e319961aafbaa2.jpg)

说明已经启动成功了，可以直接在浏览器上分别输入`ip:8080`、`ip:8081`进行访问了，结果如下：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/90806dacf45c4eca9953262afedd049f.jpg)

3）编写Html

为了便于测试，我们创建一个`html`格式的页面，文件命名为`index.html`，内容如下：
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
	</head>
	<body>
		Hello server!
	</body>
</html>
```
进入`tomcat`的`webapps`文件夹，删除`ROOT`文件夹里面的东西，创建`index.html`文件；

在 tomcat-1 中，`index.html` 内容如下：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/8436c998da7e4722b0b0202e11e8b642.jpg)

在 tomcat-2 中，`index.html` 内容如下：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/d23b429abd2a409f8b50270be8cce4ee.jpg)

4）测试

创建好了之后，分别在浏览器上访问`ip:8080`、`ip:8081`；

`ip:8080`，结果如下：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/7a02126b480a4343a72a493322ae4b60.jpg)

`ip:8081`，结果如下：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/fb2a1e76271e478aa8423305fb08d8dc.jpg)
### 04、Nginx实现负载均衡
进入 Nginx 的配置文件夹
```
cd /usr/local/nginx/conf
```
编辑`nginx.conf`配置文件
```
vim nginx.conf
```
主要新增`upstream`集群配置点，配置如下：
```
worker_processes  1;

events {
    worker_connections  1024;
}  
  
http {  
    include       mime.types;
	
    default_type  application/octet-stream;
	
    sendfile        on;

    keepalive_timeout  65;

    gzip  on;

     #服务器的集群（这个就是我们要配置的地方）
	 #test.com:服务器集群名字,自定义
    upstream  test.com {
		#服务器配置   weight是权重的意思，权重越大，分配的概率越大。
		#127.0.0.1:8080、127.0.0.1:8081对应tomcat服务器地址
        server    127.0.0.1:8080  weight=1;
        server    127.0.0.1:8081  weight=2;
    }

    server {  
        listen       81;
        server_name  localhost;
  
    location / {
	·		#配置反向代理地址
            proxy_pass http://test.com;
            proxy_redirect default;
        }


        error_page   500 502 503 504  /50x.html;  
        location = /50x.html {  
            root   html;  
        }  
    }  
}  
```
参数说明：

* worker_processes：工作进程的个数，一般与计算机的cpu核数一致 
* worker_connections：单个进程最大连接数（最大连接数=连接数*进程数）
* include：文件扩展名与文件类型映射表
* default_type：默认文件类型
* sendfile ：开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
* keepalive_timeout：长连接超时时间，单位是秒
* upstream：服务器的集群配置点

配置好之后，进入`/usr/local/nginx/sbin/` 文件夹，重新刷新配置文件
```
./nginx -s reload
```
最后，访问`Nginx`服务器所在`ip:81`地址，多次刷新，看看效果：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/46a08f2039ed430cbe0039e9b6872a49.jpg)
![](http://www.justdojava.com/assets/images/2019/java/image-jay/4433ba8844b74dca9925c371948fe270.jpg)

至此，Nginx 与 Tomcat 搭配实现负载均衡已经配置完了，是不是很酷！

赶紧去试试吧！