---
layout: post
categories: Java
title: mybatis系列之mybatis一级缓存
tagline: by 小九
tags: 
  - 小九
---

> hello~各位读者好，我是鸭血粉丝（大家可以称呼我为「阿粉」）。今天，阿粉带着大家来了解一下 `mybatis` 一级缓存的实现原理。

## 1.上期回顾

首先，我们还是回顾一下上篇文件的类容。毕竟离上次讲 `mybatis`还是过去了很久，汗~。

还是先看下这个测试类，大家还有印象吗：

```java
public class MybatisTest {
    @Test
    public void testSelect() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession session = sqlSessionFactory.openSession();
        try {
            FruitMapper mapper = session.getMapper(FruitMapper.class);
            Fruit fruit = mapper.findById(1L);
            System.out.println(fruit);
        } finally {
            session.close();
        }
    }
}
```

上篇源码分析讲了 `SqlSession` 构建的过程。这次，我们来了解下 `mybatis` 一级缓存的实现原理。

### 2.mybatis缓存

**2.1 缓存的作用**

`mybatis` 缓存的作用就是提升查询的效率和减少数据库的压力。

**2.2 mybatis的缓存类**

mybatis缓存相关的类都在cache包里面，有个 `Cache `的接口，默认实现是 `PerpetualCache` 类。当然，还有一些其他缓存类，是通过装饰器模式实现的。我们来看下包结构：

![](D:\GitHub\justdojava.github.io\assets\images\2019\java\image-xiaojiu\20200427\1.jpg)

然后看下这些缓存类的作用：

* PerpetualCache ：基本缓存类，默认实现。
* LruCache ：LRU策略的缓存，作用是当缓存到达上限时候，删除最近最少使用的缓存。
* FifoCache ： FIFO 策略的缓存，作用是当缓存到达上限时候，删除最先入队的缓存。
* SoftCache ： 带清理策略的缓存，作用是通过JVM 的软引用来实现缓存，当JVM内存不足时，会自动清理掉这些缓存。
* WeakCache ：带清理策略的缓存，作用是通过JVM 的弱引用来实现缓存，当JVM内存不足时，会自动清理掉这些缓存。
* LoggingCache ： 带日志功能的缓存。
* SynchronizedCache ： 同步缓存，基于synchronized 关键字实现，作用是解决并发问。
* BlockingCache ： 阻塞缓存，通过在get/put 方式中加锁，保证只有一个线程操作缓存，基于Java 重入锁实现
* SerializedCache ： 支持序列化的缓存，将对象序列化以后存到缓存中，取出时反序列化。
* ScheduledCache ：定时调度的缓存，在进行get/put/remove/getSize 等操作前，判断缓存时间是否超过了设置的最长缓存时间（默认是一小时），如果是则清空缓存--即每隔一段时间清
  空一次缓存。这个有点像 redis 设置的超时时间。
* TransactionalCache ：事务缓存。

**2.3 一级缓存**

一级缓存也叫本地缓存，MyBatis 的一级缓存是在会话（SqlSession）层面进行缓存的。MyBatis 的一级缓存是默认开启的，不需要任何的配置。

上面说到缓存的默认实现对象是 PerpetualCache ，那么这个对象是在哪里维护的呢？MyBatis 的一级缓存是在会话共享，那么我们先看下 SqlSession 这个里面有没有维护。SqlSession  是接口，阿粉上一篇源码讲了创建 SqlSession 最后返回的是 DefaultSqlSession 对象。我们看下这个类的成员变量：

```java
public class DefaultSqlSession implements SqlSession {

  private final Configuration configuration;
  private final Executor executor;

  private final boolean autoCommit;
  private boolean dirty;
  private List<Cursor<?>> cursorList;
    。。。
}
```

首先，Configuration 对象是全局的，不可能放在这个里面。后面3个也不像，我们看下 Executor 类。

Executor  类也是一个接口，我们看下它的抽象实现 BaseExecutor ：

```java
public abstract class BaseExecutor implements Executor {

  private static final Log log = LogFactory.getLog(BaseExecutor.class);

  protected Transaction transaction;
  protected Executor wrapper;

  protected ConcurrentLinkedQueue<DeferredLoad> deferredLoads;
  protected PerpetualCache localCache;
  protected PerpetualCache localOutputParameterCache;
  protected Configuration configuration;
    。。。
}
```

看到没，缓存对象是在这个对象里面维护的。Executor 这个类就是执行 sql 的，可以理解为 sql 执行器。

然后再来看下 PerpetualCache 类是怎么实现缓存的。

```java
public class PerpetualCache implements Cache {

  private final String id;

  private Map<Object, Object> cache = new HashMap<>();
    。。。
}
```

很明显，是用 HashMap 实现的。既然是 HashMap ，那么用什么作为 key 呢？我们看下 BaseExecutor 里面有一个 createCacheKey() 的方法：

```java
public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    CacheKey cacheKey = new CacheKey();
    cacheKey.update(ms.getId());
    cacheKey.update(rowBounds.getOffset());
    cacheKey.update(rowBounds.getLimit());
    cacheKey.update(boundSql.getSql());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();
    // mimic DefaultParameterHandler logic
    for (ParameterMapping parameterMapping : parameterMappings) {
      if (parameterMapping.getMode() != ParameterMode.OUT) {
        Object value;
        String propertyName = parameterMapping.getProperty();
        if (boundSql.hasAdditionalParameter(propertyName)) {
          value = boundSql.getAdditionalParameter(propertyName);
        } else if (parameterObject == null) {
          value = null;
        } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
          value = parameterObject;
        } else {
          MetaObject metaObject = configuration.newMetaObject(parameterObject);
          value = metaObject.getValue(propertyName);
        }
        cacheKey.update(value);
      }
    }
    if (configuration.getEnvironment() != null) {
      cacheKey.update(configuration.getEnvironment().getId());
    }
    return cacheKey;
  }
```

然后我们来看下：

* ms.getId() ：ms 是解析 mapper.xml 创建的对象，每个 select/update/delete/insert 标签会创建一个 MappedStatement 对象。id 就是 mapper 的 namespace 加上 4种标签的 id。
* rowBounds.getOffset() ：分页参数。
* rowBounds.getLimit() ： 分页参数。
* boundSql.getSql() ： sql 语句。
* value ：这个是解析的 sql 传入的参数。
* configuration.getEnvironment().getId() ： 这个是配置的数据源 id ，spring 里面，数据源不会在 mybatis里面配置。

这就是缓存 key 的组成。

最后阿粉还是用例子来验证一下一级缓存是否是在 session 中共享的。判断是否命中缓存，可以根据是否打印 sql 来判断，没有缓存就会去数据库查询，所有会打印 sql，否则就是有缓存，不会打印sql。来看下例子：

```java
 @Test
    public void testSelect() throws IOException {

        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = sqlSessionFactory.openSession();
        try {
            FruitMapper mapper = session.getMapper(FruitMapper.class);
            Fruit fruit = mapper.findById(1L);
            System.out.println("第一次查询:"+ fruit);
            Fruit fruit1 = mapper.findById(1L);
            System.out.println("第二次查询:" + fruit1);
        } finally {
            session.close();
        }
    }
```

结果为:

```java
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@42f48531]
==>  Preparing: select id,name from fruit where id =? 
==> Parameters: 1(Long)
<==    Columns: id, name
<==        Row: 1, 苹果
<==      Total: 1
第一次查询:Fruit(id=1, name=苹果)
第二次查询:Fruit(id=1, name=苹果)
```

说明同一个 session 里面，缓存是共享的。接下来我们在不同的 session 中看下：

```java
public class MybatisTest {

    @Test
    public void testSelect() throws IOException {

        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = sqlSessionFactory.openSession();
        SqlSession session2 = sqlSessionFactory.openSession();
        try {
            FruitMapper mapper = session.getMapper(FruitMapper.class);
            FruitMapper mapper2 = session2.getMapper(FruitMapper.class);
            Fruit fruit = mapper.findById(1L);
            System.out.println("第一次查询:"+ fruit);
            Fruit fruit1 = mapper2.findById(1L);
            System.out.println("第二次查询:" + fruit1);
        } finally {
            session.close();
        }

    }
}
```

结果为:

```java
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@42f48531]
==>  Preparing: select id,name from fruit where id =? 
==> Parameters: 1(Long)
<==    Columns: id, name
<==        Row: 1, 苹果
<==      Total: 1
第一次查询:Fruit(id=1, name=苹果)
Opening JDBC Connection
Created connection 1358857082.
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@50fe837a]
==>  Preparing: select id,name from fruit where id =? 
==> Parameters: 1(Long)
<==    Columns: id, name
<==        Row: 1, 苹果
<==      Total: 1
第二次查询:Fruit(id=1, name=苹果)
```

说明不同的 session ，缓存是不共享的。

### 3.总结

今天的 mybatis 一级缓存到这里就结束了。喜欢阿粉的同学记得点个赞哦。我们下次再见。

