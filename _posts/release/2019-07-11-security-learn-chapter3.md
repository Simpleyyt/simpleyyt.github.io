---
layout: post
title:  跟我学spring security系列文章第三章 Remember Me功能
tagline: by 小马
category: java
tags: [java,spring,security,remember me]
---

* content
{:toc}


源码地址:

https://github.com/pony-maggie/spring-security-learn

remember me(记住我)这个功能大家都很熟悉，现在各大网站用得很普遍。当我们勾选了这个功能并且登录成功后，在一定的时间内再次登录就不用输入用户名和密码了，即使浏览器退出重新打开也是如此。

本文会实现一个基本的remember me功能，并且简单分析下它的实现原理。

<!--more-->

## 示例实现

首先，前端登录页面增加勾选按钮，这个简单，

```html
<div class="checkbox">
                <label><input type="checkbox" id="rememberme" name="remember-me"/>记住我</label>
            </div>
```

接着我们的配置也需要增加一些内容，

```java
.and()
                .rememberMe()
                .userDetailsService(myUserDetailsService)
                .tokenRepository(persistentTokenRepository()) //数据访问层,token持久化方案
                .tokenValiditySeconds(60)//token有效期，这里配置60秒
```

input标签中这个name很重要，这个属性必须有，并且名字一定要是remember-me，因为这是spring security默认识别的名字。当然这个名字可以自定义，只要在配置中加入remember-me-parameter选项指定自己喜欢的名字即可。


```java
                .rememberMe()
                .rememberMeParameter("my-remember-me")
```

userDetailsService用来指定获取用户信息的服务，因为token验证最终是要拿到用户信息的。

persistentTokenRepository是指明token的持久化方案。remember me功能是基于token，持久化方案有两种，一种基于内存，使用的是InMemoryTokenRepositoryImpl，一种基于数据库，使用的是JdbcTokenRepositoryImpl。这里我选择基于数据库的方式。

```java
//datasource的配置是通过配置文件加载过来的
    @Autowired
    DataSource dataSource;

    //注入BCryptPasswordEncoder，不然会报错
    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    PersistentTokenRepository persistentTokenRepository(){

        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        return jdbcTokenRepository;
    }
```

MyUserDetailsService跟第二章是一样的，功能就是从数据库获取用户信息。

我这里继续使用第二章的数据库配置。另外还需要在数据库里新建一张名为persistent_logins 的表，这是JdbcTokenRepositoryImpl缺省要使用的存放token的表，建表语句如下：

```sql
 create table persistent_logins (username varchar(64) not null default '', series varchar(64) primary key, token varchar(64) not null , last_used timestamp not null)
```

也可以在代码中自动建表，首次启动时使用:

```java
 jdbcTokenRepository.setCreateTableOnStartup(true);
```
然后再把这一行注释掉即可。

一般情况下，当用户主动点击注销时是不希望下次直接登录，所以我们需要在注销的时候删除cookie。

```java
.and().logout().permitAll()
                .logoutSuccessHandler(logoutSuccessHandler())
                .deleteCookies("remember-me")
```


启动工程，进行如下测试：

1. 访问首页，http://localhost:9090，需要授权，会自动跳入登录页：http://localhost:9090/login
2. 从登录页输入用户名和密码，然后勾选“记住我”，登录进入用户主页
3. 直接退出浏览器，然后输入首页网址，发现在60秒内可以不输入用户名和密码直接登录
4. 点击注销按钮，然后输入首页网址，需要重新登录



## 原理分析

如果在登录时勾选了Remember Me，登录成功后，会在浏览器中生成一个Cookie项，有效期就是我们通过tokenValiditySeconds指定的时间，其值是一个加密字符串。


![cookie](./images/cookie.jpg)


基本原理如下图所示:

![基本原理](https://img2018.cnblogs.com/blog/647585/201811/647585-20181109110908290-328952476.png)

当用户发起认证请求，会通过UsernamePasswordAuthenticationFilter进行认证，成功之后，可以调用SpringSecurity提供的RememberMeService生成一个Token并将它写入浏览器的Cookie中，同时TokenRepository会将Token放入数据库中。 

下次浏览器请求的时候，会经过RememberMeAuthenticationFiler，在这个filter里面会读取Cookie中的Token，然后去数据库中查找是否有相应的Token，找到token后再根据对应的用户名通过UserDetailsService获取用户的信息。


参考:

https://www.cnblogs.com/xuwenjin/p/9933218.html

https://docs.spring.io/spring-security/site/docs/5.2.0.M3/reference/htmlsingle/



