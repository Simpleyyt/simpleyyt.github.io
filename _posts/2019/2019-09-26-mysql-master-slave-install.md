---
layout: post
title: 【MySQL】CentOS7 下安装及搭建主从复制
tagline: by 郑璐璐
categories: mysql
tags:
    - 郑璐璐
---

一篇文章，教你搞定 MySQL 主从复制。
<!--more-->
# CentOS7 下安装 MySQL


```
1，wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
2，rpm -ivh mysql-community-release-el7-5.noarch.rpm
3，yum install mysql-community-server
4，service mysqld restart（此为重启MySQL服务命令）
5，mysql -uroot -p（此为进入MySQL命令）
```

至此，就算安装完毕。刚安装上 MySQL 时，是没有密码的，所以运行第 5 个命令之后，直接回车，就能进入到 MySQL 界面，如图，即表示成功
![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-01.jpg)
# MySQL 修改密码
没有密码就能进入 MySQL ，安全性肯定是不能保证的，所以接下来介绍一下，如何修改密码。运行以下命令即可（这里以将密码改为 root 为例）：

```
use mysql;
update user set password=password("root") where user='root';
flush privileges;
exit;
```

检测密码是否成功，重新进入 MySQL ：

```
mysql -uroot -p
```

输入root之后，能看到如下界面，即为成功：
![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-02.jpg)
# MySQL搭建主从复制
写在前面：搭建主从复制的前提是，都安装好了 MySQL 。这篇文章以 192.168.243.133 为主，192.168.243.132 为从为例，来演示搭建过程。同时请注意，MySQL 密码为 root
1，133 为主，132 为从，从 133 上面，进入 MySQL 给 132 授权：

```
grant replication slave on *.* to 'root'@'192.168.243.132' identified by 'root';
```

```
参数说明:
用户名:root
密码:root
意为:允许192.168.243.132使用用户名为root,密码为root访问133
```

成功效果如图：
![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-03.jpg)
2，开启 133 的 binarylog
MySQL Binary Log 也就是常说的 bin-log  ,是 mysql 执行改动产生的二进制日志文件,其主要作用有两个:
* 数据恢复
* 主从数据库。用于 slave 端执行增删改，保持与 master 同步。

```
编辑my.cnf这个配置文件：vi /etc/my.cnf
将以下内容保存至该配置文件中：
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
server-id=1
log-bin=mysql-bin
expire_logs_days= 7
max_binlog_size= 100m                       //binlog每个日志文件大小
binlog_cache_size= 4m                        //binlog缓存大小
max_binlog_cache_size= 512m                     //最大binlog缓存大小
binlog-do-db=401_test
lower_case_table_names=1
```

具体如图：
![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-04.jpg)
进入 mysql ,查看 binary 是否开启成功：
![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-05.jpg)
3，在 133 和 132 上面分别创建数据库。此处以 401_test (如果数据库和我的不同,则主从的 my.cnf 上面相对应的数据库名称都要更改)为例

```
创建数据库：create database 401_test;
```

4，编辑 132 的 my.cnf 文件：

```
编辑my.cnf配置文件：vi /etc/my.cnf
将以下内容保存至该配置文件中：
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-bin=mysql-bin
binlog_format=mixed
server-id=2

replicate-do-db=401_test
```

与 133 稍微有些不同，请注意。效果如图：
![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-06.jpg)
5，查看 133 的 binary 日志位置，这在后续配置 132 时会用到。
连接 MySQL ，使用命令查看 binary ：

```
show master status\G
```

![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-07.jpg)

```
具体解析：
File:日志名称
Position:日志偏移量
Binlog_Do_DB:记录日志的库
```

6，开启 132 的同步：
在 132 上面运行以下命令:

```
CHANGE MASTER TO
    -> MASTER_HOST='192.168.243.133',
    -> MASTER_USER='root',
    -> MASTER_PASSWORD='root',
    -> MASTER_LOG_FILE='mysql-bin.000002',
    -> MASTER_LOG_POS=120;
```

具体如图:
![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-08.jpg)

```
HOST:主节点ip
USER: 133 授权给 132 的用户名
PASSWORD:授权给 132 的密码
MASTER_LOG_FILE: 133 的日志名称
MASTER_LOG_POS:日志偏移量,需要和 133 的一样
如果忘记了，请回看第 5 步
```

7，查看 132 的 slave 线程是否开启：

```
show slave status\G
```

![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-09.jpg)
Slave_IO_Running为读取master的binaryLog的线程
Slave_SQL_Running为执行SQL的线程
这两个线程必须都为YES才可以实现主从复制
至此主从复制就搭建完了。

# MySQL搭建互为主从
在以上配置的基础之上,将 132 作为 master , 133 作为 slave 进行再次配置。
1，在 132 上面连接 MySQL 之后，为 133 授权

```
grant replication slave on *.* to 'root'@'192.168.243.133' identified by 'root';
```

2，查看 132 的 binarylog

```
show master status;
```

![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-10.jpg)
3，开启 133 的同步(这里的步骤和 132 配置相同,我就不在这里展示了,如果忘记了,可以往上面再翻翻看)
4，查看 133 slave 的状态：

```
show slave status\G；
```

![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-11.jpg)
可能出现的错误：
![](http://www.justdojava.com/assets/images/2019/java/image-zll/mysqlInstall/mysql-12.jpg)
解决办法：
出现上图的错误就先将 slave 停掉,再操作一遍,使用命令: STOP SLAVE ,(此处命令必须为大写)
开启完同步之后需要打开 slave ,使用命令: START SLAVE (此处命令必须为大写)。
至此，搭建互为主从复制结束。

# 常用到的命令
在这个过程中有几个命令是常用到的，来总结一下(#后面为注释内容)：

```
进入 mysql ：mysql -uroot -p
重启 mysql ：systemctl restart mysql或service mysqld restart
查看 slave 线程：show slave status\G
mysql 授权命令:GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root';
#参数说明:
#第一个 root 是 mysql 的用户名
#第二个 root 是 mysql 的密码
# %表示所有机器都可以通过用户名 root ,密码 root 访问该 mysql
flush privileges ;  #使修改生效
```

感谢您的阅读。