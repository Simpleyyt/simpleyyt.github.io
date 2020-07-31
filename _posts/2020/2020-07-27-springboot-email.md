---
layout: post
categories: SpringBoot
title: SpringBoot2.x 整合 邮件发送
tagline: by 淼淼之森
tags: 
    - 淼淼之森
---
在实际项目中，经常需要用到邮件通知功能。比如，用户通过邮件注册，通过邮件找回密码等；又比如通过邮件发送系统情况，通过邮件发送报表信息等等，实际应用场景很多。

正常我们会用 JavaMail 相关 api 来写发送邮件的相关代码，但现在 SpringBoot 提供了一套更简易使用的封装。这篇文章，阿粉就带大家通过 SpringBoot 快速的实现发送邮件的功能。

<!--more-->

## 1、开启smtp
这里以 163 邮箱为例。登录 163 邮箱之后，点击设置，如下图：

1.1、登录邮箱-设置
获取 `spring.mail.password` 授权码:

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/01.png)

1.2、开启IMAP/SMTP服务，根据提示走获取授权码

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/02.png)

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/03.png)

这个授权码，就是发送邮件时需要的密码。

1.3、下方有服务地址SMTP服务器：smtp.163.com就是我们要的

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/04.png)

以上步骤完成之后，就可以开始开发了。

## 2、新建 maven 项目

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/05.png)

## 3、 `pom` 文件中所涉及的依赖包
导入 SpringBoot 父依赖版本为 2.02
```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.2.RELEASE</version>
</parent>
```
导入 `web` 和 `mail` 邮件相关依赖包
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```
## 4、配置文件
配置文件中参数的获取，最后介绍。
```yml
server:
  port: 8082
spring:
  http:
    encoding:
      # 编码集
      charset: utf-8
      enabled: true
  mail:
    default-encoding: UTF-8
    #发送邮件的账户
    username: xxxxxxx@qq.com
    # 授权码（获取方式前文已描述）
    password: xxxxxxyyyyyy    
    # （邮箱服务器地址，获取方式前文已描述）  
    # 163 邮箱是smtp.163.com
    # qq邮箱则为smtp.qq.com
    host: smtp.163.com
    properties: 
      mail:
        smtp:
          ssl:
            enable: true
```            
## 5、发送类实现

Spring Email 抽象的核心是 MailSender 接口，MailSender 的实现能够把 Email 发送给邮件服务器，由邮件服务器实现邮件发送的功能。

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/06.png)

Spring 自带了一个 MailSender 的实现 `JavaMailSenderImpl`，它会使用 `JavaMail API` 来发送 Email。后来 spring 推出
 JavaMailSender 进一步简化邮件发送的过程，然后 `SpringBoot` 对此进行了封装，就有了
现在的 `spring-boot-starter-mail`。

接下来，阿粉和大家一起通过实例看看如何在 SpringBoot 中使用 `JavaMailSenderImpl` 发送邮件。


**简单消息邮件**：
```java
@Resource
private JavaMailSenderImpl javaMailSenderImpl;
    
public void sendSimpleMail() {
    SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
    // 设置邮件主题
    simpleMailMessage.setSubject("账号激活");
    // 设置要发送的邮件内容
    simpleMailMessage.setText("hello!");
    // 要发送的目标邮箱
    simpleMailMessage.setTo("yyyyyyyyyy@163.com");
    // 发送者邮箱和配置文件中的邮箱一致
    simpleMailMessage.setFrom("xxxxxxx@qq.com");
    javaMailSenderImpl.send(simpleMailMessage);
}
```

**复杂消息邮件**：
```java
@Resource
private JavaMailSenderImpl javaMailSenderImpl;

public void sendMimeMail() {
    MimeMessage mimeMessage = javaMailSenderImpl.createMimeMessage();
    try {
        // 开启文件上传
        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);
        // 设置文件主题
        mimeMessageHelper.setSubject("账号激活");
        // 设置文件内容 第二个参数设置是否支持html
        mimeMessageHelper.setText("<b style='color:red'>账号激活，请点击我</b>", true);
        // 设置发送到的邮箱
        mimeMessageHelper.setTo("yyyyyyyyyy@163.com");
        // 设置发送人和配置文件中邮箱一致
        mimeMessageHelper.setFrom("xxxxxxx@qq.com");
        // 上传附件
        // mimeMessageHelper.addAttachment("", new File(""));
    } catch (MessagingException e) {
        e.printStackTrace();
    }
    javaMailSenderImpl.send(mimeMessage);
}
```

## 6、 `Controller` 类
```java
@RestController
public class SendMailController {
	@Resource
	private SendMail sendMail;
	
	@GetMapping("/sendSimpleMail")
	public void sendSimpleMail() {
		sendMail.sendSimpleMail();
	}
	
	@GetMapping("/sendMimeMail")
	public void sendMimeMail() {
		sendMail.sendMimeMail();
	}
}
```
## 7、测试
### 7.1、简单邮件

利用 postman 发送请求：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/07.png)


查看邮箱结果：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/08.png)
### 7.2、复杂邮件

利用 postman 发送请求：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/09.png)

查看邮箱结果：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/07-01/10.png)

从结果可以看出，我们设置的 `<b style='color:red'>账号激活，请点击我</b>` 字体样式已经展示出效果了！

**参考** ：[https://www.cnblogs.com/jmcui/p/9758442.html](https://www.cnblogs.com/jmcui/p/9758442.html)


