---
layout: post
categories: Docker系列
title: Docker 容器编排入门[Docker 系列-8]
tagline: by 江南一点雨
tags: 
  - 江南一点雨
---

在实际的开发环境或者生产环境，容器往往都不是独立运行的，经常需要多个容器一起运行，此时，如果继续使用 run 命令启动容器，就会非常不便，在这种情况下，docker-compose 是一个不错的选择，使用 docker-compose 可以实现简单的容器编排,本文就来看看 docker-compose 的使用。  

<!--more-->

本文以 jpress 这样一个开源网站的部署为例，向读者介绍 docker-compose 的使用。jpress 是 Java 版的 wordPress ，不过我们不必关注 jpress 的实现，在这里我们只需要将之当作一个普通的应用即可，完成该项目的部署工作。  

# 准备工作

这里我们一共需要两个容器：  

- Tomcat
- MySQL

然后需要 jpress 的 war 包，war 包地址：[jpress](https://github.com/JpressProjects/jpress/raw/alpha/wars/jpress-web-newest.war)  

当然，这里的 jpress 并不是必须的，读者也可以结合自身情况，选择其他的 Java 项目或者自己写一个简单的 Java 项目部署都行。  

# 编写 Dockerfile  

Tomcat 容器中，要下载相关的 war 等，因此我这里编写一个 Dockerfile 来做这个事。在一个空的文件夹下创建 Dockerfile ，内容如下：  

```
FROM tomcat
ADD https://github.com/JpressProjects/jpress/raw/alpha/wars/jpress-web-newest.war /usr/local/tomcat/webapps/
RUN cd /usr/local/tomcat/webapps/ \
    && mv jpress-web-newest.war jpress.war
```  

解释：  
1. 容器基于 Tomcat 创建。
2. 下载 jpress 项目的 war 包到 tomcat 的 webapps 目录下。  
3. 给 jpress 项目重命名。  

# 编写 docker-compose.yml

在相同的目录下编写 docker-compose.yml ，内容如下（关于 yml 的基础知识，这里不做介绍，读者可以自行查找了解）：  

```yml
version: "3.1"
services:
  web:
    build: .
    container_name: jpress
    ports:
      - "8080:8080"
    volumes:
      - /usr/local/tomcat/
    depends_on:
      - db
  db:
    image: mysql
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 123
      MYSQL_DATABASE: jpress
```  

解释：  

1. 首先声明了 web 容器，然后声明db容器。 
2. `build .` 表示 web 容器项目构建上下文为 `.` ，即，将在当前目录下查找 Dockerfile 构建 web 容器。  
3. container_name 表示容器的名字。  
4. ports 是指容器的端口映射。 
5. volumes 表示配置容器的数据卷。 
6. depends_on 表示该容器依赖于 db 容器，在启动时，db 容器将先启动，web 容器后启动，这只是启动时机的先后问题，并不是说 web 容器会等 db 容器完全启动了才会启动。
7. 对于 db 容器，则使用 image 来构建，没有使用 Dockerfile 。
8. restart 描述了容器的重启策略。 
9. environment 则是启动容器时的环境变量，这里配置了数据库 root 用户的密码以及在启动时创建一个名为 jpress 的库，environment 的配置可以使用字典和数组两种形式。

OK，经过如上步骤，docker-compose.yml 就算配置成功了

# 运行

运行的方式有好几种，但是建议使用 up 这个终极命令，up 命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。对于大部分应用都可以直接通过该命令来启动。默认情况下， `docker-compose up` 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试，通过 Ctrl-C 停止命令时，所有容器将会停止，而如果使用 `docker-compose up -d` 命令，则将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。  

因此，这里进入到 docker-compose.yml 所在目录下，执行如下命令：  

```
docker-compose up -d
```  
执行结果如下：  

![21-1](/assets/images/2019/java/image_javaboy/0530/21-1.png)  

执行后，通过 `docker-compose ps` 命令可以看到容器已经启动了。  

# 初始化配置

接下来，浏览器中输入 http://localhost:8080/jpress ，就可以看到 jpress 的配置页面，如下：  

![21-2](/assets/images/2019/java/image_javaboy/0530/21-2.png)  

根据引导页面配置数据库的连接信息以及网站的基本信息：  


![21-3](/assets/images/2019/java/image_javaboy/0530/21-3.png)  
![21-4](/assets/images/2019/java/image_javaboy/0530/21-4.png)  

**注意：由于 mysql 和 web 都运行在容器中，因此在配置数据库地址时，不能写回环地址，否则就去 web 所在的容器里找数据库了**。  

配置完成后，运行如下命令，重启 web 容器：

```
docker restart jpress
```  
# 测试

浏览器中分别查看博客首页以及后台管理页，如下图：  


![21-5](/assets/images/2019/java/image_javaboy/0530/21-5.png)  
![21-6](/assets/images/2019/java/image_javaboy/0530/21-6.png)  

# 其他

如果想要停止容器的运行，可以执行如下命令：  

```
docker-compose down
```  

# 总结

本文主要向大家介绍了简单的容器编排，专业的或者大型项目的容器编排需要结合 K8s 来做，我们后面有机会再向大家介绍。

参考资料：

[1] 曾金龙，肖新华，刘清.Docker开发实践[M].北京：人民邮电出版社，2015.