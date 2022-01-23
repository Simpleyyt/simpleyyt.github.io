---
layout: post
title: Shiro--从一个简单的 Realm 开始权限认证
tagline: by 江南一点雨
categories: shiro
tags: 
  - 江南一点雨
---

通过上篇文章的学习，小伙伴们对 shiro 应该有了一个大致的了解了，本文我们就来通过一个简单的案例，先来看看 shiro 中登录操作的一个基本用法。  

<!--more-->

## shiro下载

要学习 shiro，我们首先需求去 shiro 官网下载 shiro，官网地址地址 https://shiro.apache.org/，截至本文写作时，shiro 的最新稳定版本为 1.4.0 ，本文将采用这个版本。当然，shiro 我们也可以从 github 上下载到源码。两个源码下载地址如下：  

1. [apache shiro](http://www.apache.org/dyn/closer.cgi/shiro/1.3.2/shiro-root-1.3.2-source-release.zip)  
2. [github-shiro](https://github.com/apache/shiro/archive/master.zip)  

上面我主要是和小伙伴们介绍下源码的下载，并没有涉及到 jar 包的下载，jar 包我们到时候直接使用 maven 即可。  

## 创建演示工程

这里我们先不急着写代码，我们先打开刚刚下载到的源码，源码中有一个samples目录，如下：  

![](http://www.justdojava.com/assets/images/2019/java/image_javaboy/shiro/2-1.jpg)

这个 samples 目录是官方给我们的一些演示案例，其中有一个 quickstart 项目，这个项目是一个 maven 项目，参考这个 quickstart ，我们来创建一个自己的演示工程。  

- 首先使用 maven 创建一个 JavaSE 工程  
工程创建成功后在pom文件中添加如下依赖：  


```xml
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-all</artifactId>
	<version>RELEASE</version>
</dependency>
```

- 配置用户  

参考 quickstart 项目中的 shiro.ini 文件，我们来配置一个用户，配置方式如下：首先在 resources 目录下创建一个 shiro.ini 文件，文件内容如下：  

```
[users]
sang=123,admin
[roles]
admin=*
```

以上配置表示我们创建了一个名为 sang 的用户，该用户的密码是 123 ，该用户的角色是 admin ，而 admin 具有操作所有资源的权限。  

- 执行登录  

OK，做完上面几步之后，我们就可以来看看如何实现一次简单的登录操作了。这个登录操作我们依然是参考 quickstart 项目中的类来实现，首先我们要通过 shiro.ini 创建一个 SecurityManager ，再将这个 SecurityManager 设置为单例模式，如下：  

```
Factory<org.apache.shiro.mgt.SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
org.apache.shiro.mgt.SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
```

如此之后，我们就配置好了一个基本的 Shiro 环境，注意此时的用户和角色信息我们配置在 shiro.ini 这个配置文件中，接下来我们就可以获取一个 Subject 了，这个 Subject 就是我们当前的用户对象，获取方式如下：  

```java
Subject currentUser = SecurityUtils.getSubject();
```

拿到这个用户对象之后，接下来我们可以获取一个 session 了，这个 session 和我们 web 中的 HttpSession 的操作基本上是一致的，不同的是，这个 session 不依赖任何容器，可以随时随地获取，获取和操作方式如下：  

```java
//获取session
Session session = currentUser.getSession();
//给session设置属性值
session.setAttribute("someKey", "aValue");
//获取session中的属性值
String value = (String) session.getAttribute("someKey");
```

说了这么多，我们的用户到现在还没有登录呢，Subject 中有一个 isAuthenticated 方法用来判断当前用户是否已经登录，如果 isAuthenticated 方法返回一个 false，则表示当前用户未登录，那我们就可以执行登陆，登录方式如下：  

```java
if (!currentUser.isAuthenticated()) {
    UsernamePasswordToken token = new UsernamePasswordToken("sang", "123");
    try {
        currentUser.login(token);
    } catch (UnknownAccountException uae) {
        log.info("There is no user with username of " + token.getPrincipal());
    } catch (IncorrectCredentialsException ice) {
        log.info("Password for account " + token.getPrincipal() + " was incorrect!");
    } catch (LockedAccountException lae) {
        log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                "Please contact your administrator to unlock it.");
    }
    catch (AuthenticationException ae) {
    }
}
```

首先构造 UsernamePasswordToken ，两个参数就是我们的用户名和密码，然后调用 Subject 中的 login 方法执行登录，当用户名输错，密码输错、或者账户锁定等问题出现时，系统会通过抛异常告知调用者这些问题。  

当登录成功之后，我们可以通过如下方式获取当前登陆用户的用户名：  

```
log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");
```

我们也可以通过调用 Subject 中的 hasRole 和 isPermitted 方法来判断当前用户是否具备某种角色或者某种权限，如下：  

```java
if (currentUser.hasRole("admin")) {
    log.info("May the Schwartz be with you!");
} else {
    log.info("Hello, mere mortal.");
}
if (currentUser.isPermitted("lightsaber:wield")) {
    log.info("You may use a lightsaber ring.  Use it wisely.");
} else {
    log.info("Sorry, lightsaber rings are for schwartz masters only.");
}
```

最后，我们可以通过 logout 方法注销本次登录，如下：  

```
currentUser.logout();
```  

OK，至此，我们通过官方案例给小伙伴们简单介绍了 Shiro 中的登录操作，完整案例大家可以参考官方的 demo。

参考资料：  

1. 维基百科-Apache Shiro  
2. Shiro 参考手册 