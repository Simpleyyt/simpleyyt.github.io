---
layout: post
title: 轻量级权限管理框架--Shiro 简介
tagline: by 江南一点雨
categories: Java
tags: 
  - Shiro
---

Apache Shiro是一个开源安全框架，提供身份验证、授权、密码学和会话管理。Shiro框架具有直观、易用等特性，同时也能提供健壮的安全性，虽然它的功能不如SpringSecurity那么强大，但是在普通的项目中也够用了。  


## 由来

Shiro 的前身是 JSecurity，2004年，Les Hazlewood 和 Jeremy Haile 创办了 Jsecurity。当时他们找不到适用于应用程序级别的合适 Java 安全框架，同时又对 JAAS 非常失望。2004 年到 2008 年期间，JSecurity 托管在 SourceForge 上，贡献者包括 Peter Ledbrook、Alan Ditzel 和 Tim Veil。2008年，JSecurity 项目贡献给了Apache软件基金会（ASF），并被接纳成为Apache Incubator 项目，由导师管理，目标是成为一个顶级 Apache 项目。期间，Jsecurity 曾短暂更名为 Ki，随后因商标问题被社区更名为 “Shiro” 。随后项目持续在 Apache Incubator 中孵化，并增加了贡献者 Kalle Korhonen。2010年7月，Shiro 社区发布了 1.0 版，随后社区创建了其项目管理委员会，并选举 Les Hazlewood 为主席。2010年9月22日，Shrio 成为 Apache 软件基金会的顶级项目（TLP）。  

## 有哪些功能

Apache Shiro 是一个强大而灵活的开源安全框架，它干净利落地处理身份认证，授权，企业会话管理和加密。Apache Shiro 的首要目标是易于使用和理解。安全有时候是很复杂的，甚至是痛苦的，但它没有必要这样。框架应该尽可能掩盖复杂的地方，露出一个干净而直观的 API，来简化开发人员在应用程序安全上所花费的时间。  

以下是你可以用 Apache Shiro 所做的事情：

1. 验证用户来核实他们的身份  
2. 对用户执行访问控制，如：判断用户是否被分配了一个确定的安全角色；判断用户是否被允许做某事  
3. 在任何环境下使用 Session API，即使没有 Web 容器  
4. 在身份验证，访问控制期间或在会话的生命周期，对事件作出反应  
5. 聚集一个或多个用户安全数据的数据源，并作为一个单一的复合用户“视图”  
6. 单点登录（SSO）功能
7. 为没有关联到登录的用户启用"Remember Me"服务  

。。。。等等

Apache Shiro 是一个拥有许多功能的综合性的程序安全框架。下面的图表展示了 Shiro 的重点：  

![](http://www.justdojava.com/assets/images/2019/java/image_javaboy/shiro/1-1.jpg) 

Shiro 中有四大基石——身份验证，授权，会话管理和加密。  

1. Authentication：有时也简称为“登录”，这是一个证明用户是谁的行为。  
2. Authorization：访问控制的过程，也就是决定“谁”去访问“什么”。  
3. Session Management：管理用户特定的会话，即使在非Web 或EJB 应用程序。  
4. Cryptography：通过使用加密算法保持数据安全同时易于使用。  

除此之外，Shiro 也提供了额外的功能来解决在不同环境下所面临的安全问题，尤其是以下这些：
  
1. Web Support：Shiro 的 web 支持的 API 能够轻松地帮助保护 Web 应用程序。  
2. Caching：缓存是 Apache Shiro 中的第一层公民，来确保安全操作快速而又高效。  
3. Concurrency：Apache Shiro 利用它的并发特性来支持多线程应用程序。  
4. Testing：测试支持的存在来帮助你编写单元测试和集成测试。  
5. "Run As"：一个允许用户假设为另一个用户身份（如果允许）的功能，有时候在管理脚本很有用。
6. "Remember Me"：在会话中记住用户的身份，这样用户只需要在强制登录时候登录。  

## 学习资料

Shiro 的学习资料并不多，没看到有相关的书籍。张开涛的《跟我学Shiro》是一个非常不错的资料，小伙伴可以搜索了解下。  

好了，上面就是我们对 shiro 的一个简单介绍，从下篇文章开始，我将带着小伙伴们一起来学习如何在项目中使用 shiro。  

参考资料：  

1. 维基百科-Apache Shiro  
2. Shiro 参考手册 