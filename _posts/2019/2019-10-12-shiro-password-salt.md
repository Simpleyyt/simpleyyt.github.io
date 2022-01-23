---
layout: post
categories: shiro
title: 【Shiro 系列 07】Shiro 中密码加盐
tagline: by 江南一点雨
tags:
  - 江南一点雨
---

上篇文章和小伙伴们聊了密码加密问题，但是还不够，本文我们再来看看密码加盐问题。

<!--more-->

---

## 密码为什么要加盐

不管是消息摘要算法还是安全散列算法，如果原文一样，生成密文也是一样的，这样的话，如果两个用户的密码原文一样，存到数据库中密文也就一样了，还是不安全，我们需要做进一步处理，常见解决方案就是加盐。盐从那里来呢？我们可以使用用户 id（因为一般情况下，用户 id 是唯一的），也可以使用一个随机字符，我这里采用第一种方案。

## Shiro 中如何实现加盐

shiro 中加盐的方式很简单，在用户注册时生成密码密文时，就要加入盐，如下几种方式：

```java
Md5Hash md5Hash = new Md5Hash("123", "sang", 1024);
Sha512Hash sha512Hash = new Sha512Hash("123", "sang", 1024);
SimpleHash md5 = new SimpleHash("md5", "123", "sang", 1024);
SimpleHash sha512 = new SimpleHash("sha-512", "123", "sang", 1024)
```

然后我们首先将 sha512 生成的字符串放入数据库中，接下来我要配置一下我的 jdbcRealm ，因为我要指定我的盐是什么。在这里我的盐就是我的用户名，每个用户的用户名是不一样的，因此这里没法写死，在 JdbcRealm 中，系统提供了四种不同的 SaltStyle ，如下：

|SaltStyle|含义|
|:--|:--|
|NO_SALT|默认，密码不加盐|
|CRYPT|密码是以 Unix 加密方式储存的|
|COLUMN|salt 是单独的一列储存在数据库中|
|EXTERNAL|salt 没有储存在数据库中，需要通过 JdbcRealm.getSaltForUser(String) 函数获取|

四种不同的 SaltStyle 对应了四种不同的密码处理方式，部分源码如下：

```java
switch (saltStyle) {
case NO_SALT:
    password = getPasswordForUser(conn, username)[0];
    break;
case CRYPT:
    // TODO: separate password and hash from getPasswordForUser[0]
    throw new ConfigurationException("Not implemented yet");
    //break;
case COLUMN:
    String[] queryResults = getPasswordForUser(conn, username);
    password = queryResults[0];
    salt = queryResults[1];
    break;
case EXTERNAL:
    password = getPasswordForUser(conn, username)[0];
    salt = getSaltForUser(username);
}
```

在 COLUMN 这种情况下，SQL 查询结果应该包含两列，第一列是密码，第二列是盐，这里默认执行的 SQL 在 JdbcRealm 一开头就定义好了，如下：

```java
protected static final String DEFAULT_SALTED_AUTHENTICATION_QUERY = "select password, password_salt from users where username = ?";
```

即系统默认的盐是数据表中的 password_salt 提供的，但是我这里是 username 字段提供的，所以这里我一会要自定义这条 SQL。自定义方式很简单，修改 shiro.ini 文件，添加如下两行：

```java
jdbcRealm.saltStyle=COLUMN
jdbcRealm.authenticationQuery=select password,username from users where username=?
```

首先设置 saltStyle 为 COLUMN ，然后重新定义 authenticationQuery 对应的 SQL。注意返回列的顺序很重要，不能随意调整。如此之后，系统就会自动把 username 字段作为盐了。

不过，由于 ini 文件中不支持枚举，saltStyle 的值实际上是一个枚举类型，所以我们在测试的时候，需要增加一个枚举转换器在我们的 main 方法中，如下：

```java
BeanUtilsBean.getInstance().getConvertUtils().register(new AbstractConverter() {
    @Override
    protected String convertToString(Object value) throws Throwable {
        return ((Enum) value).name();
    }

    @Override
    protected Object convertToType(Class type, Object value) throws Throwable {
        return Enum.valueOf(type, value.toString());
    }

    @Override
    protected Class getDefaultType() {
        return null;
    }
}, JdbcRealm.SaltStyle.class);
```

当然，以后当我们将 shiro 和 web 项目整合之后，就不需要这个转换器了。

如此之后，我们就可以再次进行登录测试了，会发现没什么问题了。

## 非 JdbcRealm 如何配置盐

OK，刚刚是在 JdbcRealm 中配置了盐，如果没用 JdbcRealm ，而是自己定义的普通 Realm，要怎么解决配置盐的问题？

首先要说明一点是，我们前面的文章在自定义 Realm 时都是通过实现 Realm 接口实现的，这种方式有一个缺陷，就是密码比对需要我们自己完成，一般在项目中，我们自定义 Realm 都是通过继承 AuthenticatingRealm 或者 AuthorizingRealm ，因为这两个方法中都重写了 getAuthenticationInfo 方法，而在 getAuthenticationInfo 方法中，调用 doGetAuthenticationInfo 方法获取登录用户心些，获取到之后，会调用 assertCredentialsMatch 方法进行密码比对，而我们直接实现 Realm 接口则没有这一步，部分源码如下：

```java
public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    AuthenticationInfo info = getCachedAuthenticationInfo(token);
    if (info == null) {
        //调用doGetAuthenticationInfo获取info，这个doGetAuthenticationInfo是我们在自定义Realm中自己实现的
        info = doGetAuthenticationInfo(token);
        log.debug("Looked up AuthenticationInfo [{}] from doGetAuthenticationInfo", info);
        if (token != null && info != null) {
            cacheAuthenticationInfoIfPossible(token, info);
        }
    } else {
        log.debug("Using cached authentication info [{}] to perform credentials matching.", info);
    }
    if (info != null) {
        //获取到info之后，进行密码比对
        assertCredentialsMatch(token, info);
    } else {
        log.debug("No AuthenticationInfo found for submitted AuthenticationToken [{}].  Returning null.", token);
    }

    return info;
}
```

基于上面所述的原因，这里我先继承 AuthenticatingRealm ，如下：

```java
public class MyRealm extends AuthenticatingRealm {
    public String getName() {
        return "MyRealm";
    }
    public boolean supports(AuthenticationToken token) {
        return token instanceof UsernamePasswordToken;
    }
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = token.getPrincipal().toString();
        if (!"sang".equals(username)) {
            throw new UnknownAccountException("用户不存在");
        }
        String dbPassword = "a593ccad1351a26cf6d91d5f0f24234c6a4da5cb63208fae56fda809732dcd519129acd74046a1f9c5992db8903f50ebf3c1091b3aaf67a05c82b7ee470d9e58";
        return new SimpleAuthenticationInfo(username, dbPassword, ByteSource.Util.bytes(username), getName());
    }
}
```

关于这个类，我说如下几点：

1. 用户名我这里还是手动判断了下，实际上这个地方要从数据库查询用户信息，如果查不到用户信息，则直接抛 UnknownAccountException
2. 返回的 SimpleAuthenticationInfo 中，第二个参数是密码，正常情况下，这个密码是从数据库中查询出来的，我这里直接写死了
3. 第三个参数是盐值，这样构造好 SimpleAuthenticationInfo 之后返回，shiro 会去判断用户输入的密码是否正确

上面的核心步骤是第三步，系统去自动比较密码输入是否正确，在比对的过程中，需要首先对用户输入的密码进行加盐加密，既然加盐加密，就会涉及到credentialsMatcher ，这里我们要用的 credentialsMatcher 实际上和在 JdbcRealm 中用的 credentialsMatcher 一样（忘记的小伙伴可以先去复习下上篇文章），只需要在配置文件中增加如下一行即可：

```java
MyRealm.credentialsMatcher=$sha512
```

sha512 和我们上文定义的一致，这里就不再重复说了。

好了，关于密码加盐问题我们先说到这里。有问题欢迎留言讨论。

本文案例下载地址：https://github.com/lenve/shiroSamples/archive/v7.zip