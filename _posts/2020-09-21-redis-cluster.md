---
layout: post
categories: Java
title: SpringBoot 项目接入 Redis 集群
tagline: by 子悠
tags: 
  - 子悠
---
Hello 大家好，我是鸭血粉丝，`Redis` 想必大家一定不会陌生，平常工作中或多或少都会用到，不管是用来存储登录信息还是用来缓存热点数据，对我们来说都是很有帮助的。但是 `Redis` 的集群估计并不是每个人都会用到，因为很多业务场景或者系统都是一些简单的管理系统，并不会需要用到 `Redis` 的集群环境。

阿粉之前也是这样，项目中用的的 `Redis` 是个单机环境，但是最近随着终端量的上升，慢慢的发现单机已经快支撑不住的，所以思考再三决定将 `Redis` 的环境升级成集群。下面阿粉给大家介绍一下在升级的过程中项目中需要调整的地方，这篇文章不涉及集群的搭建和配置，感兴趣的同学自行搜索。

<!--more-->

# 配置参数

因为这篇文章不介绍 `Redis` 集群的搭建，这里我们假设已经有了一个 `Redis` 的集群环境，我们项目中需要调整以下几个部分

1. 修改配置参数，集群的节点和密码配置；
2. 确保引入的 `Jedis` 版本支持设置密码，`spring-data-redis` 1.8 以上，`SpringBoot` 1.5 以上才支持设置密码；
3. 注入 `RedisTemplate`；
4. 编写工具类；

## 修改配置参数

```properties
############### Redis 集群配置 #########################
spring.custome.redis.cluster.nodes=172.20.0.1:7001,172.20.0.2:7002,172.20.0.3:7003
spring.custome.redis.cluster.max-redirects=3
spring.custome.redis.cluster.max-active=500
spring.custome.redis.cluster.max-wait=-1
spring.custome.redis.cluster.max-idle=500
spring.custome.redis.cluster.min-idle=20
spring.custome.redis.cluster.timeout=3000
spring.custome.redis.cluster.password=redis.cluster.password

```

## 引入依赖（如果需要）

确保 `SpringBoot` 的版本大于 1.4.x 如果不是的话，采用如下配置，先排除 `SpringBoot` 中旧版本 `Jedis` 和 `spring-data-redis`，再依赖高版本的 `Jedis` 和 `spring-data-redis`。

```pom
			<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <!-- 1.4 版本 SpringBoot 中 Jedis 不支持密码登录 -->
            <exclusions>
                <exclusion>
                    <groupId>redis.clients</groupId>
                    <artifactId>jedis</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework.data</groupId>
                    <artifactId>spring-data-redis</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- 手动依赖 Jedis 和 spring-data-redis-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
            <version>1.8.0.RELEASE</version>
        </dependency>
```

## 注入 `RedisTemplate`

注入 `RedisTemplate` 我们需要三个组件，分别是`JedisConnectionFactory` 、`RedisClusterConfiguration`、`JedisPoolConfig`，下面是注入`RedisTempalte` 的代码。先根据配置创建 `JedisConnectFactory` 同时需要配置 `RedisClusterConfiguration`、`JedisPoolConfig`，最后将`JedisConnectionFactory` 返回用于创建`RedisTemplate`

```java

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisNode;
import org.springframework.data.redis.connection.jedis.JedisClientConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;

public class RedisClusterConfig {

    @Bean(name = "redisTemplate")
    @Primary
    public RedisTemplate redisClusterTemplate(@Value("${spring.custome.redis.cluster.nodes}") String host,
                                     @Value("${spring.custome.redis.cluster.password}") String password,
                                     @Value("${spring.custome.redis.cluster.timeout}") long timeout,
                                     @Value("${spring.custome.redis.cluster.max-redirects}") int maxRedirect,
                                     @Value("${spring.custome.redis.cluster.max-active}") int maxActive,
                                     @Value("${spring.custome.redis.cluster.max-wait}") int maxWait,
                                     @Value("${spring.custome.redis.cluster.max-idle}") int maxIdle,
                                     @Value("${spring.custome.redis.cluster.min-idle}") int minIdle) {

        JedisConnectionFactory connectionFactory =  jedisClusterConnectionFactory(host, password,
                timeout, maxRedirect, maxActive, maxWait, maxIdle, minIdle);
        return createRedisClusterTemplate(connectionFactory);
    }

    private JedisConnectionFactory jedisClusterConnectionFactory(String host, String password,
                                                                   long timeout, int maxRedirect, int maxActive, int maxWait, int maxIdle, int minIdle) {
        RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration();
        List<RedisNode> nodeList = new ArrayList<>();
        String[] cNodes = host.split(",");
        //分割出集群节点
        for (String node : cNodes) {
            String[] hp = node.split(":");
            nodeList.add(new RedisNode(hp[0], Integer.parseInt(hp[1])));
        }
        redisClusterConfiguration.setClusterNodes(nodeList);
        redisClusterConfiguration.setPassword(password);
        redisClusterConfiguration.setMaxRedirects(maxRedirect);

        // 连接池通用配置
        GenericObjectPoolConfig genericObjectPoolConfig = new GenericObjectPoolConfig();
        genericObjectPoolConfig.setMaxIdle(maxIdle);
        genericObjectPoolConfig.setMaxTotal(maxActive);
        genericObjectPoolConfig.setMinIdle(minIdle);
        genericObjectPoolConfig.setMaxWaitMillis(maxWait);
        genericObjectPoolConfig.setTestWhileIdle(true);
        genericObjectPoolConfig.setTimeBetweenEvictionRunsMillis(300000);

        JedisClientConfiguration.DefaultJedisClientConfigurationBuilder builder = (JedisClientConfiguration.DefaultJedisClientConfigurationBuilder) JedisClientConfiguration
                .builder();
        builder.connectTimeout(Duration.ofSeconds(timeout));
        builder.usePooling();
        builder.poolConfig(genericObjectPoolConfig);
        JedisConnectionFactory connectionFactory = new JedisConnectionFactory(redisClusterConfiguration, builder.build());
        // 连接池初始化
        connectionFactory.afterPropertiesSet();

        return connectionFactory;
    }

    private RedisTemplate createRedisClusterTemplate(JedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key采用String的序列化方式
        redisTemplate.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        redisTemplate.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();

        return redisTemplate;
    }
}

```

> 相关代码已经提交到GitHub，公众号回复【源码仓库】即可获取。这里一定要注意 Jedis 的 Spring-data-redis 的版本支持设置密码，毕竟生产环境是一定要配置密码的。

## 编写工具类

其实到这里基本上已经完成了，我们可以看到 `SpringBoot` 项目接入 `Redis` 集群还是比较简单的，而且如果之前单机环境就是采用`RedisTemplate` 的话，现在也就不需要编写工具类，之前的操作依旧有效。不过作为贴心的阿粉，我还是给大家准备了一个工具类，代码太长，我只贴部分，需要完成代码的可以到公众号回复【源码仓库】获取。

```java
/**
     *  删除KEY
     * @param key
     * @return
     */
    public boolean delete(String key) {
        try {
            return getTemplate().delete(key);
        } catch (Exception e) {
            log.error("redis hasKey() is error");
            return false;
        }
    }

    /**
     * 普通缓存获取
     *
     * @param key 键
     * @return 值
     */
    public Object get(String key) {

        return key == null ? null : getTemplate().opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {

        try {
            getTemplate().opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            log.error("redis set() is error");
            return false;
        }

    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                getTemplate().opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            log.error("redis set() is error");
            return false;
        }
    }

    /**
     * 计数器
     *
     * @param key 键
     * @return 值
     */
    public Long incr(String key) {

        return getTemplate().opsForValue().increment(key);
    }

    public Long incrBy(String key, long step) {

        return getTemplate().opsForValue().increment(key, step);
    }

    /**
     * HashGet
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {

        return getTemplate().opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {

        return getTemplate().opsForHash().entries(key);
    }

    /**
     * 获取hashKey对应的批量键值
     * @param key
     * @param values
     * @return
     */
    public List<Object> hmget(String key, List<String> values) {

        return getTemplate().opsForHash().multiGet(key, values);
    }
```

上面随机列了几个方法，更多方案等待你的探索。

## 总结

今天阿粉给大家介绍了一下 `SpringBoot` 项目如何接入 `Redis` 集群，需要的朋友可以参考一下，不过阿粉还是要说一下，系统的设计不能过于冗余，如果短期内还能支撑业务的发展，那就暂时不要考虑太复杂，毕竟系统的架构是需要不断的完善的，不可能刚开始的时候就设计出一套很完善的系统框架。随着业务的不断发展，当真正发现单机`Redis` 已经无法满足业务需求的时候再接入也不迟！

# 写在最后

最后邀请你加入我们的知识星球，这里有 1800+ 优秀的人与你一起进步，如果你是小白那你是稳赚了，很多业内经验和干活分享给你；如果你是大佬，那可以进来我们一起交流分享你的经验，说不定日后我们还可以有合作，给你的人生多一个可能。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)
