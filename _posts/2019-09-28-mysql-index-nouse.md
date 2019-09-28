---
layout: post
title: 说一个在工作中遇到的mysql索引失效的问题
category: mysql
tagline: by 小马
tags: [mysql,索引,失效,字符串]
---

* content
{:toc}

下面分享的是我在工作中遇到的一个问题。有关 mysql 索引失效的问题。
<!--more-->

处于保密考虑，我拿一个类似的场景举例子。

## 1、现象描述
先说结论。我遇到的问题是，mysql varchar类型的字段，传入的查询条件没有加引号，导致索引失效。

比如我有一张表，结构如下：

```sql
CREATE TABLE `order_test` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `user_id` varchar(32) DEFAULT '',
  `name` varchar(11) DEFAULT '',
  `age` tinyint(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;
```

可以看到 user_id 字段是 varchar 类型，并且这个字段傻姑娘有一个普通索引。

我分别用下面两条语句查询，你看看效果，
```sql
explain SELECT * from order_test t where t.`user_id` = '11223344';
```
结果，

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/mysql-index-nouse/1.png)

```sql
explain SELECT * from order_test t where t.`user_id` = 11223344;
```
结果，


![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/mysql-index-nouse/2.png)

很明显，第二条 sql 语句索引没有生效。

**虽然user_id是字符串类型，但是我们传入一个整型的数字并没有报错，而是 mysql 帮我们做了转换并且进行了全表扫描查询。**

## 2、场景复现
一般我们都是通过 mybatis 操作数据库，我当时遇到的情况就是在 mybatis 的xml里通过map作为入参传递查询字段。举例如下，

xml文件，
```xml
<select id="selectByMap" resultMap="BaseResultMap" parameterType="java.util.Map" >
    select * from order_test where user_id = ${user_id}
  </select>
```

代码层是这样的，
```java
public void test2(){
        Map<String, Object> map = new HashMap<>();
        map.put("user_id", 11223344);
        mapper.selectByMap(map);

    }
```
这种情况下，mysql解析出来的sql是这样的，
```sql
select * from order_test where user_id = 11223344
```
根据第1部分的结论，没有引号的查询是不走索引的。

只所以没有加引号，是因为 `${user_id}` ，如果我用 `#
{user_id}` 就会加上引号了。这个原因就不在本文展开了。


## 3、索引失效的原因
我们做技术，要有刨根问底的精神。知其然还要知其所以然。

索引失效的原因，简单来说就是 mysql 内部的隐士类型转换，导致了优化器执行计划出问题。

mysql底层有个叫优化器的东东，当我们执行一条sql时，会经过优化器确定是否使用索引。也就是说执行计划是在这个阶段确定的。

我借用一张网上的图：

![image](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1569585685231&di=bc113a44e3848ab3219a263d3df3cd4a&imgtype=0&src=http%3A%2F%2Fstatic.codeceo.com%2Fimages%2F2017%2F05%2Fa9078e8653368c9c291ae2f8b74012e74.jpg)

如果对索引字段做函数操作（本例是cast函数做了隐式的转换），可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能。

## 4、总结
下面这句话跟我读三遍：

1. 字符串类型的索引查询语句中必须加单引号，否则MySQL不会使用该索引。
2. MySQL不支持函数索引 ，在开发时要避免在查询条件加入函数。
