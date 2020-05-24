---
layout: post
categories: Java
title: mybatis系列之mapper接口
tagline: by 小九
tags: 
  - 小九
---

> hello~各位读者好，我是鸭血粉丝（大家可以称呼我为「阿粉」）。今天，阿粉带着大家来了解一下 `mybatis` 接口的创建。

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

上篇源码分析讲了 `mybatis` 一级缓存的实现原理。这次，我们来了解下 `mybatis` 接口的创建。

## 2. mapper接口的创建流程

### 2.1 SqlSession的getMapper()

首先，我们来看下 `FruitMapper mapper = session.getMapper(FruitMapper.class);` 这段代码，意思很简单，根据传入的`class` 获取这个对象的实例。这个流程有点复杂，阿粉带着大家来跟下源码：

首先还是`ctrl` + 左键点击 `getMapper` 方法，然后会进入到 `SqlSession` 的 `getMapper()` 方法。然后之前阿粉也带着大家了解了， `SqlSession` 的默认实现类是 `DefaultSqlSession` ，所以我们直接看下 `getMapper()` 在 `DefaultSqlSession` 里面的实现：

```java
@Override
public <T> T getMapper(Class<T> type) {
    return configuration.getMapper(type, this);
}
```

### 2.2  Configuration 的getMapper()

这里从 `configuration` 里面去获取， `configuration` 是全局配置对象，也就是上下文。参数 `this` 是当前的`SqlSession` 对象，继续跟进去看下：

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
}
```

### 2.3  MapperRegistry  的getMapper()

`mapperRegistry` 对象是干什么的呢？继续点进去：

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
        throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
        return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
        throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
}
```

这里就不好看懂了，需要先看下了解下 `MapperRegistry` 这个类，我们一步一步来，跟着阿粉的思路走：

```java
public class MapperRegistry {

  private final Configuration config;
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();

  public MapperRegistry(Configuration config) {
    this.config = config;
  }
    ...
}
```

了解一个类，首先看下成员变量和构造方法。这里 `config` 不用多说了吧，主要的是 `knownMappers` 这个成员变量。这就是个`map` 对象，只是这个 `map` 对象的 `value`值是个对象，所以又要去看下 `MapperProxyFactory` 这个对象,点进去：

```java
public class MapperProxyFactory<T> {
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap<>();

  public MapperProxyFactory(Class<T> mapperInterface) {
    this.mapperInterface = mapperInterface;
  }
    ...
}
```

首先，单独看下这个类名  `MapperProxyFactory` ，取名是很有学问的，好的名字让你一下就知道是干啥的。所以一看   `MapperProxyFactory` ，首先就会联想到工厂模式，工厂模式是干啥的？创建对象的，创建什么对象呢？创建 `MapperProxy` 对象的。 `MapperProxy` 也是有玄机的，`Proxy` 的是什么？看到这个一般都是使用代理模式来创建代理对象的。所以就很清楚了，  `MapperProxyFactory` 这个类就是个工厂，创建的是 `mapper` 的代理对象。

然后这个类里面存的是 `mapper` 的接口和接口里面的方法。

最后，我们回到 `MapperRegistry`  类里面的 `getMapper()` 方法。现在是不是要清楚一些，通过 `mapper` 接口去 `map` 里面获取工厂类  `MapperProxyFactory` ，然后通过工厂类去创建我们的 `mapper` 代理对象。然后在看下  `getMapper()`  方法里面的 `mapperProxyFactory.newInstance(sqlSession);` 这段代码，继续点进去：

```java
public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
}
```

你看，阿粉猜测对不对，`MapperProxy` 对象是不是出来了。然后看 `newInstance()` 这个方法：

```java
protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }
```

两个 `newInstance()` 方法都在`MapperProxyFactory` 这个类里面，这里就很明显嘛。典型的 JDK 代理对象的创建。

好了，到这里我们的 `mapper `对象就获取到了。大家可以想一想，为什么获取一个 `mapper` 对象会那么复杂？或者说 `mapper` 对象有什么作用？其实就是为了通过 `mapper` 接口的方法获取到 `mapper.xml` 里面的 `sql`，具体怎么获取的，请允许阿粉卖个关子，请听阿粉下回分解。

## 3.总结

最后，阿粉以一个时序图来结束本篇文章，喜欢的话，记得点个赞哦。么么哒~

![](http://www.justdojava.com\assets\images\2019\java\image-xiaojiu\20200524\01.jpg)

