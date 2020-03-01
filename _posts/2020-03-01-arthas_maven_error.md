---
layout: post
categories: Java
title: Arthas 实战，助你解决同名类依赖冲突问题
tagline: by 小黑
tags: 
  - 小黑
---

上篇文章中，阿粉分析 Maven 依赖冲突分为两类：

- 项目同一依赖应用，存在多版本，每个版本同一个类，可能存在差异。
- 项目不同依赖应用，存在包名，类名完全一样的类。

第二种情况，往往是这个场景，本地/测试环境运行的都是好好的，上线之后测试就是不行。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc486in8ogj3075072mx3.jpg)

这其实与 `JVM` 类加载有关，本地/测试环境加载正确类，而生产环节加载错的类，为什么会这样？

主要有两个原因：

- 同一个类只会被加载器加载一次
- 不同环境，类的加载顺序不同

## 同一个类只会被加载器加载一次

`JVM` 类加载具有缓存机制，每个类加载的时候首先检查一遍，类是否被当前类加载器加载。若未被加载，先交给其父类加载器加载，父类加载器不能加载，才会交给当前类加载器。

当前类加载器加载完成之后，将会将其缓存起来。

![图片来自网络](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc2lqxi64pj31140q6qv5.jpg)

类加载的核心源码位于 `ClassLoader#loadClass`：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc2lngio7zj30vs0u04qp.jpg)

① 处将会检查`ClassLoader#findLoadedClass` 最终将会调用 `ClassLoader#findLoadedClass0`,这是一个 `native` 方法，最终将会根据**类名加类加载器**查找缓存。

每个类加载器负责的加载范围都不一样：

- `BootstrapClassLoader` 引导类加载加载最核心的类库，如 **$JAVA_HOME/jre/lib/**
- `ExtClassLoader` 扩展类加载器负责加载`$JAVA_HOME/jre/lib/ext`下的一些扩展类
- `AppClassLoader` 应用类加载器将加载 `classpath` 指定的类。

我们运行的应用依赖的各种类，一般将会由 `AppClassLoader` 记载，同名类被加载后，下次碰到就不会再被加载。

> 画外音:利用缓存加快查询速度

## 不同环境，类的加载顺序不同

Java 可以使用 `-classpath` 参数指定依赖类所在位置。

类的加载顺序可以通过以下方式指定：


```shell
java -classpath a.jar:b.jar:c.jar xx.xx.Main
```

上面这种方式，类加载首先会从 **a.jar** 中查找相关类，找不到才会继续往后查找。所以可以通过这种方式可以指定使用哪个 `jar` 包内同名类。

但是这种方式有点繁琐，如果依赖 100 个 `jar` 包，需要全部写上去。

所以生产环境可以使用使用 `shell` 命令将 jar 拼接起来：

```shell
LIB_DIR=lib
LIB_JARS=`ls $LIB_DIR|grep .jar|awk '{print "'$LIB_DIR'/"$0}'|tr "\n" ":"`
```

另外 `java` 支持通配符的写法:

```shel
java -classpath './*' xx.xx.Main
```

这种方式的加载顺序将会受到底层系统文件加载顺序影响。

## 复现依赖冲突

假设我们现在应用依赖如下:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc36x8zewaj30ha0c0755.jpg)

A 应用依赖 B、C，且 B，C 中存在同包同名类 `org.example.App`，代码如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc375lgiulj30xu0u0grl.jpg)

如果指定 jar 包顺序启动应用：

```java
# A,B,C 放置同一文件夹下
java -classpath A-1.0-SNAPSHOT.jar:B-1.0-SNAPSHOT.jar:C-1.0-SNAPSHOT.jar org.example.ClassA
```

日志输出如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc37gleh4nj317i05in16.jpg)

改变 B ,C 顺序：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc37hxpa1aj31a2048whx.jpg)

类加载器的类的查找顺序将会通过 `classpath` 指定顺序从前往后查找。

如果使用通配符启动：

```java
java -classpath './*' org.example.ClassA
```

这种情况 `jvm` 到底加载那个类就成了薛定谔的**类**了，运行之前无法确定加载类来自哪个 `jar` 包。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc49hgv13xj3088065weh.jpg)

## 使用 verbose:class 打印加载类

我们可以在 `jvm` 启动脚本加入如下参数 `-verbose:class`,然后重启，日志里会打印出每个类的加载信息。

```shell
java -verbose:class -classpath './*' xx.xx.Main
```

日志输出如下:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc37mxgsv4j31kk0e2kfl.jpg)

不过这种方式需要重启应用，对生产系统来说，影响还是比较大,不太优雅。

## Arthas 查到来源类

阿里开源项目 **Arthas  sc** 可以用来查找加载类的信息。。

**sc** 命令是 **Search-Class** 简写，这个命令能搜索出所有已经加载到 JVM 中的 Class 信息，支持参数如下表格所示。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc1zzv6zt5j315g0jotry.jpg)

程序启动之后，启动 **arthas**，进入 A 应用。

运行如下命令：

```java
sc -d org.example.App
```

输出结果如下 :

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc37pt1k00j31f20li1kx.jpg)

**code-source** 显示当前查找类 `org.example.App` 来自的 C。

另外我们可以 **jad** 命令反编译类，在线查看源码。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200301/0082zybply1gc37rqsc4zj312e0n8h1c.jpg)

## 总结

这篇文章主要解释应用中存在多个同名类，环境不同，类加载不同的原因。接着介绍了两种快速查找运行应用依赖类来源的方法。

当定位到了冲突类的来源，我们可以显示指定 `classpath` jar 包的顺序，指定类加载的顺序。但这只是暂时解决一下。本质依赖冲突的问题，还是深层次排除的。