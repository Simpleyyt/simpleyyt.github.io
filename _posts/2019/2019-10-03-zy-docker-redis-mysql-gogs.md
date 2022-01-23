---
layout: post
categories: Docker系列
title: Docker 安装 Redis、MySQL、Gogs
tagline: by 子悠
tags:
  - 子悠

---

**人生有涯，学海无涯**

最近接到一个项目，由于项目的独特性需要自己搭建一些环境，刚好之前学了一些 Docker（这里如果大家不熟悉 Docker 可以查看公号前面松哥写的 Docker 的系列文章），所以就决定采用 Docker 搭建，毕竟搭建方便、简单、快速。

<!--more-->

### 01、Docker 安装 Redis

1. 搜索镜像 `docker search redis`：使用该命令可以搜索出所有的 `redis` 镜像列表![image-20191003102625851](http://justdojava.com/assets/images/2019/java/image_ziyou/100301.png)

2.  如果没有特殊版本需求，可以使用：`docker pull redis` 命令直接安装最新版本 `Redis`
3. 下载过后使用 `docker images | grep redis` 命令查看已经获取到的镜像![image-20191003104525395](http://justdojava.com/assets/images/2019/java/image_ziyou/100302.png)

4. 启动 `Redis` 服务：`docker run --name redis -d -p 6379:6379 redis --requirepass "zxcvbnmdfghjk09876"`
   1. `docker run`：表示创建并运行一个容器；
   2. `--name redis`：表示创建一个名字为 `redis` 的容器；
   3. `-d`：表示后台运行；
   4. `-p 6379:6379`：表示将宿主机的 6379 端口映射到容器的 6379 端口上；
   5. `redis`：表示依赖的镜像名称；
   6. `--requirepass "zxcvbnmdfghjk09876"` ：表示设置密码
5. 查看运行的`Redis` 实例![image-20191003105336215](http://justdojava.com/assets/images/2019/java/image_ziyou/100303.png)

#### 注意事项

大家在公网服务器安装 Redis 的时候**一定要设置密码，一定要设置密码，一定要设置密码**。

如果不设置密码很容易被黑客利用 Redis 的漏洞进行比特币的勒索。如果不巧遇到了那都是血的教训！切记不要抱有侥幸心理，或者简单的以为换个端口就可以了，端口的数量是有限制了，黑客完全可以遍历一下就破解了。最好两个都设置，既改端口也加密码，双保险，当然密码也不要简单到随便一个字典库就能破解的那种，尽量复杂点。

### 02、Docker 安装 MySQL5.6

与  Redis 安装方式类似，不过这里获取的是指定版本的 MySQL 。

1. `docker pull mysql:5.6`；
2. 创建相关文件夹存放数据 `mkdir -p ~/mysql/data ~/mysql/logs ~/mysql/conf`；
3. 进到上面创建的 `mysql` 文件夹中执行  `docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=dswrtyui9876 -d mysql:5.6`；
4. `-v`：该参数表示关联宿主机的文件夹与容器中的文件夹，将配置文件目录，日志文件目录，数据文件目录保存到宿主机上，避免在容器挂掉或者重启后数据丢失，其他参数跟上面一致；
5. `-e MYSQL_ROOT_PASSWORD=dswrtyui9876`： 表示初始化 `root` 用户的密码；
6. 查看运行的 `MySQL` 实例![image-20191003111951804](http://justdojava.com/assets/images/2019/java/image_ziyou/100304.png)

### 03、Docker 安装 gogs

#### gogs 简介

首先提到代码管理平台，大家首先想到的肯定是 Github 以及 Gitlab，这两种大家平时应该用到的比较多，开源软件用的大部分是 Github，公司内部大部分使用的是 Gitlab。Gogs 也是一种代码管理平台，相比 Gitlab 来说相对轻量级。

我这里为什么要使用 Gogs 而不使用 Gitlab 呢？**主要是个人服务器配置跟不上啊！！！**

尝试了安装 Gitlab，安装后服务器完全跑不起来了，本来个人服务器性能就不是很好，上面还跑了几个程序，安装完 Gitlab 后连博客网站都打不开，果断放弃。官方推荐的安装 Gitlab 硬件配置是 4 核 8G，相对来说 Gogs 就轻量很多，安装后基本对服务器没什么影响，而且 Docker 安装十分方便。

#### 安装

1. 拉取镜像 `docker pull gogs/gogs`；
2. 创建 gogs 文件夹，`mkdir -p docker/gogs`
3. 进入文件夹，`cd docker/gogs/`
4. 运行 gogs `docker run -d --name=mygogs -p 10022:22 -p 10080:3000 -v /root/docker/gogs:/data gogs/gogs`

> /root/docker/gogs 这里用的是 root 用户创建的，所以可以这样使用，或者参考 MySQL 的方式，使用$PWD

	5. 启动成功后打开网页 `http://ip:10080`，首次打开的时候需要配置数据库以及管理员账号密码信息
 	6. 数据库配置，这里需要先在 MySQL 中创建一个对应的数据库如：gogs，以及可以配置一个对应的账号和密码。![image-20191003114654670](http://justdojava.com/assets/images/2019/java/image_ziyou/100305.png)

注意修改端口号：

![image-20191003115758410](http://justdojava.com/assets/images/2019/java/image_ziyou/100306.png)



#### 可能遇到的坑

如果在上一步点击安装后一切正常那边跳过这一步，如果出现 `MySQL error: The maximum column size is 767 bytes`，那么很高兴你遇到一个坑，不过别怕，我们可以解决它只需要对 mysql 进行参数的设置就好了。

解决方案：

1. 使用命令 `docker exec -it ced036c68e08 bash` 进入上面安装的 MySQL 容器中，并且登录 root 账号进入 MySQL 的交互模式中![image-20191003131242089](http://justdojava.com/assets/images/2019/java/image_ziyou/100307.png)

2. 然后输入下面两条命令，再次安装即可

   1. ```mysql
      set global innodb_file_format = BARRACUDA;
      set global innodb_large_prefix = ON;
      ```

3.  `docker exec -it ced036c68e08 bash`：通过使用 `docker exec -it ` 进入到相应容器中并且打开终端交互模式。
4. 安装完成后就可以登录创建仓库和组织了，开始玩耍吧。

### 04、总结

这篇文章简单给大家介绍了一下使用 Docker 安装常用的一些组件，作为普通的开发人员可能平时需要自己搭建组件环境的情况不多，但是我们还是需要有这个技能的，毕竟要向着全栈的方向走。在折腾的过程中，也顺便学到了一个 Linux 系统的技能，关于这个技能我会分享到《Java 极客技术》的知识星球，环境大家到我们《Java 极客技术》的知识星球中共同学习，共同成长。

![子悠-知识星球](http://justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)

