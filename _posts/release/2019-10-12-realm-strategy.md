---
layout: post
categories: shiro
title: 【Shiro 系列 05】Shiro 中多 Realm 的认证策略问题
tagline: by 江南一点雨
tags:
  - 江南一点雨
---

上篇文章和小伙伴们分享了 JdbcRealm，本文我想和小伙伴们聊聊多 Realm 的认证策略问题。

<!--more-->

---


## 多 Realm 认证策略

不知道小伙伴们是否还记得这张登录流程图：

![p308](http://www.justdojava.com/assets/images/2019/java/image_javaboy/shiro/5-1.jpg)

从这张图中我们可以清晰看到 Realm 是可以有多个的，不过到目前为止，我们所有的案例都还是单 Realm，那么我们先来看一个简单的多 Realm 情况。

注意，本文的案例我们也是在前面案例的基础上完成，因此强烈建议小伙伴们阅读前文。

前面的文章我们自己创建了一个 MyRealm，也用过 JdbcRealm，但都是单独使用的，现在我想将两个一起使用，只需要修改 shiro.ini 配置即可，如下：

```java
MyRealm= org.sang.MyRealm

jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql://localhost:3306/shiroDemo
dataSource.username=root
dataSource.password=123
jdbcRealm.dataSource=$dataSource
jdbcRealm.permissionsLookupEnabled=true
securityManager.realms=$jdbcRealm,$MyRealm
```

但是此时我数据库中用户的信息是 sang/123,MyRealm 中配置的信息也是 sang/123，我把 MyRealm 中的用户信息修改为 `江南一点雨/456` ，此时，我的 MyRealm 的 getAuthenticationInfo 方法如下：

```java
public AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    String password = new String(((char[]) token.getCredentials()));
    String username = token.getPrincipal().toString();
    if (!"江南一点雨".equals(username)) {
        throw new UnknownAccountException("用户不存在");
    }
    if (!"456".equals(password)) {
        throw new IncorrectCredentialsException("密码不正确");
    }
    return new SimpleAuthenticationInfo(username, password, getName());
}
```

这个时候我们就配置了两个 Realm，还是使用我们一开始的测试代码进行登录测试，这个时候我们发现我既可以使用 `江南一点雨/456` 进行登录，也可以使用 `sang/123` 进行登录，用 `sang/123` 登录成功之后用户的角色信息和之前是一样的，而用 `江南一点雨/456` 登录成功之后用户没有角色，这个也很好理解，因为我们在 MyRealm 中没有给用户配置任何权限。总而言之，就是当我有了两个 Realm 之后，现在只需要这两个 Realm 中的任意一个认证成功，就算我当前用户认证成功。

## 原理追踪

好了，有了上面的问题后，接下来我们在 Subject 的 login 方法上打断点，跟随程序的执行步骤，我们来到了 ModularRealmAuthenticator 类的 doMultiRealmAuthentication 方法中，如下：

```java
protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
    this.assertRealmsConfigured();
    Collection<Realm> realms = this.getRealms();
    return realms.size() == 1?this.doSingleRealmAuthentication((Realm)realms.iterator().next(), authenticationToken):this.doMultiRealmAuthentication(realms, authenticationToken);
}
```

在这个方法中，首先会获取当前一共有多少个 realm ，如果只有一个则执行 doSingleRealmAuthentication 方法进行处理，如果有多个 realm ，则执行 doMultiRealmAuthentication 方法进行处理。 doSingleRealmAuthentication 方法部分源码如下：

```java
protected AuthenticationInfo doSingleRealmAuthentication(Realm realm, AuthenticationToken token) {
    ...
    ...
    AuthenticationInfo info = realm.getAuthenticationInfo(token);
    if(info == null) {
        String msg = "Realm [" + realm + "] was unable to find account data for the submitted AuthenticationToken [" + token + "].";
        throw new UnknownAccountException(msg);
    } else {
        return info;
    }
}
```

小伙伴们看到这里就明白了，这里调用了 realm 的 getAuthenticationInfo 方法，这个方法实际上就是我们自己实现的 MyRealm 中的 getAuthenticationInfo 方法。

那如果有多个 Realm 呢？我们来看看 doMultiRealmAuthentication 方法的实现，部分源码如下：

```java
protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token) {
    AuthenticationStrategy strategy = this.getAuthenticationStrategy();
    AuthenticationInfo aggregate = strategy.beforeAllAttempts(realms, token);
    Iterator var5 = realms.iterator();
    while(var5.hasNext()) {
        Realm realm = (Realm)var5.next();
        aggregate = strategy.beforeAttempt(realm, token, aggregate);
        if(realm.supports(token)) {
            AuthenticationInfo info = null;
            Throwable t = null;
            try {
                info = realm.getAuthenticationInfo(token);
            } catch (Throwable var11) {
            }
            aggregate = strategy.afterAttempt(realm, token, info, aggregate, t);
        } else {
            log.debug("Realm [{}] does not support token {}.  Skipping realm.", realm, token);
        }
    }
    aggregate = strategy.afterAllAttempts(token, aggregate);
    return aggregate;
}
```

我这里主要来说下这个方法的实现思路：

1. 首先获取多 Realm 认证策略
2. 构建一个 AuthenticationInfo 用来存放一会认证成功之后返回的信息
3. 遍历 Realm，调用每个 Realm 中的 getAuthenticationInfo 方法，看是否能够认证成功
4. 每次获取到 AuthenticationInfo 之后，都调用 afterAttempt 方法进行结果合并
5. 遍历完所有的 Realm 之后，调用 afterAllAttempts 进行结果合并，这里主要判断下是否一个都没匹配上

## 自由配置认证策略

OK，经过上面的简单解析，小伙伴们对认证策略应该有一个大致的认识了，那么在 Shiro 中，一共支持三种不同的认证策略，如下：

1. AllSuccessfulStrategy，这个表示所有的Realm都认证成功才算认证成功
2. AtLeastOneSuccessfulStrategy，这个表示只要有一个 Realm 认证成功就算认证成功，默认即此策略
3. FirstSuccessfulStrategy，这个表示只要第一个 Realm 认证成功，就算认证成功

配置方式也很简单，在 shiro.ini 中进行配置，在上面配置的基础上，增加如下配置：

```ini
authenticator=org.apache.shiro.authc.pam.ModularRealmAuthenticator
securityManager.authenticator=$authenticator
allSuccessfulStrategy=org.apache.shiro.authc.pam.AllSuccessfulStrategy
securityManager.authenticator.authenticationStrategy=$allSuccessfulStrategy
```

此时，我们再进行登录测试，则会要求每个 Realm 都认证通过才算认证通过。

好了，Realm 的认证策略问题，我们就先说到这里，有问题欢迎留言讨论。

本文案例下载地址：https://github.com/lenve/shiroSamples/archive/v5.zip