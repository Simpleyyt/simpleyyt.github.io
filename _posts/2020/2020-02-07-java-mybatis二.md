---
layout: post
categories: Java
title: mybatis系列之mybatis源码讲解
tagline: by 小九
tags: 
  - 小九
---

> hello~各位读者好，我是鸭血粉丝（大家可以称呼我为「阿粉」），在这个特殊的日子里,大家要注意安全，尽量不要出门，无聊的话，就像阿粉一样，把时间愉快的花在学习上吧。
<!--more-->
## 1.上期回顾

前面初识 `mybatis` 章节，阿粉首先搭建了一个简单的项目，只用了 `mybatis` 的 `jar` 包。然后通过一个测试代码，讲解了几个重要的类和步骤。

先看下这个测试类：

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

这章的话，阿粉会带着大家理解一下源码，基于上面测试的代码。阿粉先申明一下，源码一章肯定是讲不完的，所以阿粉会分成几个章节，并且讲源码的话，代码肯定比较多，比较干，所以请大家事先准备好开水哦。

### 2.源码分析

这里阿粉会根据 `SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream)` 这段代码来分析 `mybatis` 做了哪些事情。

**2.1设计模式**

通过这段代码，我们首先分析一下用到了哪些设计模式呢？

首先 `SqlSessionFactory` 这个类用到了工厂模式，并且上一章阿粉也说到了，这个类是全局唯一的，所以它还使用了单列模式。

然后是 `SqlSessionFactoryBuilder` 这个类，一看就知道用到了建造者模式。

所以，只看这段代码就用到了工厂模式，单列模式和建造者模式。

**2.2mybatis做了什么事情**

这里就开始源码之旅了，首先 `ctrl` + 左键点击 `build` 方法，我们会看到

```java
 public SqlSessionFactory build(InputStream inputStream) {
    return build(inputStream, null, null);
  }
```

再点击 `build` ，然后就是

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      return build(parser.parse());
        //后面代码省略
        ...
    }
}
```

首先 `XMLConfigBuilder` 这个类就是用来解析我们的配置文件 `mybatis-config.xml` ,调用这个类的构造函数,主要是创建一个 `Configuration` 对象，这个对象的属性就对应`mybatis-config.xml `里面的一级标签。这里不贴代码，有兴趣的自己点开看下。

然后就是 `build(parser.parse())` 这段代码，先执行 `parse()` 方法，再执行 `build()` 方法。不过这里，阿粉先说下 `build()` 这个方法，首先`parse()` 方法返回的就是 `Configuration` 对象,然后我们点击 `build` 

```java
public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }
```

这里就是返回的默认的`SqlSessionFactory` 。

然后我们再说 `parse()` 方法，因为这个是核心方法。我们点进去看下

```java
public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
```

首先是 `if` 判断，就是防止解析多次。看 `parseConfiguration` 方法

```java
private void parseConfiguration(XNode root) {
    try {
      //issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      loadCustomLogImpl(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```

这里就开始了解析`mybatis-config.xml`里面的一级标签了。阿粉不全部讲，只说几个主要的。

1. settings：全局配置，比如我们的二级缓存，延迟加载，日志打印等
2. mappers：解析dao层的接口和mapper.xml文件
3. properties：这个一般是配置数据库的信息，如：数据库连接，用户名和密码等，不讲
4. typeAliases ：参数和返回结果的实体类的别名，不讲
5. plugins：插件，如：分页插件，不讲
6. objectFactory，objectWrapperFactory：实例化对象用的，比如返回结果是一个对象，就是通过这个工厂反射获取的，不讲
7. environments：事物和数据源的配置，不讲
8. databaseIdProvider：这个是用来支持不同厂商的数据库
9. typeHandlers：这个是java类型和数据库的类型做映射，比如数据库的 `varchar` 类型对应 `java` 的`String` 类型，不讲

**2.3解析settings**

我们点击 `settingsElement(settings)` 这个方法

```java
private void settingsElement(Properties props) {
  configuration.setAutoMappingBehavior(AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));
  configuration.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));
  configuration.setCacheEnabled(booleanValueOf(props.getProperty("cacheEnabled"), true));
  configuration.setProxyFactory((ProxyFactory) createInstance(props.getProperty("proxyFactory")));
  configuration.setLazyLoadingEnabled(booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));
  configuration.setAggressiveLazyLoading(booleanValueOf(props.getProperty("aggressiveLazyLoading"), false));
  configuration.setMultipleResultSetsEnabled(booleanValueOf(props.getProperty("multipleResultSetsEnabled"), true));
  configuration.setUseColumnLabel(booleanValueOf(props.getProperty("useColumnLabel"), true));
  configuration.setUseGeneratedKeys(booleanValueOf(props.getProperty("useGeneratedKeys"), false));
  configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE")));
  configuration.setDefaultStatementTimeout(integerValueOf(props.getProperty("defaultStatementTimeout"), null));
  configuration.setDefaultFetchSize(integerValueOf(props.getProperty("defaultFetchSize"), null));
  configuration.setMapUnderscoreToCamelCase(booleanValueOf(props.getProperty("mapUnderscoreToCamelCase"), false));
  configuration.setSafeRowBoundsEnabled(booleanValueOf(props.getProperty("safeRowBoundsEnabled"), false));
  configuration.setLocalCacheScope(LocalCacheScope.valueOf(props.getProperty("localCacheScope", "SESSION")));
  configuration.setJdbcTypeForNull(JdbcType.valueOf(props.getProperty("jdbcTypeForNull", "OTHER")));
  configuration.setLazyLoadTriggerMethods(stringSetValueOf(props.getProperty("lazyLoadTriggerMethods"), "equals,clone,hashCode,toString"));
  configuration.setSafeResultHandlerEnabled(booleanValueOf(props.getProperty("safeResultHandlerEnabled"), true));
  configuration.setDefaultScriptingLanguage(resolveClass(props.getProperty("defaultScriptingLanguage")));
  configuration.setDefaultEnumTypeHandler(resolveClass(props.getProperty("defaultEnumTypeHandler")));
  configuration.setCallSettersOnNulls(booleanValueOf(props.getProperty("callSettersOnNulls"), false));
  configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), true));
  configuration.setReturnInstanceForEmptyRow(booleanValueOf(props.getProperty("returnInstanceForEmptyRow"), false));
    configuration.setLogPrefix(props.getProperty("logPrefix"));
  configuration.setConfigurationFactory(resolveClass(props.getProperty("configurationFactory")));
  }
```

这个就是设置我们的全局配置信息, `setXXX` 方法的第一个参数是 `mybatis-config.xml` 里面<setting>标签里面配置的。有的同学就会问了，阿粉阿粉，我们<setting>里面没有配置那么多啊，这里怎么设置那么多属性，值是什么呢？那我们看下方法的第二个参数，这个就是默认值。比如 `cacheEnabled` 这个属性，缓存开关，没有配置的话，默认是开启的 `true`。所有以后看全局配置的默认值，不用去官网看了，直接在这个方法里面看。

**2.4解析mappers**

点击 `mapperElement` 方法

```java
private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```

这里有4个判断 page (包)，resource（相对路径，阿粉配的就是这个，所有按照这个讲解源码），url（绝对路径），class（单个接口）

`XMLMapperBuilder` 这个类是用来解析mapper.xml文件的。然后我们点击 `parse` 方法

```java
public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
      configurationElement(parser.evalNode("/mapper"));
      configuration.addLoadedResource(resource);
      bindMapperForNamespace();
    }
    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
  }
```

if 判断是判断 `mapper` 是不是已经注册了，单个Mapper重复注册会抛出异常。

1. configurationElement：解析 mapper 里面所有子标签

   ```java
   private void configurationElement(XNode context) {
       try {
         String namespace = context.getStringAttribute("namespace");
         if (namespace == null || namespace.equals("")) {
           throw new BuilderException("Mapper's namespace cannot be empty");
         }
         builderAssistant.setCurrentNamespace(namespace);
         cacheRefElement(context.evalNode("cache-ref"));
         cacheElement(context.evalNode("cache"));
         parameterMapElement(context.evalNodes("/mapper/parameterMap"));
         resultMapElements(context.evalNodes("/mapper/resultMap"));
         sqlElement(context.evalNodes("/mapper/sql"));
         buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
       } catch (Exception e) {
         throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
       }
     }
   ```

   cacheRefElement：缓存

   cacheElement：是否开启二级缓存

   parameterMapElement：配置的参数映射

   resultMapElements：配置的结果映射

   sqlElement：公用sql配置

   buildStatementFromContext：解析 `select|insert|update|delete` 标签，获得 MappedStatement 对象,一个标签一个对象

2. bindMapperForNamespace：把namespace对应的接口的类型和代理工厂类绑定起来。

   ```java
   private void bindMapperForNamespace() {
       String namespace = builderAssistant.getCurrentNamespace();
       if (namespace != null) {
         Class<?> boundType = null;
         try {
           boundType = Resources.classForName(namespace);
         } catch (ClassNotFoundException e) {
           //ignore, bound type is not required
         }
         if (boundType != null) {
           if (!configuration.hasMapper(boundType)) {
             // Spring may not know the real resource name so we set a flag
             // to prevent loading again this resource from the mapper interface
             // look at MapperAnnotationBuilder#loadXmlResource
             configuration.addLoadedResource("namespace:" + namespace);
             configuration.addMapper(boundType);
           }
         }
       }
     }
   ```

   看最后 `addMapper` 方法

   ```java
   public <T> void addMapper(Class<T> type) {
       if (type.isInterface()) {
         if (hasMapper(type)) {
           throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
         }
         boolean loadCompleted = false;
         try {
           knownMappers.put(type, new MapperProxyFactory<>(type));
           // It's important that the type is added before the parser is run
           // otherwise the binding may automatically be attempted by the
           // mapper parser. If the type is already known, it won't try.
           MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
           parser.parse();
           loadCompleted = true;
         } finally {
           if (!loadCompleted) {
             knownMappers.remove(type);
           }
         }
       }
     }
   ```

   判断 `namespace `对应的类是否是接口，然后判断是否已经加载了这个接口，最后将`namespace `对应的接口和 `MapperProxyFactory` 放到 map 容器中。`MapperProxyFactory`  工厂模式,创建 `MapperProxy` 对象,这个一看就是代理对象。这个就是根据接口里面的方法获取 `mapper.xml`里面对应的sql语句的关键所在。具体的等下次讲解getMapper()源码的时候在深入的讲解。

### 3.时序图

![](http://www.justdojava.com\assets\images\2019\java\image-xiaojiu\20200207\1.jpg)

### 4.总结

在这一步，我们主要完成了 `config` 配置文件、`Mapper` 文件、`Mapper` 接口解析。我们得到了一个最重要的对象`Configuration`，这里面存放了全部的配置信息，它在属性里面还有各种各样的容器。最后，返回了一个`DefaultSqlSessionFactory`，里面持有了 `Configuration` 的实例。

