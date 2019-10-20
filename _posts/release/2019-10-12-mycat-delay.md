---
layout: post
title: 生产上遇到的一例mycat读写分离延时问题
category: mysql
tagline: by 小马
tags: [mycat,读写分离,延时]
---

* content
{:toc}

场景是这样的，我们的支付系统在一笔支付完成后，需要发出通知给到商户。支付完成的消息通过消息队列发送给通知的服务。
<!--more-->

## 问题描述


```java
            notifyPersist.saveNotifyRecord(notifyRecord);
            notifyRecord = rpNotifyService.getNotifyByMerchantNoAndMerchantOrderNoAndNotifyType(notifyRecord.getMerchantNo(), notifyRecord.getMerchantOrderNo(), notifyRecord.getNotifyType());
            notifyQueue.addElementToList(notifyRecord);
```
三行代码。我解释下，通知服务收到消息解析成 notifyRecord 对象，然后存入数据库，然后马上取出添加到任务队列。另外又一个独立的线程去处理这个任务队列。

项目上线后，客户反馈偶尔会出现收不到通知的情况。


## 问题排查

经过日志跟踪，我发现是在上述代码的第二行，查询记录的时候数据库返回null，也就是没有查询到记录。导致任务队列没有该笔支付的通知任务。

一开始觉得非常不解，因为通过日志我们发现第一行代码是执行成功的，既然插入成功了，没理由查询不到啊？？而且我去数据库看确定是有这条记录的。莫非是见鬼了！！

我先说下

慢慢静下心来思考，结合这个现象是偶发性的，我想到有没有可能是因为读写分离延时造成的。我先说下我们的存储架构：


centos 6.5 64位操作系统。mycat1.6版本，mysql 5.6.21

数据库服务器有两台，一台主，一台从，利用mycat配置了主从复制和读写分离。写操作在主机上，读操作在从机上。如下图所示：

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/mycat/1.png)

有没有可能是主库上插入成功后，从库还没有来得急同步完成，应用就马上查询，所以查不到。为了验证我的想法，我把代码改成下面这样发布到生产先看看效果：

```java
notifyPersist.saveNotifyRecord(notifyRecord);
Thread.sleep(300);
notifyRecord = rpNotifyService.getNotifyByMerchantNoAndMerchantOrderNoAndNotifyType(notifyRecord.getMerchantNo(), notifyRecord.getMerchantOrderNo(), notifyRecord.getNotifyType());
notifyQueue.addElementToList(notifyRecord);
```

思路也很简单粗暴，就是插入之后不马上读，而是等一会让从库同步完再读。发布后，跑了一个段时间，没有反馈异常。证明我怀疑的没错，问题确实出现在mycat读写分离延时上。

## 解决方案
当然，上面定位问题的sleep也勉强算是一个解决方案。只不过感觉比较low，原理很好理解。在插入数据和查询数据中间加一个sleep()方法，相当于等一会再读。如果应用对时效要求不高，
此方法也不失唯一种快速有效的方案。

找到了问题的根源我就去mycat的官网和相关论坛寻找解决方案。

功夫不负有心人，后来我查到相关资料，mycat可以通过注解的方式指定某些 sql 语句强制走主库，如下所示：

```
<select id="listBy" parameterType="java.util.Map" resultMap="BaseResultMap">
    /**mycat:db_type=master*/select * from rp_notify_record
    <where>
      <include refid="condition_sql" />
    </where>
    <![CDATA[ order by create_time desc]]>
  </select>
```
注意到 /** */里面的注解，通过指定 db_type=master 保证后面的 sql 语句走主库。这样就不存在延时导致查询不到的问题了。


