---
layout: post
title: Shiro ，绕不过的 Realm
tagline: by 江南一点雨
categories: Java
tags: 
  - Shiro
---

上篇文章和小伙伴们仔细聊了聊 Shiro 中的登录操作，官方的 demo 中对登录操作也有很多英文注释，相信大家都能理解，但是对于整个登录的过程，可能小伙伴们还有一些疑惑，那么本篇文章我将和小伙伴们分享一下 Shiro 中的登录流程，顺便介绍一个重要的类--Realm。  


## 登录流程是什么样的

首先我们来看 shiro 官方文档中这样一张登录流程图：  

![](http://www.justdojava.com/assets/images/2019/java/image_javaboy/shiro/3-1.jpg) 

参照此图，我们的登录一共要经过如下几个步骤：  

1.  应用程序代码调用 Subject.login 方法，传递创建好的包含终端用户的 Principals(身份)和 Credentials(凭证)的 AuthenticationToken 实例(即上文例子中的 UsernamePasswordToken)。  
2.  Subject 实例，通常是 DelegatingSubject（或子类）委托应用程序的 SecurityManager 通过调用 securityManager.login(token) 开始真正的验证工作(在 DelegatingSubject 类的 login 方法中打断点即可看到)。  
3.  SubjectManager 作为一个基本的“保护伞”的组成部分，接收 token 以及简单地委托给内部的 Authenticator 实例通过调用 authenticator.authenticate(token)。这通常是一个 ModularRealmAuthenticator 实例，支持在身份验证中协调一个或多个 Realm 实例。ModularRealmAuthenticator 本质上为 Apache Shiro 提供了 PAM-style 范式（其中在 PAM 术语中每个 Realm 都是一个'module'）。  
4. 如果应用程序中配置了一个以上的 Realm，ModularRealmAuthenticator 实例将利用配置好的 AuthenticationStrategy 来启动 Multi-Realm 认证尝试。在 Realms 被身份验证调用之前，期间和以后，AuthenticationStrategy 被调用使其能够对每个 Realm 的结果作出反应。如果只有一个单一的 Realm  被配置，它将被直接调用，因为没有必要为一个单一 Realm 的应用使用 AuthenticationStrategy。  
5.  每个配置的 Realm 用来帮助看它是否支持提交的 AuthenticationToken。如果支持，那么支持 Realm 的 getAuthenticationInfo 方法将会伴随着提交的 token 被调用。  

OK，通过上面的介绍，相信小伙伴们对整个登录流程都有一定的理解了，小伙伴可以通过打断点来验证我们上文所说的五个步骤。那么在上面的五个步骤中，小伙伴们看到了有一个 Realm 承担了很重要的一部分工作，那么这个 Realm 到底是个什么东西，接下来我们就来仔细看一看。  

## 什么是 Realm

根据 Realm 文档上的解释，Realms 担当 Shiro 和你的应用程序的安全数据之间的“桥梁”或“连接器”。当它实际上与安全相关的数据如用来执行身份验证（登录）及授权（访问控制）的用户帐户交互时，Shiro 从一个或多个为应用程序配置的 Realm 中寻找许多这样的东西。在这个意义上说，Realm  本质上是一个特定安全的 DAO：它封装了数据源的连接详细信息，使 Shiro 所需的相关的数据可用。当配置 Shiro 时，你必须指定至少一个 Realm 用来进行身份验证和/或授权。SecurityManager 可能配置多个 Realms，但至少有一个是必须的。Shiro 提供了立即可用的 Realms 来连接一些安全数据源（即目录），如 LDAP，关系数据库（JDBC），文本配置源，像 INI 及属性文件，以及更多。你可以插入你自己的 Realm 实现来代表自定义的数据源，如果默认地 Realm 不符合你的需求。  

看了上面这一段解释，可能还有小伙伴云里雾里，那么接下来我们来通过一个简单的案例来看看 Realm 到底扮演了一个什么样的作用，注意，本文的案例在上文案例的基础上完成。首先自定义一个 MyRealm，内容如下：  

```java
public class MyRealm implements Realm {
    public String getName() {
        return "MyRealm";
    }
    public boolean supports(AuthenticationToken token) {
        return token instanceof UsernamePasswordToken;
    }
    public AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String password = new String(((char[]) token.getCredentials()));
        String username = token.getPrincipal().toString();
        if (!"sang".equals(username)) {
            throw new UnknownAccountException("用户不存在");
        }
        if (!"123".equals(password)) {
            throw new IncorrectCredentialsException("密码不正确");
        }
        return new SimpleAuthenticationInfo(username, password, getName());
    }
}
```

自定义 Realm 实现 Realm 接口，该接口中有三个方法，第一个 getName 方法用来获取当前 Realm 的名字，第二个 supports 方法用来判断这个 realm 所支持的 token，这里我假设值只支持 UsernamePasswordToken 类型的 token，第三个 getAuthenticationInfo 方法则进行了登陆逻辑判断，从 token 中取出用户的用户名密码等，进行判断，当然，我这里省略掉了数据库操作，当登录验证出现问题时，抛异常即可，这里抛出的异常，将在执行登录那里捕获到（注意，由于我这里定义的 MyRealm 是实现了 Realm 接口，所以这里的用户名和密码都需要我手动判断是否正确，后面的文章我会介绍其他写法）。  

OK，创建好了 MyRealm 之后还不够，我们还需要做一个简单配置，让 MyRealm 生效，将 shiro.ini 文件中的所有东西都注释掉，添加如下两行：  

```
MyRealm= org.sang.MyRealm
securityManager.realms=$MyRealm
```  

第一行表示定义了一个 realm ，第二行将这个定义好的交给 securityManger，这里实际上会调用到 RealmSecurityManager 类的 setRealms 方法。OK，做好这些之后，小伙伴们可以在 MyRealm 类中的一些关键节点打上断点，再次执行 main 方法，看看整个的登录流程。  

好了，本文我们介绍了一个简单的 realm 操作，更多 realm 操作，我们下篇文章继续来谈。  

本文案例下载地址：https://github.com/lenve/shiroSamples/archive/v3.zip  

参考资料：  

1. 维基百科-Apache Shiro  
2. Shiro 参考手册 