---
layout: post
categories: java
title: 使用uuid作为数据库主键，被技术总监怼了一顿！
tagline: by 炸鸡可乐
tags: 
  - 炸鸡可乐
---

看完本文，你一定会有所收获

<!--more-->

### 一、摘要
在日常开发中，数据库中主键id的生成方案，主要有三种

* 数据库自增ID
* 采用随机数生成不重复的ID
* 采用jdk提供的uuid

对于这三种方案，我发现在数据量少的情况下，没有特别的差异，但是当单表的数据量达到百万级以上时候，他们的性能有着显著的区别，光说理论不行，还得看实际程序测试，今天小编就带着大家一探究竟！

### 二、程序实例
首先，我们在本地数据库中创建三张单表`tb_uuid_1`、`tb_uuid_2`、`tb_uuid_3`，同时设置`tb_uuid_1`表的主键为自增长模式，脚本如下：
```sql
CREATE TABLE `tb_uuid_1` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='主键ID自增长';
```
```sql
CREATE TABLE `tb_uuid_2` (
  `id` bigint(20) unsigned NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='主键ID随机数生成';
```
```sql
CREATE TABLE `tb_uuid_3` (
  `id` varchar(50)  NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='主键采用uuid生成';
```
下面，我们采用`Springboot + mybatis`来实现插入测试。
#### 2.1、数据库自增
以数据库自增为例，首先编写好各种实体、数据持久层操作，方便后续进行测试

```java
/**
 * 表实体
 */
public class UUID1 implements Serializable {

    private Long id;

    private String name;
	 
	 //省略set、get
}
```
```java
/**
 * 数据持久层操作
 */
public interface UUID1Mapper {

    /**
     * 自增长插入
     * @param uuid1
     */
    @Insert("INSERT INTO tb_uuid_1(name) VALUES(#{name})")
    void insert(UUID1 uuid1);
}
```
```java
/**
 * 自增ID，单元测试
 */
@Test
public void testInsert1(){
    long start = System.currentTimeMillis();
    for (int i = 0; i < 1000000; i++) {
        uuid1Mapper.insert(new UUID1().setName("张三"));
    }
    long end = System.currentTimeMillis();
    System.out.println("花费时间：" +  (end - start));
}
```
#### 2.2、采用随机数生成ID
这里，我们**采用`twitter`的雪花算法来实现随机数ID的生成**，工具类如下：
```java
public class SnowflakeIdWorker {

    private static SnowflakeIdWorker instance = new SnowflakeIdWorker(0,0);

    /**
     * 开始时间截 (2015-01-01)
     */
    private final long twepoch = 1420041600000L;
    /**
     * 机器id所占的位数
     */
    private final long workerIdBits = 5L;
    /**
     * 数据标识id所占的位数
     */
    private final long datacenterIdBits = 5L;
    /**
     * 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
     */
    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);
    /**
     * 支持的最大数据标识id，结果是31
     */
    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
    /**
     * 序列在id中占的位数
     */
    private final long sequenceBits = 12L;
    /**
     * 机器ID向左移12位
     */
    private final long workerIdShift = sequenceBits;
    /**
     * 数据标识id向左移17位(12+5)
     */
    private final long datacenterIdShift = sequenceBits + workerIdBits;
    /**
     * 时间截向左移22位(5+5+12)
     */
    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
    /**
     * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
     */
    private final long sequenceMask = -1L ^ (-1L << sequenceBits);
    /**
     * 工作机器ID(0~31)
     */
    private long workerId;
    /**
     * 数据中心ID(0~31)
     */
    private long datacenterId;
    /**
     * 毫秒内序列(0~4095)
     */
    private long sequence = 0L;
    /**
     * 上次生成ID的时间截
     */
    private long lastTimestamp = -1L;
    /**
     * 构造函数
     * @param workerId     工作ID (0~31)
     * @param datacenterId 数据中心ID (0~31)
     */
    public SnowflakeIdWorker(long workerId, long datacenterId) {
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }
    /**
     * 获得下一个ID (该方法是线程安全的)
     * @return SnowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();
        // 如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(
                    String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }
        // 如果是同一时间生成的，则进行毫秒内序列
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            // 毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        }
        // 时间戳改变，毫秒内序列重置
        else {
            sequence = 0L;
        }
        // 上次生成ID的时间截
        lastTimestamp = timestamp;
        // 移位并通过或运算拼到一起组成64位的ID
        return ((timestamp - twepoch) << timestampLeftShift) //
                | (datacenterId << datacenterIdShift) //
                | (workerId << workerIdShift) //
                | sequence;
    }
    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     * @param lastTimestamp 上次生成ID的时间截
     * @return 当前时间戳
     */
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }
    /**
     * 返回以毫秒为单位的当前时间
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }

    public static SnowflakeIdWorker getInstance(){
        return instance;
    }


    public static void main(String[] args) throws InterruptedException {
        SnowflakeIdWorker idWorker = SnowflakeIdWorker.getInstance();
        for (int i = 0; i < 10; i++) {
            long id = idWorker.nextId();
            Thread.sleep(1);
            System.out.println(id);
        }
    }
}
```
其他的操作，与上面类似。
#### 2.3、uuid
同样的，uuid的生成，我们事先也可以将工具类编写好：
```java
public class UUIDGenerator {

    /**
     * 获取uuid
     * @return
     */
    public static String getUUID(){
        return UUID.randomUUID().toString();
    }
}
```
最后的单元测试，代码如下：
```java
@RunWith(SpringRunner.class)
@SpringBootTest()
public class UUID1Test {

    private static final Integer MAX_COUNT = 1000000;

    @Autowired
    private UUID1Mapper uuid1Mapper;

    @Autowired
    private UUID2Mapper uuid2Mapper;

    @Autowired
    private UUID3Mapper uuid3Mapper;

    /**
     * 测试自增ID耗时
     */
    @Test
    public void testInsert1(){
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
            uuid1Mapper.insert(new UUID1().setName("张三"));
        }
        long end = System.currentTimeMillis();
        System.out.println("自增ID，花费时间：" +  (end - start));
    }

    /**
     * 测试采用雪花算法生产的随机数ID耗时
     */
    @Test
    public void testInsert2(){
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
            long id = SnowflakeIdWorker.getInstance().nextId();
            uuid2Mapper.insert(new UUID2().setId(id).setName("张三"));
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间：" +  (end - start));
    }

    /**
     * 测试采用UUID生成的ID耗时
     */
    @Test
    public void testInsert3(){
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
            String id = UUIDGenerator.getUUID();
            uuid3Mapper.insert(new UUID3().setId(id).setName("张三"));
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间：" +  (end - start));
    }
}
```
### 三、性能测试
程序环境搭建完成之后，啥也不说了，直接撸起袖子，将单元测试跑起来！

首先测试一下，插入100万数据的情况下，三者直接的耗时结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-mysql-uuid/01.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-mysql-uuid/02.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-mysql-uuid/03.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-mysql-uuid/04.jpg)


在原有的数据量上，我们继续插入30万条数据，三者耗时结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-mysql-uuid/05.jpg)

可以看出在数据量 100W 左右的时候，uuid的插入效率垫底，随着插入的数据量增长，uuid 生成的ID插入呈直线下降！

时间占用量总体效率排名为：**自增ID > 雪花算法生成的ID >> uuid生成的ID**。



**在数据量较大的情况下，为什么uuid生成的ID远不如自增ID呢**？

关于这点，我们可以从 mysql 主键存储的内部结构来进行分析。

#### 3.1、自增ID内部结构

自增的主键的值是顺序的，所以 Innodb 把每一条记录都存储在一条记录的后面。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-mysql-uuid/06.png)

当达到页面的最大填充因子时候(innodb默认的最大填充因子是页大小的15/16,会留出1/16的空间留作以后的修改)，会进行如下操作：

* 下一条记录就会写入新的页中，一旦数据按照这种顺序的方式加载，主键页就会近乎于顺序的记录填满，提升了页面的最大填充率，不会有页的浪费
* 新插入的行一定会在原有的最大数据行下一行，mysql定位和寻址很快，不会为计算新行的位置而做出额外的消耗

#### 3.2、使用uuid的索引内部结构
uuid相对顺序的自增id来说是毫无规律可言的，新行的值不一定要比之前的主键的值要大，所以innodb无法做到总是把新行插入到索引的最后，而是需要为新行寻找新的合适的位置从而来分配新的空间。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-mysql-uuid/07.png)

这个过程需要做很多额外的操作，数据的毫无顺序会导致数据分布散乱，将会导致以下的问题：

* 写入的目标页很可能已经刷新到磁盘上并且从缓存上移除，或者还没有被加载到缓存中，innodb在插入之前不得不先找到并从磁盘读取目标页到内存中，这将导致大量的随机IO
* 因为写入是乱序的,innodb不得不频繁的做页分裂操作,以便为新的行分配空间,页分裂导致移动大量的数据，一次插入最少需要修改三个页以上
* 由于频繁的页分裂，页会变得稀疏并被不规则的填充，最终会导致数据会有碎片

在把值载入到聚簇索引(innodb默认的索引类型)以后，有时候会需要做一次`OPTIMEIZE TABLE`来重建表并优化页的填充，这将又需要一定的时间消耗。

因此，在选择主键ID生成方案的时候，尽可能别采用uuid的方式来生成主键ID，随着数据量越大，插入性能会越低！

### 四、总结
在实际使用过程中，推荐使用主键自增ID和雪花算法生成的随机ID。

但是使用自增ID也有缺点：

1、别人一旦爬取你的数据库，就可以根据数据库的自增id获取到你的业务增长信息，很容易进行数据窃取。
2、其次，对于高并发的负载，innodb在按主键进行插入的时候会造成明显的锁争用，主键的上界会成为争抢的热点，因为所有的插入都发生在这里，并发插入会导致间隙锁竞争。

总结起来，如果业务量小，推荐采用自增ID，如果业务量大，推荐采用雪花算法生成的随机ID。

本篇文章主要从实际程序实例出发，讨论了三种主键ID生成方案的性能差异，
鉴于笔者才疏学浅，可能也有理解不到位的地方，欢迎网友们批评指出！

### 五、参考
1、[方志明 - 使用雪花id或uuid作为Mysql主键，被老板怼了一顿！](https://mp.weixin.qq.com/s/YbKyjlcniEtSFO2WPqacuA)

