---
layout: post
categories: SpringBoot
title: SpringBoot2.x 配合 Redis 操作
tagline: by 淼淼之森
tags: 
    - 淼淼之森
---

我们都知道，把首页数据放到Redis里，能够加快首页数据的访问速度。但是我们要如何准确又快速的将 Redis 整合到自己的 SpringBoot2.x 项目中呢？今天阿粉就带大家爬一爬其中的门门道道。
<!--more-->


## 序、Redis 介绍

Redis 使用了浪费流量的文本协议，但因为它数据存储在内存中的，相对而言，依然可以取得极高的访问性能。并且 Redis 是线程安全的。

RESP 就是 Redis 序列化协议的简称。它是一种直观的文本协议，优势在于实现异常简单，解析性能极好。

Redis 协议里面虽然有大量冗余的回车换行符，但是这不影响它成为技术领域非常受欢迎的一个文本协议。在技术领域，性能并不总是一切，还有简单性、易理解性和易实现性，这些都需要进行适当权衡。

## 一、Redis 基础数据结构
**1、字符串**：（缓存）
- key：value

value 可以是对象转换成的 JSON 字符串，也可以是对象序列化后的二进制字符串

**2、列表**：（异步队列）
类似linkedlist
- 右边进左边出：队列
- 右边进右边出：栈

**3、字典**（哈希）
类似hashmap：数组+链表

不过rehash是渐进式hash策略

**4、集合**：（去重）
- 无序 set： 类似hashset
- 有序 zset：类似SortedSet和HashMap的结合体，内部是现实现是跳跃列表


## 二、Lettuce
随着 Spring Boot2.x 的到来，支持的组件越来越丰富，也越来越成熟，其中对 Redis 的支持不仅仅是丰富了它的API，更是替换掉底层 Jedis 的依赖，取而代之换成了 Lettuce。

虽然 Lettuce 和 Jedis 的都是连接 Redis Server 的客户端程序，但是 Jedis 在实现上是直连 redis server，多线程环境下非线程安全，除非使用连接池，为每个Jedis实例增加物理连接。而 Lettuce 基于 Netty 的连接实例（StatefulRedisConnection），可以在多个线程间并发访问，且线程安全，满足多线程环境下的并发访问，同时它是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。

Lettuce是可扩展的Redis客户端，用于构建无阻塞的Reactive应用程序.

Luttuce官网：[https://lettuce.io/](https://lettuce.io/)

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/06-02/00.jpg)

谷歌翻译后的页面是：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/06-02/01.png)

原来这玩意儿叫生菜，你别说，看着图标还真有点像。

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/06-02/02.jpg)


## 三、实操
项目中使用的 SpringBoot2.x 实现，如果之前是 SpringBoot1.x 则需要注意，底层已经由 Jedis 升级成了 Lettuce 。
### 3.1、引入依赖
除去 SpringBoot 项目需要的 jar 包外，另外还需要引入 redis 相关的依赖：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```
### 3.2、`application.yml` 配置文件
此处用到的 `application.yml` 文件，配置如下:
```
spring:
  redis:
    # Redis默认情况下有16个分片，这里配置具体使用的分片。默认是索引为0的分片
    database: 1
    # Redis服务器地址
    host: 127.0.0.1
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    password: mmzsblog
    # 连接超时时间（毫秒）
    timeout: 2000s

    # 配置文件中添加 lettuce.pool 相关配置，则会使用到lettuce连接池
    lettuce:
      pool:
        # 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
        max-wait: 60s
        # 连接池中的最大空闲连接 默认 8
        max-idle: 10
        # 连接池中的最小空闲连接 默认 0
        min-idle: 10
        # 连接池最大连接数（使用负值表示没有限制） 默认 8
        max-activ: 8
```

如果项目是由 SpringBoot1.x 升级到 SpringBoot2.x 的，要沿用 jedis 连接池配置时会用到配置 jedis 相关的属性：
```yml
    # 配置文件中添加 jedis.pool 相关配置，则会使用到 jedis 连接池
    jedis:
      pool:
        max-active: 10
        max-idle: 8
        min-idle: 0
        max-wait: 60s
```
并且引用的 jar 包也需要调整：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```


另外，这里再贴一下 Spring Boot 关于 RedisProperties 的所有配置项
```properties 
# REDIS RedisProperties
spring.redis.cluster.max-redirects= # Maximum number of redirects to follow when executing commands across the cluster.
spring.redis.cluster.nodes= # Comma-separated list of "host:port" pairs to bootstrap from.
spring.redis.database=0 # Database index used by the connection factory.
spring.redis.url= # Connection URL. Overrides host, port, and password. User is ignored. Example: redis://user:password@example.com:6379
spring.redis.host=localhost # Redis server host.
spring.redis.jedis.pool.max-active=8 # Maximum number of connections that can be allocated by the pool at a given time. Use a negative value for no limit.
spring.redis.jedis.pool.max-idle=8 # Maximum number of "idle" connections in the pool. Use a negative value to indicate an unlimited number of idle connections.
spring.redis.jedis.pool.max-wait=-1ms # Maximum amount of time a connection allocation should block before throwing an exception when the pool is exhausted. Use a negative value to block indefinitely.
spring.redis.jedis.pool.min-idle=0 # Target for the minimum number of idle connections to maintain in the pool. This setting only has an effect if it is positive.
spring.redis.lettuce.pool.max-active=8 # Maximum number of connections that can be allocated by the pool at a given time. Use a negative value for no limit.
spring.redis.lettuce.pool.max-idle=8 # Maximum number of "idle" connections in the pool. Use a negative value to indicate an unlimited number of idle connections.
spring.redis.lettuce.pool.max-wait=-1ms # Maximum amount of time a connection allocation should block before throwing an exception when the pool is exhausted. Use a negative value to block indefinitely.
spring.redis.lettuce.pool.min-idle=0 # Target for the minimum number of idle connections to maintain in the pool. This setting only has an effect if it is positive.
spring.redis.lettuce.shutdown-timeout=100ms # Shutdown timeout.
spring.redis.password= # Login password of the redis server.
spring.redis.port=6379 # Redis server port.
spring.redis.sentinel.master= # Name of the Redis server.
spring.redis.sentinel.nodes= # Comma-separated list of "host:port" pairs.
spring.redis.ssl=false # Whether to enable SSL support.
spring.redis.timeout= # Connection timeout.
```
### 3.3、自定义一个 `RedisTemplate` 
这个看你自己，不自定义也不影响使用，只是说可能不那么顺手，所以阿粉习惯自定义一个。因为 `Spring Boot` 在 `RedisAutoConfiguration` 中默认配置了 `RedisTemplate<Object, Object>`、`StringRedisTemplate`两个模板类，然而`RedisTemplate<Object, Object>`并未指定`key`、`value`的序列化器。

```java
@Configuration
public class RestTemplateConfig {
    @Bean
    public RedisTemplate<String, Serializable> redisCacheTemplate(LettuceConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Serializable> template = new RedisTemplate<>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```
### 3.4、`Person` 实体类
声明一个 `Person` 实体类：
```java
/**
 * @author ：created by mmzsblog
 * @date ：created at 2020/06/23 16:41
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person implements Serializable {

    private static final long serialVersionUID = -8183942491930372236L;
    private Long userId;
    private String username;
    private String password;
}
```



### 3.5、测试
通过编写一个 UserController 来进行测试：
```java
@RestController
public class UserController {
    @Resource
    private RedisTemplate redisTemplate;

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @GetMapping("/set")
    public String set() {
        stringRedisTemplate.opsForValue().set("one", "1");
        
        // redisTemplate 保存的是字节序列，因为 RestTemplateConfig 自定义的时候指定了 key 和 value 的序列化器。
        redisTemplate.opsForValue().set("two", "2");
        redisTemplate.opsForValue().set("person", new Person(1L, "luffy", "123456789"));

        // 测试线程安全
        ExecutorService executorService = Executors.newFixedThreadPool(1000);
        IntStream.range(0, 1000).forEach(i -> {
            executorService.execute(() -> stringRedisTemplate.opsForValue().increment("num", 1));
        });
        return "Ok!";
    }

    @GetMapping("/get")
    public String get() {
        String one = stringRedisTemplate.opsForValue().get("one");
        if ("1".equals(one)) {
            System.out.println("key: one" + " || value: " + one);
        }

        Object two = redisTemplate.opsForValue().get("two");
        if ("2".equals(two.toString())) {
            System.out.println("key: two" + " || value: " + two);
        }

        Person user = (Person) redisTemplate.opsForValue().get("person");
        if ("luffy".equals(user.getUsername())) {
            System.out.println("key: person" + " || value: " + user);
        }
        return "Ok!";
    }
}
```
**用RedisDesktopManager工具查看，数据如下：**

用 `StringRedisTemplate` 设置的键值是String类型的：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/06-02/03.png)

用 `RedisTemplate` 设置的键值是二进制的字节流形式存储的，从截图中的 `[Binary]` 标识符也能看出：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/06-02/04.png)

自定义的 `RedisTemplate` 和 `StringRedisTemplate` 并不会有什么冲突，想用 String 存储还是二进制的字节流形式存储完全取决于你自己。




## 参考
- https://lettuce.io/
- [Spring Boot官方文档 91.4](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-use-jedis-instead-of-lettuce)
- 《Redis深度历险：核心原理和应用实践》
- http://blog.battcn.com/2018/05/11/springboot/v2-nosql-redis/
- https://www.jianshu.com/p/f7d11e7109b7