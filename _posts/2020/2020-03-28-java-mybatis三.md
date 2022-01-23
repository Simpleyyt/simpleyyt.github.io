---
layout: post
categories: Java
title: mybatis系列之mybatis源码讲解
tagline: by 小九
tags: 
  - 小九
---

> hello~各位读者好，我是鸭血粉丝（大家可以称呼我为「阿粉」）。今天，我们继续 `mybatis` 源码讲解的第三篇文章。

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

上篇源码分析讲了 `SqlSessionFactory` 构建的过程。这次，我们来了解下 `SqlSession` 。

### 2.SqlSession

**2.1 什么是 SqlSession**

`SqlSession` 是我们自己的程序与数据库的一次会话。它是线程不安全的，所以我们在请求开始的时候创建一个SqlSession 对象，在请求结束或者说方法执行完毕的时候要及时关闭它。

阿粉在这里举例简单的例子说明一下`一次会话`。比如我们打电话，电话接通，通了之后一般情况下都是你说完一件事，对方在回答，然后你在说，对方在回答，直到你事情说完了挂断电话，这个过程就相当于一次会话。在我们程序中也是一样的道理，先和数据库建立一个连接，然后发送sql，数据库执行完成之后会返回结果，然后又可以发送sql，直到方法执行完毕，关闭连接，这就是一次会话。

然后阿粉在补充一点，`mysql` 的通信方式是半双工,被动的执行请求。在一次会话中，必须等 `mysql` 返回结果才能执行下一个sql操作，相当于同步。

**2.2 创建`SqlSession` 的过程**

我们来看上面代码的这段 `SqlSession session = sqlSessionFactory.openSession();` ，然后上次文章中阿粉带着大家了解了，`build`方法返回的是 `DefaultSqlSessionFactory` 对象。

```java
public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }
```

所以，我们在 `DefaultSqlSessionFactory` 类中找到 `openSession` 方法。

```java
  @Override
  public SqlSession openSession() {
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
  }
```

然后 `openSessionFromDataSource` 方法：

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      final Executor executor = configuration.newExecutor(tx, execType);
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

我们一句一句的来看下，首先是 `final Environment environment = configuration.getEnvironment();` `Environment ` 这个还有印象吗？阿粉第一次讲 `mybatis.xml` 一级标签的时候有说哦，忘了的话阿粉在带着大家回忆一下哦。

```xml
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/><!-- 单独使用时配置成MANAGED没有事务 -->
        <dataSource type="POOLED">
            <property name="driver" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </dataSource>
    </environment>
</environments>
```

这个对象里面就是存放的事物的配置和数据源的配置信息。然后后面两句就清楚了呗，先创建一个事物工厂，事物工厂通过数据源信息创建事物对象，这里又用到了工厂模式。这个就不详细讲了，因为和Spring整合后，这些事情就不是 `mybatis` 做了。

我们看下面一句：Executor executor = configuration.newExecutor(tx, execType)，这句是创建`Executor`实例。先来了解一下`Executor`：

`Executor` 是个接口，执行器的抽象对象，有3种基本类型：SIMPLE、BATCH、REUSE，默认是SIMPLE。然后看下这三个的区别：

1. SimpleExecutor：每执行一次update 或select，就开启一个Statement 对象，用完立刻关闭Statement 对象。
2. ReuseExecutor：执行update 或select，以sql 作为key 查找Statement 对象，存在就使用，不存在就创建，用完后，不关闭Statement 对象，而是放置于Map 内，供下一次使用。简言之，就是重复使用Statement 对象。
3. BatchExecutor：执行update时将所有sql 都添加到批处理中（addBatch()），等待统一执行。它缓存了多个Statement 对象，每个Statement 对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC 批处理相同。

然后继续源码，看下`newExecutor`方法

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
  }
```

前面的`if` 判断就是根据传入的类型来创建`Executor` ，看下后面的判断：是否开启缓存，这里使用了装饰器模式，这个类（CachingExecutor）就是二级缓存的实现。

然后执行器（`Executor`）创建后，就会创建一个`DefaultSqlSession`对象。

### 4.总结

这次主要是分享创建 `SqlSession`的源码流程，不知道阿粉有没有讲清楚，因为源码比较多，阿粉都调整文章好几次。有兴趣的同学可以跟着阿粉的思路走一遍源码，肯定还是有收获的。

