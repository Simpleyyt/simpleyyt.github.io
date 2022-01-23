---
layout: post
categories: java基础
title: 如果有人问你 JFinal 如何集成  EhCache，把这篇文章甩给他
tagline: by 沉默王二
tags: 
  - 沉默王二
---

废话不多说，就说一句：在 JFinal 中集成 EhCache，可以提高系统的并发访问速度。

<!--more-->
可能有人会问 JFinal 是什么，EhCache 是什么，简单解释一下。

JFinal 是一个基于Java 语言的极速 Web 开发框架，用起来非常爽，谁用谁知道。EhCache 是一个纯 Java 的进程内缓存框架，具有快速、精干的特点，用起来非常爽，谁用谁知道。

JFinal 本身已经集成了 EhCache 这个缓存插件，但默认是没有启用的。那怎么启用呢？

请随我来。

### 01、在 pom.xml 中加入 EhCache 依赖

```xml
<dependency>
	<groupId>net.sf.ehcache</groupId>
	<artifactId>ehcache-core</artifactId>
	<version>2.6.11</version>
</dependency>
```

### 02、在 JFinalConfig 中配置 EhCachePlugin

```java
public class DemoConfig extends JFinalConfig {
  public void configPlugin(Plugins me) {
    me.add(new EhCachePlugin());
  }
}
```

基于 JFinal 的 Web 项目需要创建一个继承自 JFinalConfig 类的子类，该类用于对整个 Web 项目进行配置。

### 03、添加 ehcache.xml

在项目的 src 目录 / resources 目录下添加 ehcache.xml 文件，该文件的初始内容如下所示。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="ehcache.xsd"
         updateCheck="false" monitoring="autodetect"
         dynamicConfig="true">

    <diskStore path="java.io.tmpdir"/>

	<defaultCache
			maxEntriesLocalHeap="10000"
			eternal="false"
			timeToIdleSeconds="120"
			timeToLiveSeconds="120"

			diskSpoolBufferSizeMB="30"
			maxEntriesLocalDisk="10000000"
			diskExpiryThreadIntervalSeconds="120"
			memoryStoreEvictionPolicy="LRU"
			statistics="false">
		<persistence strategy="localTempSwap"/>
	</defaultCache>
    
</ehcache>
```

简单解释一下常用的配置项，否则大家在配置的时候容易犹豫不决。

1）maxEntriesLocalHeap：内存中最大缓存对象数 

2）eternal：true 表示对象永不过期，此时会忽略 timeToIdleSeconds 和 timeToLiveSeconds 属性，默认为 false

3）timeToIdleSeconds：对象最近一次被访问后的闲置时间，如果闲置的时间超过了 timeToIdleSeconds 属性值，这个对象就会过期，EhCache 将把它从缓存中清空；即缓存被创建后，最后一次访问时间到缓存失效的时候之间的间隔，单位为秒（s）

4）timeToLiveSeconds：对象被存放到缓存中后存活时间，如果存活时间超过了 timeToLiveSeconds 属性值，这个对象就会过期，EhCache 将把它从缓存中清除；即缓存被创建后，能够存活的最长时间，单位为秒（s）

假如我们现在增加以下配置：

```xml
<cache name="keywordsCache"
       maxEntriesLocalHeap="500"
       eternal="false"
       overflowToDisk="true"
       diskPersistent="true"
       timeToIdleSeconds="300"
       timeToLiveSeconds="600">
</cache>
```

结合之前的默认缓存配置，再来对比介绍下，大家就完全掌握了。

1）name 为该缓存的名字，后续使用缓存的时候要用到。

2）overflowToDisk：true 表示内存中缓存的对象数目达到了 maxEntriesLocalHeap 界限后，会把溢出的对象写到硬盘缓存中。此时的对象必须实现要实现 Serializable 接口（为什么？欢迎查看我以前的文章 [Java Serializable：明明就一个空的接口嘛](https://qingmiaogu.blog.csdn.net/article/details/93176705)）。  

3）diskPersistent：是否缓存虚拟机重启时的数据  

再来理解一下 timeToIdleSeconds 和 timeToLiveSeconds 这两个配置项。

```xml
timeToIdleSeconds="300"
timeToLiveSeconds="600"
```

以上表示，一个数据被添加进缓存后，该数据能够在缓存中存活的最长时间为 600 秒（）timeToLiveSeconds）；在这 600 秒内，假设不止一次去缓存中取该数据，那么相邻 2 次获取数据的时间间隔如果小于 300 秒（timeToIdleSeconds），则能成功获取到数据；但如果最近一次获取到下一次获取的时间间隔超过了 300 秒，那么，将得到 null，因为此时该数据已经被移出缓存了。

### 04、使用 CacheKit 操作缓存

 CacheKit 类是 JFinal 提供的缓存操作工具类，使用起来非常简便。

```java
Map<String, Keywords> map = CacheKit.get("keywordsCache", "keywordMap");
if (map == null) {
	map = new HashMap<>();

	List<Keywords> keywordList = dao.findAll();
	for (Keywords item : keywordList) {
		map.put(item.getKeyword(), item);
	}

	CacheKit.put("keywordsCache", "keywordMap", map);
}
```

CacheKit 中有两个最重要的方法：

1）`get(String cacheName, Object key)`，从 cache 中取数据。

2）`put(String cacheName, Object key, Object value)` ，将数据放入 cache 中。

参数 cacheName 与 ehcache.xml 中的 `<cache name="keywordsCache" …>` name 属性值对应，这个很好理解。

参数 key 是指取值用到的 key；参数 value 是被缓存的数据，这个其实也好理解。比如在上面的代码中，我们使用了 `keywordsCache` 这个配置项，在里面放了一个 HashMap，key 为 `keywordMap`，value 就是 map 这个对象。

JFinal 内部提供了很多使用 Ehcache 的工具方法，比如：

```java
List<Keywords> keywordList = dao.findByCache("keywordsCache", "keywordList", "select * from keywords");
```

这段代码的作用就是，当我们要从数据库中查询 Keywords 的时候，先从 Ehcache 缓存中取，如果缓存失效的话，再从数据库中取。

我是怎么知道的呢？当然不是靠猜的，我们来看一下源码。

```java
public List<M> findByCache(String cacheName, Object key, String sql, Object... paras) {
	Config config = _getConfig();
	ICache cache = config.getCache();
	List<M> result = cache.get(cacheName, key);
	if (result == null) {
		result = find(config, sql, paras);
		cache.put(cacheName, key, result);
	}
	return result;
}
```

### 05、最后

当数据的查询频率很高，远大于修改的频率，就要使用缓存了，这可以在很大程度上提高系统的性能。那现在我就提一个问题了，假如现在要修改一下数据，是先更新 DB，还是先更新缓存呢？