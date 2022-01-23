---
layout: post
categories: Java
title: mybatis系列之初识mybatis
tagline: by 小九
tags: 
  - 小九
---

> hello~各位读者新年好，我是鸭血粉丝（大家可以称呼我为「阿粉」），一位立志要成为码王的男人！

## 说在前面的话

今天开始，阿粉准备把 `mybatis` 的知识梳理一遍，为什么梳理 `mybatis` 呢？因为 `mybatis` 的源码最简单啊（说什么大实话）。no~no~no~，是因为现在的这些框架都和 `springboot` 整合在一起了，用起来是方便了，但是其中的原理就越不了解了。所以阿粉整理几篇 `mybatis` 的文章分享给大家，配合代码案例，希望大家有所收获。另外因为这是第一篇，所以代码量相对来说比较多，希望大家耐下看下去，毕竟阿粉也是下了一番功夫的，绝对是原创，如有雷同，肯定是他们抄袭阿粉的。

好了，话不多说，我们直接进入正题。

### 1 mybatis为我们做了哪些事情

1. 使用连接池对连接进行管理 ：连接池，这个不多说。
2. SQL 和代码分离，集中管理 ：主要是 `mapper.xml` 文件，专门用来配置sql语句。
3. 重复 SQL 的提取 ：比如<sql>标签。
4. 参数映射和动态 SQL ：比如<if> <where>等。
5. 结果集映射 :查询结果后会映射成对象。
6. 缓存管理 :一级缓存和二级缓存。
7. 插件机制：分页插件等。

这些阿粉会在 mybatis 系列文章中都会讲到。

### 2 准备工作（小demo，配合讲解）

先整体说下需要有哪些东西：

* `pom.xml` 引入 `mybatis` 引入 `jar` 包。
* `mybatis-config.xml` 配置文件。
* `mapper` 接口和 `mapper.xml` 文件。
* 测试类。

**2.1 `pom.xml` 文件**  ,因为不和 `spring/springBoot` 集成，所以引入的 `jar` 包比较少。

```xml
	<dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.1</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.16</version>
    </dependency>
```

**2.2 `mybatis-config.xml`文件(重点)** ，`db.properties` 为数据库的配置信息，不单独贴出来。

```xml
<configuration>
    <properties resource="db.properties"></properties>
    <settings>
        <!-- 打印查询语句 -->
        <setting name="logImpl" value="STDOUT_LOGGING" />
    </settings>
    <!-- 给实体类取别名 在mapper.xml里面就可以这样使用:resultType="fruit"-->
    <typeAliases>
        <typeAlias alias="fruit" type="com.cl.mybatis.pojo.Fruit" />
    </typeAliases>
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
    <!-- 配置mapper.xml文件 -->
    <mappers>
        <mapper resource="FruitMapper.xml"/>
    </mappers>
</configuration>
```

**2.3 mapper接口和 `mapper.xml`**，Fruit为水果 pojo 类，只有id和name两个属性，不单独贴出来

```java
public interface FruitMapper {
    Fruit findById(Long id);
}
```

```xml
<mapper namespace="com.cl.mybatis.mapper.FruitMapper">

    <sql id="fruit">id,name </sql>

    <select id="findById" resultType="fruit">
        select <include refid="fruit" /> from fruit where id =#{id}
    </select>
</mapper>
```

**2.4  测试方法(重点)**

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

**2.5 执行测试方法的结果**

```java
Created connection 2014838114.
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@7817fd62]
==>  Preparing: select id,name from fruit where id =? 
==> Parameters: 1(Long)
<==    Columns: id, name
<==        Row: 1, 苹果
<==      Total: 1
Fruit(id=1, name=苹果)
```

简单的 `demo` 就做好了，大家有兴趣的话可以跟着做一下。下面才是本次的重点类容。

### 3 配置文件标签作用

这里只说下`mybatis-config.xml`里面现在用到的标签，后面用到了其他标签时会说明。然后在最后一篇 `mybatias` 文章时，会把所有的标签都整理出来，并且重要的会有例子说明，还会有自定义标签里面的类容的例子。

* configuration：这个是配置的最顶层标签
* properties：用来配置数据库连接的信息
* settings：全局配置标签，这里阿粉只配置了打印sql日志
* typeAliases ：类型别名，用来设置一些别名来代替 Java 的长类型声明（如 java.lang.int 变为 int），减少配置编码的冗余
* mappers：映射器配置，配置mapper.xml文件
* environments：环境配置，主要用来配置数据源和事物

### 4 核心对象

最后讲下测试类，里面的代码就是单独使用 MyBatis 的全部流程。后面讲源码也是基于这个来讲。首先说下里面的4个核心对象：

* `SqlSessionFactory` 全局唯一的，一旦被创建之后，在应用的运行期间一直存在。
* `SqlSessionFactoryBuilder`  这个就是用来构建 `SqlSessionFactory` ，一旦构建好后面就用不到。实际上就是解析 `mapper.xml` 和 `mybatis-config.xml` 里面的标签的属性值设置到 `Configuration`这个类对应的属性上。
* `SqlSession` ：是一次会话，它不是线程安全的，不能在线程间共享。所以我们在请求开始的时候创建一个 SqlSession 对象，在请求结束或者说方法执行完毕的时候要及时关闭它。
* `Mapper` ：是从 SqlSession 中获取的（实际上是一个代理对象）。 它的作用是发送 SQL 来操作数据库。

然后阿粉用个类比的方法，模拟一个场景来帮助大家理解一下这几个核心对象的作用。场景是这样的：比如说一个建筑公司（`SqlSessionFactory`）需要修一个房子，首先是不是有设计图纸（`mapper.xml`）和修房子需要用到的材料清单（ `mybatis-config.xml`），然后采购部门（`SqlSessionFactoryBuilder`）根据材料清单去采购材料放到仓库（`Configuration`）中。然后是工人师傅（`SqlSession`）根据设计图纸（`mapper.xml`）和仓库里面的材料（ `mybatis-config.xml`）去造房子（数据库）。

这样是不是要好理解一点。啊，阿粉都被自己的聪明给折服了（自恋中）。

### 5 主要的执行流程

![](http://www.justdojava.com\assets\images\2019\java\image-xiaojiu\20200110\2.jpg)

主要流程就这些，里面的细节后面会讲。这幅图的流程就不做讲解了，可以看下上面的类比例子。

### 6 总结

回顾一下这节的类容：

1. 一个简单的demo。
2. `mybatis-config.xml`里面一级标签的作用说明。
3. 几个核心对象的作用。
4. 一个图片说明了一下执行的流程。

这期 `mybatis` 的类容到这里就结束了,觉得阿粉讲的还可以的,记得点个赞哦！我们下期再见。