---
layout: post
categories: mysql
title: 手把手教你给 SQL 做个优化
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
在开始之前，咱们要知道：如果我的 SQL 语句执行的足够快，还有没有必要去做优化？

完全没有必要对吧
<!-- more -->

所以我们一般说，要给 SQL 做个优化，那肯定就是这条 SQL 语句执行的比较慢了

那么，为什么它会执行比较慢呢？

# SQL 语句执行较慢的 3 个原因

## 没有建立索引，或者索引失效导致了 SQL 语句执行较慢

这个应该是比较好理解的，如果数据比较多，在千万级别以上，然后呢又没有建立索引，在这千万级别的数据中查找你想要的内容，简直就是在肉搏啊（哎呦，可了不得，竟然敢肉搏

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/11-可了不得.jpg)

索引失效这块内容说起来就比较多了，比如在查询的时候，让 like 通配符在前面了，比如经常念叨的“最左匹配原则”，又比如我们在查询条件中使用 or ，而且 or 前后条件中有一个列没有索引，等等这些情况都会导致索引失效

## 锁等待

常用的存储引擎主要有 InnoDB 和 MyISAM 这两种了，前者支持行锁和表锁，后者就只支持表锁

如果数据库操作都是基于表锁的话，意思就是说，现在有个更新操作，就会把整张表锁起来，那么查询的操作都不被允许，所以就不要说提高系统的并发性能了

- 聪明的你肯定就知道了，既然 MyISAM 只支持表锁，那么使用 InnoDB 不就好了？你以为 InnoDB 的行锁不会升级成表锁嘛？ too young too simple  ！

- 如果对一张表进行大量的更新操作， mysql 就觉得你这样用会让事务的执行效率降低，到最后还是会导致性能下降，这样的话，还不如把你的行锁升级成表锁呢

- 还有一点，行锁可是基于索引加的锁，在执行更新操作时，条件索引都失效了，那么这个锁也会执行从行锁升级为表锁

## 不恰当的 SQL 语句

这个也比较常见了，啥是不恰当的 SQL 语句呢？就比如，明明你需要查找的内容是 name ， age ，但是呢，为了省事，直接 `select * `，或者在 order by 时，后面的条件不是索引字段，这就是不恰当的 SQL 语句

# 优化 SQL 语句

在知道了 SQL 语句执行比较慢的原因之后，接下来要做的就是对症下药了

针对 没有索引/索引失效 这块，最有效的办法就是 EXPLAIN 语法了，那你知不知道 Show Profile 也可以嘞

针对 锁等待 这块，没办法了，只能自己多注意

针对 不恰当的 SQL 语句 这块，介绍几个常用的 SQL 优化，比如分页查询怎么优化一下可以查询的更快一些呀，你不是说 `select * ` 不是正确的打开方式嘛？那什么是正确的 select 方式呢？别急嘛，阿粉下面都会说到的

废话不多说，咱们开始了

# 先来个表

为了确保优化后的结果和我写的一样（起码 90% 是相符的

所以咱们用一样的数据库好不好？乖~

首先建个 demo 的数据库

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/12-建库.jpg)

接下来咱们建表，就建个非常简单的表好不好

```java
CREATE TABLE demo.table(
	id int(11) NOT NULL,
	a int(11) DEFAULT NULL,
	b int(11) DEFAULT NULL,
	PRIMARY KEY(id)
) ENGINE = INNODB
```

然后插入 10 万条数据

```java
DROP PROCEDURE IF EXISTS demo_insert;
CREATE PROCEDURE demo_insert()
BEGIN
    DECLARE i INT; 
		SET i = 1;
    WHILE i <= 100000 DO
        INSERT INTO demo.`table` VALUES (i, i, i);
        SET i = i + 1 ;
    END WHILE;
END;
CALL demo_insert();
```

OK ，准备工作做好了，接下来开始实战

## 通过 EXPLAIN 分析 SQL 是怎样执行的

只要说 SQL 调优，那就离不开 EXPLAIN 

`EXPLAIN SELECT * FROM `table` WHERE id < 100 ORDER BY a;`

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/13-explain.jpg)

咱们能够看到有好几个参数:

- id :每个执行计划都会有一个 id ，如果是一个联合查询的话，这里就会显示好多个 id

- select_type :表示的是 select 查询类型，常见的就是 SIMPLE (普通查询，也就是没有联合查询/子查询), PRIMARY (主查询), UNION ( UNION 中后面的查询), SUBQUERY (子查询)

- table :执行查询计划的表，在这里我查的就是 table ，所以显示的是 table， 那如果我给 table 起了别名 a ，在这里显示的就是 a

- type :查询所执行的方式，这是咱们在分析 SQL 优化的时候一个非常重要的指标，这个值从好到坏依次是: system > const > eq_ref > ref > range > index > ALL

	- system/const :说明表中只有一行数据匹配，这个时候根据索引查询一次就能找到对应的数据

	- eq_ref :使用唯一索引扫描，这个经常在多表连接里面，使用主键和唯一索引作为关联条件时可以看到

	- ref :非唯一索引扫描，也可以在唯一索引最左原则匹配扫描看到

	- range :索引范围扫描，比如查询条件使用到了 < ， > ， between 等条件

	- index :索引全表扫描，这个时候会遍历整个索引树

	- ALL :表示全表扫描，也就是需要遍历整张表才能找到对应的行

- possible_keys :表示可能使用到的索引

- key :实际使用到的索引

- key_len :使用的索引长度

- ref :关联 id 等信息

- rows :找到符合条件时，所扫描的行数，在这里虽然有 10 万条数据，但是因为索引的缘故，所以扫描了 99 行的数据

- Extra :额外的信息，常见的有以下几种

	- Using where :不用读取表里面的所有信息，只需要通过索引就可以拿到需要的数据，这个过程发生在对表的全部请求列都是同一个索引部分时

	- Using temporary :表示 mysql 需要使用临时表来存储结果集，常见于 group by / order by

	- Using filesort :当查询的语句中包含 order by 操作的时候，而且 order by 后面的内容不是索引，这样就没有办法利用索引完成排序，就会使用"文件排序",就像例子中给出的，建立的索引是 id ， 但是我的查询语句 order by 后面是 a ，没有办法使用索引

	- Using join buffer :使用了连接缓存

	- Using index :使用了覆盖索引

如果对这些参数了解的非常不错，那么 EXPLAIN 这块内容就难不住你了

## Show Profile 分析下 SQL 执行性能

通过 EXPLAIN 分析执行计划，只能说明 SQL 的外部执行情况，如果想要知道 mysql 具体是如何查询的，需要通过 Show Profile 来分析

可以通过 `SHOW PROFILES;` 语句来查询最近发送给服务器的 SQL 语句，默认情况下是记录最近已经执行的 15 条记录，如下图我们可以看到:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/14-showprofile.jpg)

我想看具体的一条语句，看到 Query_ID 了嘛？然后运行下 `SHOW PROFILE FOR QUERY 82;` 这条命令就可以了:

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/15-queryid.jpg)

可以看到，在结果中， Sending data 耗时是最长的，这是因为此时 mysql 线程开始读取数据并且把这些数据返回到客户端，在这个过程中会有大量磁盘 I/O 操作

通过这样的分析，我们就能知道， SQL 语句在查询过程中，到底是 磁盘 I/O 影响了查询速度，还是 System lock 影响了查询速度，知道了病症所在，接下来对症下药就容易多了

## 分页查询怎么可以更快一些

在使用分页查询时，都会使用 limit 关键字

但是对于分页查询，其实还可以优化一步

我这里给出的数据库不是太好，因为它太简单了，看不出来有什么区别，我使用目前项目上正在用的表来做个实验，可以看下区别（使用的 SQL 语句如下面）:

```java
EXPLAIN SELECT * FROM `te_paper_record` ORDER BY id LIMIT 10000, 20;

EXPLAIN SELECT * FROM `te_paper_record` WHERE id >= ( SELECT id FROM `te_paper_record` ORDER BY id LIMIT 10000, 1) LIMIT 20;

```

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/16-不使用子查询.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/17-使用子查询.jpg)

上面一张图片，我没有使用子查询，可以看到执行了 0.033s ，下面的查询语句，我使用了子查询去做优化，能够看到执行了 0.007s ，优化的结果还是很显而易见的

那么，为什么使用了子查询，查询的速度就提上来了呢，这是因为当我们没有使用子查询时，查询到的 10020 行数据都返回回来了，接下来要对这 10020 行数据再进行过滤操作

那可不可以直接就返回需要的 20 行数据呢，这样就不需要再做过滤操作了，直接返回就可以了嘛

你也太聪明了吧。子查询就是在做这件事情

所以查询时间上有了一个很大的优化

## 正确的 select 打开方式

在查询时，有时为了省事，直接使用 `select * from table where id = 1 ` 这样的 SQL 语句，但是这样的写法在一些环境下是会存在一定的性能损耗的

所以最好的 select 查询就是，需要什么字段就查询什么字段

一般在查询时，都会有条件，按照条件查找

这个时候正确的 select 打开方式是什么呢？

如果可以通过主键索引的话， where 后面的条件，优先选择主键索引

为什么呢？这就要知道 MySQL 的存储规则

MySQL 常用的存储引擎有 MyISAM 和 InnoDB ， InnoDB 会创建主键索引，而主键索引属于聚簇索引，也就是在存储数据时，索引是基于 B+ 树构成的，具体的行数据则存储在叶子节点

也就是说，如果是通过主键索引查询的，会直接搜索 B+ 树，从而查询到数据

如果不是通过主键索引查询的，需要先搜索索引树，得到在 B+ 树上的值，再到 B+ 树上搜索符合条件的数据，这个过程就是“回表”

很显然，回表能够产生时间。

这也是为什么建议， where 后面的条件，优先选择主键索引

# 其他调优

看完上面的，心里应该就大概有数了， SQL 调优主要就是建立索引/防止产生锁等待/使用恰当的 SQL 语句去查询

但是，如果问你除了索引，除了上面这些手段，还有没有其他调优方式

啥？竟然还有？！

有的，这就需要跳出来，不要局限在具体的 SQL 语句上了，需要在数据库设计之初就考虑好

比如说，我们常说的要遵循三范式，但是在有的业务场景里面，如果在数据库里面多几个冗余字段的话，可能要比严格遵循三范式带来的性能要好很多。

但是这点就及其考验平时的积累了，阿粉在这里把这一点提出来之后，希望读者们可以看看自己项目上目前用的数据库有没有多余的字段，为什么要这样设计呢？这样多去观察，你的技术能力想不提高都很难

以上，就这样啦~

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/08/18-爱你.gif)