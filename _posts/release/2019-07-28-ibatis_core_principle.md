---
layout: post
categories: ORM
title: ibatis  核心原理解析
tagline: by 小黑
tags: 
  - 小黑

---

最近查找一个生产问题的原因，需要深入研究 ibatis 框架的源码。虽然最后证明问题的原因与 ibatis 无关，但是这个过程加深了对 ibatis 框架原理的理解。

这篇文章主要就来讲讲 ibatis 框架的原理。

<!--more-->

> 可能现在很多人已不再使用 ibatis 或者说也没听 ibatis，不过肯定了解过 Mybatis。ibatis 就是 Mybatis框架的前身，虽然 ibatis 框架已经比较老，但是其核心功能与 Mybatis 一致。

## ibatis 解决的痛点

我们先看一个使用 JDBC 查询的例子。

![QueryDaoByJDBC1.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/QueryDaoByJDBC1-862ba548.png)

使用原生 JDBC 查询，存在两个痛点：

1. 使用非常繁琐，且需要处理各种数据库异常，并且还需要关闭各种资源。
2. 数据转化麻烦。查询之前需要从 Java 对象属性值设置到 `PreparedStatement`中，查询返回之后又需要从 `ResultSet`获取返回设置到返回对象中。

在 ibatis 中封装这些繁杂数据库连接查询代码，并处理了各类异常以及关闭各种资源。另外 ibatis 自动处理 Java 对象与数据库类型之间的自动转化，让业务代码与 SQL 代码之间做到了解耦。

## 数据类型转化原理

数据类型转化主要分为两类，一，传入查询的 Java 对象数据转化成 SQL 类型数据。二 查询返回的数据库信息映射到 Java 对象中。

ibatis SQL 需要定义在配置文件中，一个查询 SQL 语句配置如下：

```xml

      <select id="queryName" parameterClass="com.query.QueryDO"  resultClass="com.query.QueryDO" >
		select  * from  TEST_QUERY where ID=#id#

      </select>
```

ibatis 框架启动过程将会解析配置文件，生成  `MappedStatement` 的子类。如 select 配置会生成对应的 `SelectStatement` 对象。

`MappedStatement` 相关类图如下。

![MappedStatement.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/MappedStatement-89058ff2.png)

在 `MappedStatement` 中将会保存存在两个重要的对象，`ParameterMap`与 `ResultMap`，通过这两个对象将会完成 Java 类型与数据库类型的相互转化。


### Java 对象转化成数据库类型

以上面 select 配置为例，我们这里需要做的是从传入的 `com.query.QueryDO`对象中获取属性值，然后通过 `PreparedStatement.setxx` 设置到查询参数中。

ibatis 解析配置中 SQL 语句时，将会获取 # 之间的内容，将其替换成 `?`。然后按照顺序保存到一个 `ParameterMapping[]` 数组中，这个数组将会保存到 `ParameterMap` 对象中。

ParameterMapping 将会保存解析字段相关信息。

![ParameterMapping.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/ParameterMapping-2fb5e2eb.png)

最终解析后的 SQL 为：

```sql

select  * from  TEST_QUERY where ID=?

```

该 SQL 就可以通过 ` connection.prepareStatement("select * from  TEST_QUERY where ID=?");` 生成 `PreparedStatement` 对象。

接着 ibatis 会根据 `ParameterMapping `和 `parameterClass `指定的类型创建合适的 `dataExchange `和 `parameterPlan `对象。

其中 `parameterPlan` 对象会按照 `ParameterMapping `数组中顺序保存了变量的 setter 和 getter 方法数组。

`dataExchange `会按照 `ParameterMapping ` 数组中的顺序使用反射获取 parameterPlan getter 方法返回值生成 `parameters` 数组。

最后循环 ParameterMapping 数组，在 `TypeHandler` 调用  `PreparedStatement.setxx` 设置相关值。

![TypeHandler.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/TypeHandler-314cd88b.png)

`TypeHandler` 存在很多子类，通过这些子类正确处理了 Java 对象与数据库类型转化。

转化的时序图为：

![Java 对象与数据库对象转化](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/image006.png)

> 时序图来源于：https://www.ibm.com/developerworks/cn/java/j-lo-ibatis-principle/index.html

### 数据库字段映射到 Java 对象

SQL 执行结束之后将会返回查询结果，这里将会使 SQL 查询结果转化为返回结果 `com.query.QueryDO`。这里需要用到上面提到 `ResultMap ` 对象。

当 SQL 执行结束返回 `ResultSet` 对象之后，使用 `ResultSet.getMetaData()` 获取返回信息元数据对象 `ResultSetMetaData` 。

从 `ResultSetMetaData` 可以获取返回结果字段名，类型等信息，然后按照顺序存入 `ResultMapping` 数组中。

然后按照 `ResultMapping` 数组中使用 `TypeHandler`调用 `ResultSet.getxx` 获取实际返回数据，保存到 `columnValues` 数组中。

在 `ResultMap` 对象会根据  `ResultMapping` 与 `resultClass`指定的类型合适的 `dataExchange `和 `resultPlan`对象。`resultPlan`对象与上面的  `parameterPlan `对象一样也会保存着变量的 setter 和 getter 方法数组。

最后先根据 `resultClass` 反射生成返回对象，然后使用反射调用 `resultPlan` setter 方法，依次设置相关值。

映射返回对象时序图为：

![数据库字段映射到 Java 对象](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/image007.png)

> 时序图来源于：https://www.ibm.com/developerworks/cn/java/j-lo-ibatis-principle/index.html

## ibatis 样板代码

上面讲完了 ibatis 数据类型的转化原理，接着我们来看下 ibatis 调用 JDBC 样板代码。 

使用 ibatis 执行查询语句时，如 `queryForObject`，调用到 `SqlMapExecutorDelegate` 。在 `SqlMapExecutorDelegate` 中将会会做一些前提准备，比如准备事务，最后会将 SQL 语句委托给 `SqlExecutor` 执行。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/image-1f0f4bb5.png)

> 这里使用委托者模式，接受请求的对象将请求委托给另一个对象来处理。这种模式的优点在于解耦了业务代码与实际执行代码的联系，在于对外隐藏真正执行对象，易于扩展。 

在 `SqlExecutor#executeQuery`  执行过程主要分为以下三步。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/image-866b88e2.png)

第一步，获取 `PreparedStatement`，使用 `conn.prepareStatement(sql)` 获取。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190728/image-2bb65f4a.png)

第二步调用 `PreparedStatement.setxxx` 方法设置参数。上文中的 Java 对象类型转化成 SQL 类型在这里完成。

第三步，调用 `PreparedStatement.execute()` 执行 SQL 语句。

第四步，使用 `ResultSet` 获取返回值，在这一步将会完成 数据库类型与 Java 类型的转化。


## 帮助链接

[深入分析 iBATIS 框架之系统架构与映射原理](https://www.ibm.com/developerworks/cn/java/j-lo-ibatis-principle/index.html)