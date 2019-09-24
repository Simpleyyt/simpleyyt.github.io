---
layout: post
category: Java
title: 再谈 Shiro 中的 Realm
tagline: by 江南一点雨
categories: Java
tags: 
  - Shiro
---

上篇文章和小伙伴们聊了 realm 的简单操作，本文我们来继续深入这个话题。  


## Realm的继承关系

通过查看类的继承关系，我们发现 Realm 的子类实际上有很多种，这里我们就来看看有代表性的几种：  

1.IniRealm   

可能我们并不知道，实际上这个类在我们第二篇文章中就已经用过了。这个类一开始就有如下两行定义：  

```
public static final String USERS_SECTION_NAME = "users";
public static final String ROLES_SECTION_NAME = "roles";
```

这两行配置表示 shiro.ini 文件中，[users] 下面的表示表用户名密码还有角色，[roles] 下面的则是角色和权限的对应关系。  

2.PropertiesRealm  

PropertiesRealm 则规定了另外一种用户、角色定义方式，如下：  

user.user1=password,role1
role.role1=permission1  

3.JdbcRealm  

这个顾名思义，就是从数据库中查询用户的角色、权限等信息。打开 JdbcRealm 类，我们看到源码中有如下几行：  

```
protected static final String DEFAULT_AUTHENTICATION_QUERY = "select password from users where username = ?";
protected static final String DEFAULT_SALTED_AUTHENTICATION_QUERY = "select password, password_salt from users where username = ?";
protected static final String DEFAULT_USER_ROLES_QUERY = "select role_name from user_roles where username = ?";
protected static final String DEFAULT_PERMISSIONS_QUERY = "select permission from roles_permissions where role_name = ?";
```

根据这几行预设的 SQL 我们就可以大致推断出数据库中表的名称以及字段了，当然，我们也可以自定义 SQL。JdbcRealm 实际上是 AuthenticatingRealm 的子类，关于 AuthenticatingRealm 我们在后面还会详细说到，这里先不展开。接下来我们就来详细说说这个 JdbcRealm。  

## JdbcRealm

1.准备工作  

使用 JdbcRealm，涉及到数据库操作，要用到数据库连接池，这里我使用 Druid 数据库连接池，因此首先添加如下依赖：  

```
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>RELEASE</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.27</version>
</dependency>
```

2.数据库创建  

想要使用 JdbcRealm，那我首先要创建数据库，根据 JdbcRealm 中预设的 SQL，我定义的数据库表结构如下：  

![p309](http://www.justdojava.com/assets/images/2019/java/image_javaboy/shiro/4-1.jpg)  

这里为了大家能够直观的看到表的关系，我使用了外键，实际工作中，视情况而定。然后向表中添加几条测试数据。数据库脚本小伙伴可以在 github 上下载到（https://github.com/lenve/shiroSamples/blob/v4/shiroDemo.sql）。  

3.配置文件处理  

然后将 shiro.ini 中的所有配置注释掉，添加如下注释：  

```
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql://localhost:3306/shiroDemo
dataSource.username=root
dataSource.password=123
jdbcRealm.dataSource=$dataSource
jdbcRealm.permissionsLookupEnabled=true
securityManager.realms=$jdbcRealm
```

这里的配置文件都很简单，不做过多赘述，小伙伴唯一需要注意的是 permissionsLookupEnabled 需要设置为 true，否则一会 JdbcRealm 就不会去查询权限用户权限。  

4.测试  

OK，做完上面几步就可以测试了，测试方式和第二篇文章中一样，我们可以测试下用户登录，用户角色和用户权限。  

5.自定义查询 SQL

小伙伴们看懂了上文，对于自定义查询 SQL 就没什么问题了。我这里举一个简单的例子，比如我要自定义 authenticationQuery 对对应的 SQL，查看 JdbcRealm 源码，我们发现 authenticationQuery 对应的 SQL 本来是 `select password from users where username = ?` ，如果需要修改的话，比如说我的表名不是 users 而是 employee，那么在 shiro.ini 中添加如下配置即可：  

```
jdbcRealm.authenticationQuery=select password from employee where username = ?
```

OK,这个小伙伴下来自己做尝试，我这里就不演示了。  

好了，JdbcRealm 我们就先说到这里，有问题欢迎留言讨论。  

本文案例下载地址：https://github.com/lenve/shiroSamples/archive/v4.zip  