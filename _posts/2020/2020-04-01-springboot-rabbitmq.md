---
layout: post
categories: springboot
title: 利用SpringBoot+RabbitMQ，实现一个邮件推送服务
tags: 
  - 炸鸡可乐
---

最近一直在学习RabbitMQ，但是不知如何在实际业务中撸出它的功效，最近刚好看到一篇相关文章，有一些心得，想和小伙伴们分享一下！

<!--more-->

### 一、先来一张 RabbitMQ 流程图

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/01.jpeg)

本文内容主要围绕这个流程图展开，利用 RabbitMQ 消息队列，实现生产者与消费者解耦，所以有必要先贴出来，涵盖了 RabbitMQ 很多知识点，如：

* 消息发送确认机制
* 消费确认机制
* 消息的重新投递
* 消费幂等性, 等等

### 二、实现思路

* 1.在虚拟机创建一个CentOS7上，并安装 RabbitMQ
* 2.开放QQ邮箱或者其它邮箱授权码，用于发送邮件
* 3.创建邮件发送项目并编写代码
* 4.发送邮件测试
* 5.消息发送失败处理

### 三、RabbitMQ安装
RabbitMQ 基于 erlang 进行通信，相比其它的软件，安装有些麻烦，不过本例采用`rpm`方式安装，任何新手都可以完成安装，过程如下！
#### 3.1、安装前命令准备
输入如下命令，完成安装前的环境准备。
```java
yum install lsof  build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz wget vim
```
#### 3.2、下载 RabbitMQ、erlang、socat 的安装包
本次下载的是`RabbitMQ-3.6.5`版本，采用`rpm`一键安装，适合新手直接上手。

先创建一个`rabbitmq`目录，本例的目录路径为`/usr/app/rabbitmq`，然后在目录下执行如下命令，下载安装包！

* 下载erlang

```java
wget www.rabbitmq.com/releases/erlang/erlang-18.3-1.el7.centos.x86_64.rpm
```

* 下载socat

```java
wget http://repo.iotti.biz/CentOS/7/x86_64/socat-1.7.3.2-5.el7.lux.x86_64.rpm
```

* 下载rabbitMQ

```java
wget www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-3.6.5-1.noarch.rpm
```
最终目录文件如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/02.jpg)

#### 3.3、安装软件包
下载完之后，按顺序依次安装软件包，这个很重要哦～

* 安装erlang

```java
rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm
```

* 安装socat

```java
rpm -ivh socat-1.7.3.2-5.el7.lux.x86_64.rpm
```

* 安装rabbitmq

```java
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm
```

安装完成之后，修改`rabbitmq`的配置，默认配置文件在`/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin`目录下。

```java
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app
```

修改`loopback_users`节点的值！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/03.jpg)

最后只需通过如下命令，启动服务即可！
```java
rabbitmq-server start &
```
运行脚本之后，如果报错，例如下图！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/04.jpg)

解决办法如下：
```java
vim /etc/rabbitmq/rabbitmq-env.conf
```
在文件里添加一行，如下配置！
```java
NODENAME=rabbit@localhost
```
然后，再保存！再次以下命令启动服务！
```java
rabbitmq-server start &
```
通过如下命令，查询服务是否启动成功！
```java
lsof -i:5672
```
如果出现`5672`已经被监听，说明已经启动成功！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/05.jpg)
#### 3.4、启动可视化的管控台
输入如下命令，启动控制台！
```java
rabbitmq-plugins enable rabbitmq_management
```
用浏览器打开`http://ip:15672`，这里的`ip`就是 CentOS 系统的 ip，结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/06.jpg)

账号、密码，默认为`guest`，如果出现无法访问，检测防火墙是否开启，如果开启将其关闭即可！

登录之后的监控平台，界面如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/07.jpg)

### 四、邮箱授权码的获取
获取邮箱授权码的目的，主要是为了通过代码进行发送邮件，例如 QQ 邮箱授权码获取方式，如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/08.jpg)

点击【开启】按钮，然后发送短信，即可获取授权码，该授权码就是配置文件`spring.mail.password`需要的密码！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/09.jpg)

### 五、项目介绍

* springboot版本：2.1.5.RELEASE
* RabbitMQ版本：3.6.5
* SendMailUtil：发送邮件工具类
* ProduceServiceImpl：生产者，发送消息
* ConsumerMailService：消费者，消费消息，发送邮件

### 六、代码实现
#### 6.1、创建项目
在 IDEA 下创建一个名称为`smail`的 Springboot 项目，`pom`文件中加入`amqp`和`mail`。
```xml
<dependencies>
    <!--spring boot核心-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!--spring boot 测试-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!--springmvc web-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--开发环境调试-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
    <!--mail 支持-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>
    <!--amqp 支持-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <!-- commons-lang3 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.4</version>
    </dependency>
    <!--lombok-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.16.10</version>
    </dependency>
</dependencies>
```
#### 6.2、配置rabbitMQ、mail
在`application.properties`文件中，配置`amqp`和`mail`！
```
#rabbitmq
spring.rabbitmq.host=192.168.0.103
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
# 开启confirms回调 P -> Exchange
spring.rabbitmq.publisher-confirms=true
# 开启returnedMessage回调 Exchange -> Queue
spring.rabbitmq.publisher-returns=true
# 设置手动确认(ack) Queue -> C
spring.rabbitmq.listener.simple.acknowledge-mode=manual
spring.rabbitmq.listener.simple.prefetch=100

# mail
spring.mail.default-encoding=UTF-8
spring.mail.host=smtp.qq.com
spring.mail.username=1370887518@qq.com
spring.mail.password=获取的邮箱授权码
spring.mail.from=1370887518@qq.com
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
```
其中，`spring.mail.password`第四步中获取的授权码，同时`username`和`from`要一致！
#### 6.3、RabbitConfig配置类
```java
@Configuration
@Slf4j
public class RabbitConfig {

    // 发送邮件
    public static final String MAIL_QUEUE_NAME = "mail.queue";
    public static final String MAIL_EXCHANGE_NAME = "mail.exchange";
    public static final String MAIL_ROUTING_KEY_NAME = "mail.routing.key";

    @Autowired
    private CachingConnectionFactory connectionFactory;

    @Bean
    public RabbitTemplate rabbitTemplate() {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(converter());

        // 消息是否成功发送到Exchange
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                log.info("消息成功发送到Exchange");
            } else {
                log.info("消息发送到Exchange失败, {}, cause: {}", correlationData, cause);
            }
        });

        // 触发setReturnCallback回调必须设置mandatory=true, 否则Exchange没有找到Queue就会丢弃掉消息, 而不会触发回调
        rabbitTemplate.setMandatory(true);
        // 消息是否从Exchange路由到Queue, 注意: 这是一个失败回调, 只有消息从Exchange路由到Queue失败才会回调这个方法
        rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            log.info("消息从Exchange路由到Queue失败: exchange: {}, route: {}, replyCode: {}, replyText: {}, message: {}", exchange, routingKey, replyCode, replyText, message);
        });

        return rabbitTemplate;
    }

    @Bean
    public Jackson2JsonMessageConverter converter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public Queue mailQueue() {
        return new Queue(MAIL_QUEUE_NAME, true);
    }

    @Bean
    public DirectExchange mailExchange() {
        return new DirectExchange(MAIL_EXCHANGE_NAME, true, false);
    }

    @Bean
    public Binding mailBinding() {
        return BindingBuilder.bind(mailQueue()).to(mailExchange()).with(MAIL_ROUTING_KEY_NAME);
    }
}
```
#### 6.4、Mail 邮件实体类
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Mail {

    @Pattern(regexp = "^([a-z0-9A-Z]+[-|\\.]?)+[a-z0-9A-Z]@([a-z0-9A-Z]+(-[a-z0-9A-Z]+)?\\.)+[a-zA-Z]{2,}$", message = "邮箱格式不正确")
    private String to;

    @NotBlank(message = "标题不能为空")
    private String title;

    @NotBlank(message = "正文不能为空")
    private String content;

    private String msgId;// 消息id
}
```
#### 6.5、SendMailUtil邮件发送类
```java
@Component
@Slf4j
public class SendMailUtil {

    @Value("${spring.mail.from}")
    private String from;

    @Autowired
    private JavaMailSender mailSender;

    /**
     * 发送简单邮件
     *
     * @param mail
     */
    public boolean send(Mail mail) {
        String to = mail.getTo();// 目标邮箱
        String title = mail.getTitle();// 邮件标题
        String content = mail.getContent();// 邮件正文

        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(title);
        message.setText(content);

        try {
            mailSender.send(message);
            log.info("邮件发送成功");
            return true;
        } catch (MailException e) {
            log.error("邮件发送失败, to: {}, title: {}", to, title, e);
            return false;
        }
    }
}
```
#### 6.6、ProduceServiceImpl 生产者类
```java
@Service
public class ProduceServiceImpl implements ProduceService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Override
    public boolean send(Mail mail) {
		//创建uuid
        String msgId = UUID.randomUUID().toString().replaceAll("-", "");
        mail.setMsgId(msgId);
		
		//发送消息到rabbitMQ
        CorrelationData correlationData = new CorrelationData(msgId);
        rabbitTemplate.convertAndSend(RabbitConfig.MAIL_EXCHANGE_NAME, RabbitConfig.MAIL_ROUTING_KEY_NAME, MessageHelper.objToMsg(mail), correlationData);

        return true;
    }
}
```
#### 6.7、ConsumerMailService 消费者类
```java
@Component
@Slf4j
public class ConsumerMailService {

    @Autowired
    private SendMailUtil sendMailUtil;

    @RabbitListener(queues = RabbitConfig.MAIL_QUEUE_NAME)
    public void consume(Message message, Channel channel) throws IOException {
        //将消息转化为对象
        String str = new String(message.getBody());
        Mail mail = JsonUtil.strToObj(str, Mail.class);
        log.info("收到消息: {}", mail.toString());

        MessageProperties properties = message.getMessageProperties();
        long tag = properties.getDeliveryTag();

        boolean success = sendMailUtil.send(mail);
        if (success) {
            channel.basicAck(tag, false);// 消费确认
        } else {
            channel.basicNack(tag, false, true);
        }
    }
}
```
#### 6.8、TestController 控制层类
```java
@RestController
@RequestMapping("/test")
@Slf4j
public class TestController {

    @Autowired
    private ProduceService testService;

    @PostMapping("send")
    public boolean sendMail(Mail mail) {
        return testService.send(mail);
    }
}
```
### 七、测试服务
启动 SpringBoot 服务之后，用 postman 模拟请求接口。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/10.jpg)

查看控制台信息。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/11.jpg)

查询接受者邮件信息。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/12.jpg)

邮件发送成功！

### 八、消息发送失败处理
虽然，上面案例可以成功的实现消息的发送，但是上面的流程很脆弱，例如： rabbitMQ 突然蹦了、邮件发送失败了、重启 rabbitMQ 服务器出现消息重复消费，应该怎处理呢？

很显然，我们需要对原有的逻辑进行升级改造，因此我们需要引入数据库来记录消息的发送情况。

#### 8.1、创建消息投递日志表
```sql
CREATE TABLE `msg_log` (
  `msg_id` varchar(255) NOT NULL DEFAULT '' COMMENT '消息唯一标识',
  `msg` text COMMENT '消息体, json格式化',
  `exchange` varchar(255) NOT NULL DEFAULT '' COMMENT '交换机',
  `routing_key` varchar(255) NOT NULL DEFAULT '' COMMENT '路由键',
  `status` int(11) NOT NULL DEFAULT '0' COMMENT '状态: 0投递中 1投递成功 2投递失败 3已消费',
  `try_count` int(11) NOT NULL DEFAULT '0' COMMENT '重试次数',
  `next_try_time` datetime DEFAULT NULL COMMENT '下一次重试时间',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`msg_id`),
  UNIQUE KEY `unq_msg_id` (`msg_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='消息投递日志';
```
#### 8.2、编写 MsgLog 相关服务类
```java
public interface MsgLogService {

    /**
     * 插入消息日志
     * @param msgLog
     */
    void insert(MsgLog msgLog);

    /**
     * 更新消息状态
     * @param msgId
     * @param status
     */
    void updateStatus(String msgId, Integer status);

    /**
     * 查询消息
     * @param msgId
     * @return
     */
    MsgLog selectByMsgId(String msgId);
}
```
#### 8.3、改写服务逻辑
在生产服务类中，新增数据写入。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/13.jpg)

同时，在`RabbitConfig`服务配置，当消息发送成功之后，新增更新消息状态逻辑。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/14.jpg)

改造消费者`ConsumerMailService`，每次消费的时候，从数据库中查询，如果消息已经被消费，不用再重复发送数据！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/15.jpg)

这样即可保证，如果 rabbitMQ 服务器，即使重启之后重新推送消息，通过数据库判断，也不会重复消费进而发生业务异常！

#### 8.4、利用定数任务对消息投递失败进行补偿
当 rabbitMQ 服务器突然挂掉之后，生成者就无法正常进行投递数据，此时因为消息已经被记录到数据库，因此我们可以利用定数任务查询出没有投递成功的消息，进行补偿投递。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-rabbitmq/16.jpg)

利用定数任务，对投递失败的消息进行补偿投递，基本可以保证消息 100% 消费成功！

### 九、总结
本文主要是通过发送邮件这个业务案例，来讲解 Springboot 与 rabbitMQ 技术的整合和使用！

当然解决这个业务需求的技术方案还有很多，例如 Springboot 与 rocketMQ 也可以实现这个需求，这个会在后期的文章讲解！

代码都经过自测，希望小伙伴能有所收获！

### 十、参考
1、[简书 - wangzaiplus - springboot + rabbitmq发送邮件(保证消息100%投递成功并被消费)](https://www.jianshu.com/p/dca01aad6bc8)
