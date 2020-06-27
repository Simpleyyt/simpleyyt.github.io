---
layout: post
categories: Java
title: mybatis系列之获取mapper.xml配置文件中的sql
tagline: by 小九
tags: 
  - 小九
---

> hello~各位读者好，我是鸭血粉丝（大家可以称呼我为「阿粉」）。今天，阿粉带着大家来了解一下获取 `mapper.xml` 配置文件中的sql

## 1.上期回顾

首先，我们还是回顾一下上篇文件的类容。先看下这个测试类，大家还有印象吗：

```java
public class MybatisTest {
    @Test
    public void testSelect() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession session = sqlSessionFactory.openSession();
        try {
            FruitMapper mapper = session.getMapper(FruitMapper.class);
            Fruit fruit = mapper.findById(1L);
            System.out.println(fruit);
        } finally {
            session.close();
        }
    }
}
```

上篇源码分析讲了 获取`mybatis `的 `mapper` 接口，它是一个代理类。这次，我们来了解下 获取 `mapper.xml` 配置文件中的sql。

## 2. 初始化做了什么事

阿粉其实第一篇的时候就已经讲过了 `mybatis` 的初始化，但是已经有一段时间了。这篇文章又正好要讲怎么获取 `mapper.xml` 里面的 sql，所有这里阿粉再简单说下`mapper.xml`的初始化。

#### 2.1 mapper.xml

首先看下阿粉测试的`mapper.xml` 代码。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cl.mybatis.mapper.FruitMapper">
    
    <sql id="fruit">id,name </sql>
    
    <select id="findById" resultType="fruit">
        select <include refid="fruit"/> from fruit where id =#{id}
    </select>
</mapper>
```

#### 2.2  Configuration -> mappedStatements

然后,所有解析的 xml 都会放到`Configuration `这个类里面。那么跟着阿粉来看下`mapper.xml`解析之后是个什么东西。

这里阿粉就直接给出结果，就不再次去走下初始化流程了。

```java
protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection")
      .conflictMessageProducer((savedValue, targetValue) ->
          ". please check " + savedValue.getResource() + " and " + targetValue.getResource());
```

`mappedStatements` 是 `Configuration `类里面的一个属性，这个就是用来放我们的 `mapper.xml`里面的信息。我们来看下具体是怎么放的。

首先在 SqlSession session = sqlSessionFactory.openSession(); 这段代码打个断点，sqlSessionFactory 已经构建了，里面有一个  `Configuration ` 属性。看下阿粉截的图。

![1](http://www.justdojava.com\assets\images\2019\java\image-xiaojiu\20200627\1.png)

mappedStatements 是一个map，key 等于`com.cl.mybatis.mapper.FruitMapper.findById`，这个是什么？是不是就是`mapper.xml` 类的全路径,加上 `<select> `标签的 id 。然后 value 是一个 MappedStatement 对象，每个 `<select | update | insert | delete>` 会解析成一个 `MappedStatement`  对象。 `MappedStatement`  对象里面有一个 `sqlSource`  属性，这个放的就是我们自己写的 sql 。

现在是不是很清楚了，只要我们获取到`com.cl.mybatis.mapper.FruitMapper.findById` 这个 key，是不是就可以获取到对应的 sql，那怎么获取这个key呢？

## 3. 获取对应的key

#### 3.1 代理类 -> h

还记得阿粉上一篇文章吗？获取 mapper 接口，返回的是一个代理对象，使用的是JDK的动态代理。看下创建代理类的方法。

```java
protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }
```

mapperProxy 这个是我们的 h 类，h 类里面肯定有一个 invoke 方法。首先什么是 h 类？我们看下 newProxyInstance 方法。

```java
@CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        。。。
    }
```

实现 InvocationHandler 接口的类就是 h类。还是不懂的可以去看下 JDK 实现的动态代理模式，阿粉这里就不继续深入了。

#### 3.2 获取 key

现在们去看下 h类里面的 invoke 方法。MapperProxy -> invoke 

```java
@Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, args);
      } else if (isDefaultMethod(method)) {
        return invokeDefaultMethod(proxy, method, args);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args);
  }
```

我们看下倒数第二行的 cachedMapperMethod 方法，ctrl + 左键点进去。

```java
private MapperMethod cachedMapperMethod(Method method) {
    return methodCache.computeIfAbsent(method, k -> new MapperMethod(mapperInterface, method, sqlSession.getConfiguration()));
  }
```

看下这句代码 MapperMethod(mapperInterface,method,sqlSession.getConfiguration()))
我们来猜测一下，mapperInterface  是不是我们的 mapper 接口，这个能不能获取到 `mapper` 接口的全路径，method 是不是我们当前调用的方法 ，看下阿粉的测试方法：Fruit fruit = mapper.findById(1L) ，获取 findById() 这个方法的名字，就是 "findById"，一拼接就获取到了key，对不对？我们带正式一下，ctrl + 左键 MapperMethod。

```java
public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
    this.command = new SqlCommand(config, mapperInterface, method);
    this.method = new MethodSignature(config, mapperInterface, method);
  }
```

在倒数第一行代码上打个断点。我们看下 command 这个成员变量。

![2](http://www.justdojava.com\assets\images\2019\java\image-xiaojiu\20200627\2.png)

看，我们的 key 是不是拿到了，那么 sql 是不是也拿到了。 

## 总结

最后，阿粉在来梳理下这个流程：

* `mybatis` 初始化的时候，会将 `mapper.xml` 解析放到 `Configuration ` 类的`mappedStatements` 属性上。
* `mappedStatements` 是一个map ，它的 value 是一个对象，对象里面的一个属性存了我们写的 sql 语句。要想获取 value 必须找到对应的 key。
* 获取 key 是通过 JDK代理模式实现的，代理 mapper 接口，在执行这个接口的方法的时候，就会获取到当前接口的全路径和当前方法的名字。

好了，今天阿粉就讲到这里，下次阿粉会带着到家去了解一下，获取到 sql 之后，mybatis是怎么去执行 sql 的。我们下次 mybatis 源码分析再见。