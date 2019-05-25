---
layout: post
category: mysql
title: 面试你应该知道的 MySQL 的锁
tagline: by 子悠
tags: 
  - mysql
published: true
---

### 背景
数据库的锁是在多线程高并发的情况下用来保证数据稳定性和一致性的一种机制。MySQL 根据底层存储引擎的不同，锁的支持粒度和实现机制也不同。MyISAM 只支持表锁，InnoDB 支持行锁和表锁。目前 MySQL 默认的存储引擎是 InnoDB，这里主要介绍 InnoDB 的锁。
 <!--more-->
### InnoDB 存储引擎
使用 InnoDB 的两大优点：一是支持事务；二是支持行锁。

#### MySQL 的事务

在高并发的情况下事务的并发处理会带来几个问题
1.  脏读：**指在事务 A 处理过程里读取到了事务 B 未提交的事务中的数据。**比如在转账的例子中：小 A 开了一个事务给小 B 转了1000 块，还没提交事务的时候就跟小 B 说，钱已经到账了。这个时候小 B 去看了一下余额，发现果真到账了（然后就开开心心刷抖音去了），这个时候小 A 回滚了事务，把 1000 块又搞回去了。小 B 刷完抖音再去看下余额，发现钱又不见了。

2. 不可重复读：**指在一个事务执行的过程中多次查询某一数据的时候结果不一致的现象，由于在执行的过程中被另一个事务修改了这个数据并提交了事务。**比如：事务 A 第一次读小明的年龄是 18 岁，此时事务 B 将小明的年龄改成了 20 并提交了，这个时候事务 A 再次读取小明的年龄发现是 20，这就是同一条数据不可重复读。
    
3. 幻读：**幻读通常指的是对一批数据的操作完成后，有其他事务又插入了满足条件的数据导致的现象。**比如：事务 A 将数据库性别为男的状态都改成1 表示有钱人，这个时候事务 B 又插入了一条状态为 0 没钱人的记录，这个时候，用户再查看刚刚修改的数据时就会发现还有一行没有修改，这就出现了幻读。幻读往往针对 insert 操作，脏读和不可重复读针对 select 操作。

由于高并发事务带来这几个问题，所以就产生了事务的隔离级别

- Read uncommitted (读未提交)：最低级别，任何情况都无法保证。
- Read committed (读已提交)：可避免脏读的发生。
- Repeatable read (可重复读)：可避免脏读、不可重复读的发生。
- Serializable (串行化)：可避免脏读、不可重复读、幻读的发生。

#### InnoDB 常见的几种锁机制
1. 共享锁和独占锁（Shared and Exclusive Locks），InnoDB 通过共享锁和独占所两种方式实现了标准的行锁。共享锁（S 锁）：允许事务获得锁后去读数据，独占锁（X 锁）：允许事务获得锁后去更新或删除数据。一个事务获取的共享锁 S 后，允许其他事务获取 S 锁，此时两个事务都持有共享锁 S，但是不允许其他事务获取 X 锁。如果一个事务获取的独占锁（X），则不允许其他事务获取 S 或者 X 锁，必须等到该事务释放锁后才可以获取到。大家可以通过下面的 SQL 感受下。

```sql
# T1

START TRANSACTION WITH CONSISTENT SNAPSHOT;

SELECT * FROM category WHERE category_no = 2 lock in SHARE mode; //共享锁

SELECT * FROM category WHERE category_no = 2 for UPDATE; //独占锁

COMMIT;

# T2

START TRANSACTION WITH CONSISTENT SNAPSHOT;

SELECT * FROM category WHERE category_no = 2 lock in SHARE mode; //共享锁

UPDATE category set category_name = '动漫' WHERE category_no = 2; //独占锁

COMMIT;

```

2. 意向锁（Intention Locks），上面说过 InnoDB 支持行锁和表锁，意向锁是一种表级锁，用来指示接下来的一个事务将要获取的是什么类型的锁（共享还是独占）。意向锁分为意向共享锁（IS）和意向独占锁（IX），依次表示接下来一个事务将会获得共享锁或者独占锁。意向锁不需要显示的获取，在我们获取共享锁或者独占锁的时候会自动的获取，意思也就是说，如果要获取共享锁或者独占锁，则一定是先获取到了意向共享锁或者意向独占锁。
意向锁不会锁住任何东西，除非有进行全表请求的操作，否则不会锁住任何数据。存在的意义只是用来表示有事务正在锁某一行的数据，或者将要锁某一行的数据。
> 原文：Intention locks are table-level locks that indicate which type of lock (shared or exclusive) a transaction requires later for a row in a table.

3. 记录锁（record Locks），锁住某一行，如果表存在索引，那么记录锁是锁在索引上的，如果表没有索引，那么 InnoDB 会创建一个隐藏的聚簇索引加锁。所以我们在进行查询的时候尽量采用索引进行查询，这样可以降低锁的冲突。

4. 间隙锁（Gap Locks），间隙锁是一种记录行与记录行之间存在空隙或在第一行记录之前或最后一行记录之后产生的锁。间隙锁可能占据的单行，多行或者是空记录。通常的情况是我们采用范围查找的时候，比如在学生成绩管理系统中，如果此时有学生成绩 60，72，80，95，一个老师要查下成绩大于 72 的所有同学的信息，采用的语句是` select * from student where grade > 72 for update`，这个时候 InnoDB 锁住的不仅是 80，95，而是所有在 72-80，80-95，以及 95 以上的所有记录。为什么会
这样呢？实际上是因为如果不锁住这些行，那么如果另一个事务在此时插入了一条分数大于 72 的记录，那会导致第一次的事务两次查询的结果不一样，出现了幻读。所以为了在满足事务隔离级别的情况下需要锁住所有满足条件的行。

5. Next-Key Locks，NK 是一种记录锁和间隙锁的组合锁。是 3 和 4 的组合形式，既锁住行也锁住间隙。并且采用的左开右闭的原则。InnoDB 对于查询都是采用这种锁的。

举个例子

```sql

CREATE TABLE `a` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `uid` int(10) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_uid` (`uid`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO `a`(uid) VALUES (1);
INSERT INTO `a`(uid) VALUES (2);
INSERT INTO `a`(uid) VALUES (3);
INSERT INTO `a`(uid) VALUES (6);
INSERT INTO `a`(uid) VALUES (10);

# T1
START TRANSACTION WITH CONSISTENT SNAPSHOT; //1

SELECT * FROM a WHERE uid = 6 for UPDATE; //2

COMMIT;  //5


# T2
START TRANSACTION WITH CONSISTENT SNAPSHOT;  //3

INSERT INto a(uid) VALUES(11);
INSERT INto a(uid) VALUES(5);  //4
INSERT INto a(uid) VALUES(7);
INSERT INto a(uid) VALUES(8);
INSERT INto a(uid) VALUES(9);

SELECT * FROM a WHERE uid = 6 for UPDATE;

COMMIT;

ROLLBACK;

```
按照上面 1，2，3，4 的顺序执行会发现第 4 步被阻塞了，必须执行完第 5 步后才能插入成功。这里我们会很奇怪明明锁住的是uid=6 的这一行，为什么不能插入 5 呢？原因就是这里采用了 next-key 的算法，锁住的是（3,10）整个区间。感兴趣的可以试一下。

### 小结
今天给大家分享了一下 MySQL 的 InnoDB 的事务以及锁的一些知识，通过自己的实际上手实践对这块更加熟悉了，希望大家在看的时候也可以动手试试，这样更能体会，理解的更深刻。
