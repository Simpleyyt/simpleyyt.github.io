---
layout: post
categories: shiro
title: 【Shiro 系列 06】Shiro 中密码加密
tagline: by 江南一点雨
tags:
  - 江南一点雨
---

上篇文章和小伙伴们分享了 Realm 的认证策略问题，本文我想和小伙伴们来聊一聊密码的加密问题。

<!--more-->

---

## 密码为什么要加密

2011 年 12 月 21 日，有人在网络上公开了一个包含 600 万个 CSDN 用户资料的数据库，数据全部为明文储存，包含用户名、密码以及注册邮箱。事件发生后 CSDN 在微博、官方网站等渠道发出了声明，解释说此数据库系 2009 年备份所用，因不明原因泄露，已经向警方报案。后又在官网网站发出了公开道歉信。在接下来的十多天里，金山、网易、京东、当当、新浪等多家公司被卷入到这次事件中。整个事件中最触目惊心的莫过于 CSDN 把用户密码明文存储，由于很多用户是多个网站共用一个密码，因此一个网站密码泄露就会造成很大的安全隐患。由于有了这么多前车之鉴，我们现在做系统时，密码都要加密处理。

密码加密我们一般会用到散列函数，又称散列算法、哈希函数，是一种从任何一种数据中创建小的数字“指纹”的方法。散列函数把消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来。该函数将数据打乱混合，重新创建一个叫做散列值的指纹。散列值通常用一个短的随机字母和数字组成的字符串来代表。好的散列函数在输入域中很少出现散列冲突。在散列表和数据处理中，不抑制冲突来区别数据，会使得数据库记录更难找到。我们常用的散列函数有如下几种：

### 1.MD5 消息摘要算法

MD5消息摘要算法是一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值，用于确保信息传输完整一致。MD5由美国密码学家罗纳德·李维斯特设计，于1992年公开，用以取代MD4算法。这套算法的程序在 RFC 1321中被加以规范。将数据（如一段文字）运算变为另一固定长度值，是散列算法的基础原理。1996年后被证实存在弱点，可以被加以破解，对于需要高度安全性的数据，专家一般建议改用其他算法，如SHA-2。2004年，证实MD5算法无法防止碰撞，因此不适用于安全性认证，如SSL公开密钥认证或是数字签名等用途。

### 2.安全散列算法

安全散列算法（Secure Hash Algorithm）是一个密码散列函数家族，是 FIPS 所认证的安全散列算法。能计算出一个数字消息所对应到的，长度固定的字符串（又称消息摘要）的算法。且若输入的消息不同，它们对应到不同字符串的机率很高。SHA 家族的算法，由美国国家安全局所设计，并由美国国家标准与技术研究院发布，是美国的政府标准，其分别是：

- SHA-0：1993 年发布，是 SHA-1 的前身；
- SHA-1：1995 年发布，SHA-1 在许多安全协议中广为使用，包括 TLS 和 SSL、PGP、SSH、S/MIME 和 IPsec，曾被视为是 MD5 的后继者。但 SHA-1 的安全性在 2000 年以后已经不被大多数的加密场景所接受。2017 年荷兰密码学研究小组 CWI 和 Google 正式宣布攻破了 SHA-1；
- SHA-2：2001 年发布，包括 SHA-224、SHA-256、SHA-384、SHA-512、SHA-512/224、SHA-512/256。虽然至今尚未出现对 SHA-2 有效的攻击，它的算法跟 SHA-1 基本上仍然相似；因此有些人开始发展其他替代的散列算法；
- SHA-3：2015 年正式发布，SHA-3 并不是要取代 SHA-2，因为 SHA-2 目前并没有出现明显的弱点。由于对 MD5 出现成功的破解，以及对 SHA-0 和 SHA-1 出现理论上破解的方法，NIST 感觉需要一个与之前算法不同的，可替换的加密散列算法，也就是现在的 SHA-3。

## Shiro 中如何加密

Shiro 中对以上两种散列算法都提供了支持，对于 MD5，Shiro 中生成消息摘要的方式如下：

```java
Md5Hash md5Hash = new Md5Hash("123", null, 1024);
```

第一个参数是要生成密码的明文，第二个参数密码的盐值，第三个参数是生成消息摘要的迭代次数。

Shiro 中对于安全散列算法的支持如下(支持多种算法，这里我举一个例子)：

```java
Sha512Hash sha512Hash = new Sha512Hash("123", null, 1024);
```

这里三个参数含义与上文基本一致，不再赘述。shiro 中也提供了通用的算法，如下：

```java
SimpleHash md5 = new SimpleHash("md5", "123", null, 1024);
SimpleHash sha512 = new SimpleHash("sha-512", "123", null, 1024);
```

当用户注册时，我们可以通过上面的方式对密码进行加密，将加密后的字符串存入数据库中。我这里为了简单，就不写注册功能了，就把昨天数据库中用户的密码 123 改成 sha512 所对应的字符串，如下：

```java
cb5143cfcf5791478e057be9689d2360005b3aac951f947af1e6e71e3661bf95a7d14183dadfb0967bd6338eb4eb2689e9c227761e1640e6a033b8725fabc783
```

同时，为了避免其他 Realm 的干扰，数据库中我只配置一个 JdbcRealm ，配置方式参考本系列[第四篇文章]()。

此时如果我不做其他修改的话，登录必然会失败，原因很简单：我登录时输入的密码是 123，但是数据库中的密码是一个很长的字符串，所以登录肯定不会成功。通过打断点，我们发现最终的密码比对是在 SimpleCredentialsMatcher 类中的 doCredentialsMatch 方法中进行密码比对的，比对的方式也很简单，直接使用了对用户输入的密码和数据库中的密码生成 byte 数组然后进行比较，最终的比较在 MessageDigest 类的 isEqual 方法中。部分逻辑如下：

```java
protected boolean equals(Object tokenCredentials, Object accountCredentials) {
        ...
        ...
        //获取用户输入密码的byte数组
        byte[] tokenBytes = this.toBytes(tokenCredentials);
        //获取数据库中密码的byte数组
        byte[] accountBytes = this.toBytes(accountCredentials);
        return MessageDigest.isEqual(tokenBytes, accountBytes);
        ...
}
```

MessageDigest 的 isEqual 方法如下：

```java
public static boolean isEqual(byte[] digesta, byte[] digestb) {
    if (digesta == digestb) return true;
    if (digesta == null || digestb == null) {
        return false;
    }
    if (digesta.length != digestb.length) {
        return false;
    }

    int result = 0;
    // time-constant comparison
    for (int i = 0; i < digesta.length; i++) {
        result |= digesta[i] ^ digestb[i];
    }
    return result == 0;
}
```

都是很容易理解的比较代码，这里不赘述。我们现在之所以登录失败是因为没有对用户输入的密码进行加密，通过对源代码的分析，我们发现是因为在 AuthenticatingRealm 类的 assertCredentialsMatch 方法中获取了一个名为 SimpleCredentialsMatcher 的密码比对器，这个密码比对器中比对的方法就是简单的比较，因此如果我们能够将这个密码比对器换掉就好了。我们来看一下 CredentialsMatcher 的继承关系：

![p310](http://www.justdojava.com/assets/images/2019/java/image_javaboy/shiro/6-1.jpg)

我们发现这个刚好有一个 Sha512CredentialsMatcher 比对器，这个比对器的 doCredentialsMatch 方法在它的父类 HashedCredentialsMatcher，方法内容如下：

```java
public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
    Object tokenHashedCredentials = hashProvidedCredentials(token, info);
    Object accountCredentials = getCredentials(info);
    return equals(tokenHashedCredentials, accountCredentials);
}
```

这时我们发现获取 tokenHashedCredentials 的方式不像以前那样简单粗暴了，而是调用了 hashProvidedCredentials 方法，而 hashProvidedCredentials 方法最终会来到下面这个重载方法中：

```java
protected Hash hashProvidedCredentials(Object credentials, Object salt, int hashIterations) {
    String hashAlgorithmName = assertHashAlgorithmName();
    return new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
}
```

这几行代码似曾相识，很明显，是系统帮我们对用户输入的密码进行了转换。了解了这些之后，那我只需要将 shiro.ini 修改成如下样子即可实现登录了：

```ini
sha512=org.apache.shiro.authc.credential.Sha512CredentialsMatcher
# 迭代次数
sha512.hashIterations=1024
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql://localhost:3306/shiroDemo
dataSource.username=root
dataSource.password=123
jdbcRealm.dataSource=$dataSource
jdbcRealm.permissionsLookupEnabled=true
# 修改JdbcRealm中的credentialsMatcher属性
jdbcRealm.credentialsMatcher=$sha512
securityManager.realms=$jdbcRealm
```

如此之后，我们再进行登录测试，就可以登录成功了。

好了，密码加密我们先说到这里，下篇文章我们来聊聊密码加盐的问题。

本文案例下载地址：https://github.com/lenve/shiroSamples/archive/v6.zip