---
layout: post
title: 手把手教你，在CentOS上安装ELK，进行服务器日志收集
tagline: by 炸鸡可乐
categories: 运维
tags: 
  - elk
---

每当项目上线时，因为项目是集群部署的，所以，来回到不同的服务器上查看日志会变得很麻烦，你是不是也碰到这样类似的问题，那么ELK将能解决你遇到的问题！


<!--more-->

### 01、ELK Stack 简介
ELK 不是一款软件，而是 Elasticsearch、Logstash 和 Kibana 三种软件产品的首字母缩写。这三者都是开源软件，通常配合使用，而且又先后归于 Elastic.co 公司名下，所以被简称为 ELK Stack。根据 Google Trend 的信息显示，ELK Stack 已经成为目前最流行的集中式日志解决方案。

* Elasticsearch：分布式搜索和分析引擎，具有高可伸缩、高可靠和易管理等特点。基于 Apache Lucene 构建，能对大容量的数据进行接近实时的存储、搜索和分析操作。通常被用作某些应用的基础搜索引擎，使其具有复杂的搜索功能；
* Logstash：数据收集引擎。它支持动态的从各种数据源搜集数据，并对数据进行过滤、分析、丰富、统一格式等操作，然后存储到用户指定的位置；
* Kibana：数据分析和可视化平台。通常与 Elasticsearch 配合使用，对其中数据进行搜索、分析和以统计图表的方式展示；
* Filebeat：ELK 协议栈的新成员，一个轻量级开源日志文件数据搜集器，基于 Logstash-Forwarder 源代码开发，是对它的替代。在需要采集日志数据的 server 上安装 Filebeat，并指定日志目录或日志文件后，Filebeat 就能读取数据，迅速发送到 Logstash 进行解析，亦或直接发送到 Elasticsearch 进行集中式存储和分析。

### 02、ELK 常用架构及使用场景介绍
#### 2.1、最简单架构
在这种架构中，只有一个 Logstash、Elasticsearch 和 Kibana 实例。Logstash 通过输入插件从多种数据源（比如日志文件、标准输入 Stdin 等）获取数据，再经过滤插件加工数据，然后经 Elasticsearch 输出插件输出到 Elasticsearch，通过 Kibana 展示。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/5f5141ee1e094b26909171c13dbf28fe.png)

这种架构非常简单，使用场景也有限。初学者可以搭建这个架构，了解 ELK 如何工作。

#### 2.2、Logstash 作为日志搜集器
这种架构是对上面架构的扩展，把一个 Logstash 数据搜集节点扩展到多个，分布于多台机器，将解析好的数据发送到 Elasticsearch server 进行存储，最后在 Kibana 查询、生成日志报表等。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/3f376459ecb649a69737c7d9453bea25.png)

这种结构因为需要在各个服务器上部署 Logstash，而它比较消耗 CPU 和内存资源，所以比较适合计算资源丰富的服务器，否则容易造成服务器性能下降，甚至可能导致无法正常工作。
#### 2.3、Beats 作为日志搜集器
这种架构引入 Beats 作为日志搜集器。目前 Beats 包括四种：
* Packetbeat（搜集网络流量数据）；
* Topbeat（搜集系统、进程和文件系统级别的 CPU 和内存使用情况等数据）；
* Filebeat（搜集文件数据）；
* Winlogbeat（搜集 Windows 事件日志数据）。

Beats 将搜集到的数据发送到 Logstash，经 Logstash 解析、过滤后，将其发送到 Elasticsearch 存储，并由 Kibana 呈现给用户。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/b455d42fd11f4dfca6b338ce137fe609.png)

这种架构解决了 Logstash 在各服务器节点上占用系统资源高的问题。相比 Logstash，Beats 所占系统的 CPU 和内存几乎可以忽略不计。另外，Beats 和 Logstash 之间支持 SSL/TLS 加密传输，客户端和服务器双向认证，保证了通信安全。

因此这种架构适合对数据安全性要求较高，同时各服务器性能比较敏感的场景。

#### 2.4、引入消息队列机制的架构
Beats 还不支持输出到消息队列，所以在消息队列前后两端只能是 Logstash 实例。这种架构使用 Logstash 从各个数据源搜集数据，然后经消息队列输出插件输出到消息队列中。目前 Logstash 支持 Kafka、Redis、RabbitMQ 等常见消息队列。然后 Logstash 通过消息队列输入插件从队列中获取数据，分析过滤后经输出插件发送到 Elasticsearch，最后通过 Kibana 展示。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/25db8d5351cb49468c295b446659da9d.png)

这种架构适合于日志规模比较庞大的情况。但由于 Logstash 日志解析节点和 Elasticsearch 的负荷比较重，可将他们配置为集群模式，以分担负荷。引入消息队列，均衡了网络传输，从而降低了网络闭塞，尤其是丢失数据的可能性，但依然存在 Logstash 占用系统资源过多的问题。

说了这么多理论，对于喜欢就干的小编来说，下面我将以**Beats 作为日志搜集器**的架构，进行详细安装介绍！
### 03、基于 Filebeat 架构的配置安装
由于我这边是测试环境，所以ElasticSearch + Logstash + Kibana + nginx这四个软件我都是装在一台机器上面，如果是生产环境，建议分开部署，并且ElasticSearch可配置成集群方式。

软件架构示意图：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/c70165cceb714f809ec42e1aa85d19db.jpg)

安装环境及版本：
* 操作系统：Centos7
* 内存：大于或等于4G
* ElasticSearch：6.1.0
* Logstash：6.1.0
* Kibana：6.1.0
* filebeat ：6.2.4


**建议把所需的安装包，手动从网上下载下来，因为服务器下载ELK安装包速度像蜗牛......，非常非常慢～～，可能是国内的网络原因吧！**

将手动下载下来的安装包，上传到服务器某个文件夹下。
![](http://www.justdojava.com/assets/images/2019/java/image-jay/8a95276fd6804d82a2d735c1d9604867.jpg)

#### 3.1、ElasticSearch安装
##### 3.1.1、安装JDK（已经安装过，可以跳过）
elasticsearch依赖Java开发环境支持，先安装JDK。
```
yum -y install java-1.8.0-openjdk
```
查看java安装情况
```
java -version
```
![](http://www.justdojava.com/assets/images/2019/java/image-jay/392e500169dc4f5cb8bd77bd23087560.jpg)

##### 3.1.2、安装ElasticSearch
进入到对应上传的文件夹，安装ElasticSearch
```
rpm -ivh elasticsearch-6.1.0.rpm
```
查找安装路径
```
rpm -ql elasticsearch
```
一般是装在/usr/share/elasticsearch/下。
##### 3.1.3、设置data的目录
创建/data/es-data目录，用于elasticsearch数据的存放
```
mkdir -p /data/es-data
```
修改该目录的拥有者为elasticsearch
```
chown -R elasticsearch:elasticsearch /data/es-data
```
##### 3.1.4、设置log的目录
创建/data/es-log目录，用于elasticsearch日志的存放
```
mkdir -p /log/es-log
```
修改该目录的拥有者为elasticsearch 
```
chown -R elasticsearch:elasticsearch /log/es-log
```
##### 3.1.5、修改配置文件elasticsearch.yml
```
vim /etc/elasticsearch/elasticsearch.yml
```
修改如下内容：
```
#设置data存放的路径为/data/es-data
path.data: /data/es-data

#设置logs日志的路径为/log/es-log
path.logs: /log/es-log

#设置内存不使用交换分区
bootstrap.memory_lock: false
#配置了bootstrap.memory_lock为true时反而会引发9200不会被监听，原因不明

#设置允许所有ip可以连接该elasticsearch
network.host: 0.0.0.0

#开启监听的端口为9200
http.port: 9200

#增加新的参数，为了让elasticsearch-head插件可以访问es (5.x版本，如果没有可以自己手动加)
http.cors.enabled: true
http.cors.allow-origin: "*"
```
##### 3.1.6、启动elasticsearch
启动
```
systemctl start elasticsearch
```
查看状态
```
systemctl status elasticsearch
```
设置开机启动
```
systemctl enable elasticsearch
```
启动成功之后，测试服务是否开启
```
curl -X GET http://localhost:9200
```
返回如下信息，说明安装、启动成功了
![](http://www.justdojava.com/assets/images/2019/java/image-jay/d47fa4b93f724cb0b8c2a068cb00ef18.jpg)
#### 3.2、Logstash安装
##### 3.2.1、安装logstash
```
rpm -ivh logstash-6.1.0.rpm
```
##### 3.2.2、设置data的目录
创建/data/ls-data目录，用于logstash数据的存放 
```
mkdir -p /data/ls-data
```
修改该目录的拥有者为logstash
```
chown -R logstash:logstash /data/ls-data
```
##### 3.2.3、设置log的目录
创建/data/ls-log目录，用于logstash日志的存放
```
mkdir -p /log/ls-log
```
修改该目录的拥有者为logstash
```
chown -R logstash:logstash /log/ls-log
```
##### 3.2.4、设置conf.d的目录，创建配置文件
```
#进入logstash目录
cd /etc/logstash

#创建conf.d的目录
mkdir conf.d
```
创建配置文件，日志内容输出到elasticsearch中，如下
```
vim /etc/logstash/conf.d/logstash.conf
```
内容如下：
```
input {
  beats {
    port => 5044
    codec => plain {
          charset => "UTF-8"
    }
  }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```
##### 3.2.5、修改配置文件logstash.yml
```
vim /etc/logstash/logstash.yml
```
内容如下：
```
# 设置数据的存储路径为/data/ls-data
path.data: /data/ls-data

# 设置管道配置文件路径为/etc/logstash/conf.d
path.config: /etc/logstash/conf.d

# 设置日志文件的存储路径为/log/ls-log
path.logs: /log/ls-log
```
##### 3.2.6、启动logstash
启动
```
systemctl start logstash
```
查看
```
systemctl status logstash
```
设置开机启动
```
systemctl enable logstash
```

##### 3.2.7、测试logstash
`--config.test_and_exit`表示，检查测试创建的logstash.conf配置文件，是否有问题，如果没有问题，执行之后，**显示Configuration OK 证明配置成功！**

```
/usr/share/logstash/bin/logstash  -f /etc/logstash/conf.d/logstash.conf --config.test_and_exit
```

**如果报错：WARNING: Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. You can specify the path using –path.settings. **

解决办法：
```
cd /usr/share/logstash

ln -s /etc/logstash ./config
```
##### 3.2.8、logstash指定配置进行运行
指定`logstash.conf`配置文件，以后台的方式运用，执行这段命令之后，需要回车一下
```
nohup /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/logstash.conf &
```
检查logstash是否启动
```
ps -ef|grep logstash
```
显示如下信息，说明启动了

![](http://www.justdojava.com/assets/images/2019/java/image-jay/a29ab2058ff04df4bac898a8759f1a47.jpg)

#### 3.3、kibana安装
##### 3.3.1、安装kibana
```
rpm -ivh kibana-6.1.0-x86_64.rpm
```
搜索rpm包
```
rpm -ql kibana
```
默认是装在/usr/share/kibana/下。
##### 3.3.2、修改kibana.yml
修改kibana的配置文件
```
vim /etc/kibana/kibana.yml
```
内容如下：
```
#kibana页面映射在5601端口
server.port: 5601

#允许所有ip访问5601端口
server.host: "0.0.0.0"

#elasticsearch所在的ip及监听的地址
elasticsearch.url: "http://localhost:9200"
```
##### 3.3.3、启动kibana
启动
```
systemctl start kibana
```
查看状态
```
systemctl status kibana
```
设置开机启动 
```
systemctl enable kibana
```
#### 3.4、配置nginx 访问
##### 3.4.1、安装nginx和http用户认证工具
```
yum -y install epel-release
yum -y install nginx httpd-tools
```
##### 3.4.2、修改nginx配置
```
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
vim /etc/nginx/nginx.conf
```
将`location`配置部分，注释掉

![](http://www.justdojava.com/assets/images/2019/java/image-jay/1d68de127119499384f51c28a8c7b936.jpg)

创建`kibana.conf`文件
```
vim /etc/nginx/conf.d/kibana.conf
```
内容如下：
```
server {
    listen 8000; #修改端口为8000

    server_name kibana;

    #auth_basic "Restricted Access";
    #auth_basic_user_file /etc/nginx/kibana-user;

    location / {
        proxy_pass http://127.0.0.1:5601; #代理转发到kibana
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
重新加载配置文件
```
systemctl reload nginx
```
到这一步，elk基本配置完了，输入如下命令，启动服务
```
# 启动ELK和nginx
systemctl restart elasticsearch logstash kibana nginx

#查看ELK和nginx启动状态
systemctl status elasticsearch logstash kibana nginx
```
在浏览器输入ip:8000, 就可以访问了kibana
### 04、Linux节点服务器安装配置filebeat
#### 4.1、安装filebeat
```
yum localinstall -y filebeat-6.2.4-x86_64.rpm
```
#### 4.2、启动kibana
启动
```
systemctl start filebeat
```
查看状态
```
systemctl status filebeat
```
设置开机启动
```
systemctl enable filebeat
```
#### 4.3、修改配置Filebeat
编辑filebeat.yml文件
```
vim /etc/filebeat/filebeat.yml
```
```
#============= Filebeat prospectors ===============
filebeat.prospectors:
- input_type: log
  enabled: true ＃更改为true以启用此prospectors配置。
  paths:  #支持配置多个文件收集目录
    #- /var/log/*.log
    - /var/log/messages
#==================== Outputs =====================
#------------- Elasticsearch output ---------------
#output.elasticsearch:
  # Array of hosts to connect to.
  #hosts: ["localhost:9200"]
#---------------- Logstash output -----------------
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]
```

注意：注释掉`Elasticsearch output`下面的部分，将Filebeat收集到的日志输出到 `Logstash output`

最后重启服务
```
systemctl restart filebeat
```
### 4.4、登录kibana，创建索引，并且搜集数据
![](http://www.justdojava.com/assets/images/2019/java/image-jay/0dfff467171a4d32be92610ac0538ae8.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image-jay/5f6804d11cdd41c1869531057de36d1a.png)

![](http://www.justdojava.com/assets/images/2019/java/image-jay/674218b93255470c845f111bb94c09b3.png)

ELK+ Filebeat的安装，到此，就基本结束了，以上只是简单的部署完了。

**但是，还满足不了需求，比如，一台服务器，有多个日志文件路径，改怎么配置，接下来，我们来分类创建索引！**
### 05、Filebeat收集多个tomcat目录下的日志
#### 5.1、修改filebeat.yml配置文件
```
vim /etc/filebeat/filebeat.yml
```
配置多个`paths`收集路径，并且使用`fields`标签配置自定义标签`log_topics`,以方便做索引判断，如下：
```
filebeat.prospectors:

- type: log
  enabled: true
  paths:
    - /usr/tomcat7-ysynet/logs/default/common_monitor.log
  
  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2},[0-9]{3}'
  multiline.negate: true
  multiline.match: after

  fields:
    log_topics: spd-ysynet

- type: log
  enabled: true
  paths:
    - /usr/tomcat7-ysynet-sync/logs/default/common_monitor.log

  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2},[0-9]{3}'
  multiline.negate: true
  multiline.match: after

  fields:
    log_topics: spd-ysynet-sync
- type: log
  enabled: true
  paths:
    - /usr/tomcat7-hscm/logs/default/common_monitor.log
  
  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2},[0-9]{3}'
  multiline.negate: true
  multiline.match: after

  fields:
    log_topics: spd-hscm
- type: log
  enabled: true
  paths:
    - /usr/tomcat7-hscm-sync/logs/default/common_monitor.log
  
  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2},[0-9]{3}'
  multiline.negate: true
  multiline.match: after
  
  fields:
    log_topics: spd-hscm-sync
```
filebeat安装包所在服务器有`tomcat7-ysynet`、`tomcat7-ysynet-sync`、`tomcat7-hscm`、`tomcat7-hscm-sync`，4个`tomcat`，在`fields`下分别创建不同的`log_topics`，`log_topics`属于标签名，可以自定义，然后创建不同的值！

#### 5.2、修改logstash.conf配置文件
接着，修改`logstash`中的配置文件，创建索引，将其输出
```
vim /etc/logstash/conf.d/logstash.conf
```
内容如下：
```
input{
    beats {
        port => 5044
        codec => plain {
          charset => "UTF-8"
        }
    }
}

filter{
    mutate{
        remove_field => "beat.hostname"
        remove_field => "beat.name"
        remove_field => "@version"
        remove_field => "source"
        remove_field => "beat"
        remove_field => "tags"
        remove_field => "offset"
        remove_field => "sort"
    }
}

output{
    if [fields][log_topics] == "spd-ysynet"  {
        stdout { codec => rubydebug }
        elasticsearch {
            hosts => ["localhost:9200"]
            index => "spd-ysynet-%{+YYYY.MM.dd}"
        }
    }
    if [fields][log_topics] == "spd-ysynet-sync"  {
        stdout { codec => rubydebug }
        elasticsearch {
            hosts => ["localhost:9200"]
            index => "spd-ysynet-sync-%{+YYYY.MM.dd}"
        }
    }
    if [fields][log_topics] == "spd-hscm"  {
        stdout { codec => rubydebug }
        elasticsearch {
            hosts => ["localhost:9200"]
            index => "spd-hscm-%{+YYYY.MM.dd}"
        }
    }
    if [fields][log_topics] == "spd-hscm-sync"  {
        stdout { codec => rubydebug }
        elasticsearch {
            hosts => ["localhost:9200"]
            index => "spd-hscm-sync-%{+YYYY.MM.dd}"
        }
    }
}
```
其中`filter`表示过滤的意思，在`output`中使用`filebeat`中配置的`fields`信息，方便创建不同的索引！
```
#表示，输出到控制台
stdout { codec => rubydebug }

#elasticsearch表示，输出到elasticsearch中，index表示创建索引的意思
elasticsearch {
   hosts => ["localhost:9200"]
   index => "spd-hscm-sync-%{+YYYY.MM.dd}"
}
```
#### 5.3、测试修改的logstash.conf配置文件
输入如下命令，检查`/etc/logstash/conf.d/logstash.conf`文件，是否配置异常！
```
/usr/share/logstash/bin/logstash  -f /etc/logstash/conf.d/logstash.conf --config.test_and_exit
```
输入结果，显示`Configuration OK`，就表示没问题！
![](http://www.justdojava.com/assets/images/2019/java/image-jay/78161416adcf4d5ababe035be81a879c.jpg)

#### 5.4、重启相关服务
最后，重启`filebeat `
```
systemctl restart filebeat
```

关闭`logstash`服务
```
systemctl stop logstash
```
以指定的配置文件，启动`logstash`，输入如下命令，回车就ok了，执行完之后也可以用`ps -ef|grep logstash`命令查询`logstash`是否启动成功！
```
nohup /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/logstash.conf &
```
#### 5.5、登录kibana，创建索引
登录kibana，以同样的操作，页面创建索引，查询收集的日志，以下是小编的测试服务器搜集的信息

![](http://www.justdojava.com/assets/images/2019/java/image-jay/e74a87e5b86b4911b31d6f571ebcbade.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image-jay/737c60f0582a4457ad4d52715d3ef750.jpg)

第3、4、5步骤，是筛选elasticsearch今天收集的日志信息！

![](http://www.justdojava.com/assets/images/2019/java/image-jay/490382172d89469683de5f792f8a9d58.jpg)
### 06、总结
整个安装过程已经介绍完了，安装比较简单，复杂的地方就是配置了，尤其是`logstash`、`kibana`、`nginx`、`filebeat`，这几个部分，看了网上很多的介绍，elk配置完之后，外网无法访问`kibana`，使用`nginx`代理到`kibana`，页面就出来了；同时，在配置`filebeat`多个路径的时候，`logstash`也配置了输出索引，但是就是没有日志出来，页面检查说`Elasticsearch `没有找到数据，最后才发现，一定要让`logstash`指定`/etc/logstash/conf.d/logstash.conf`配置文件，进行启动，那么就有日志出来了，整篇文章，可能有很多写的不到位的地方，请大家多多包含，也可以直接给我们留言，以便修正！
### 07、参考
[IBM - ELK 架构和 Filebeat 工作原理详解](https://www.ibm.com/developerworks/cn/opensource/os-cn-elk-filebeat/index.html)