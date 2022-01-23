---
layout: post
categories: spring
title: Spring 注解编程之注解属性别名与覆盖
tagline: by 小黑
tags: 
  - 小黑

---

前两篇文章咱聊了深入了解了 Spring 注解编程一些原理，这篇文章我们关注注解属性方法，聊聊 Spring 为注解的带来的功能，属性别名与覆盖。

<!--more-->

## 注解属性方法

在进入了解 Spring 注解属性功能之前，我们先看一个正常 Java 注解。

![annotation.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/annotation-84595ecf.png)

在注解中，属性方法与其他类/接口方法写法类似，但是存在一些区别。

注解属性方法的返回类型仅限为八种基本类型(包装类不支持)，字符串，class，enum，Annotation以及前面类型的数组。

> 复习一下，java 八种基本类型分别为，byte（字节型）、short（短整型）、int（整型）、long（长整型）、float（单精度浮点型）、double（双精度浮点型）、boolean（布尔型）、char（字符型）。

其次，注解属性方法可以使用 `default`设置默认值。如果没有设置默认值，声明注解时必须显式设置属性，否则编译将会出错。

另外 Java 注解无法继承类，也无法实现接口。

## Spring 属性方法特性

在 Spring 中，有一些注解，使用不同属性方法，却能到达相同结果。典型的如 `RequestMapping`。

在 WEB 项目中，设置 url 路径，我们可以在方法是这样声明：

```java
    @RequestMapping("hello")
    public String helloAnnotation() {
	。。。。
    }
```
> 上面方法本质使用注解 `value` 属性。当注解声明时只需要设置一个方法时，如果属性方法为 value，不需要使用 key=value 的语法，只需要直接设置属性值即可。

另外也可以使用 `path` 属性方法设置。

```java
    @RequestMapping(path = "hello")
    public String helloAnnotation() {
	。。。。
    }
```
两种方式，最后运行效果一致。

查看 `RequestMapping` 注解源码，可以发现在 `value` 与 `path` 属性方法上使用 `@AliasFor`，并且两个互相指向对方。

![RequestMapping.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/RequestMapping-531341c7.png)

Spring 4.2 加入 `@AliasFor` 注解，并使用  `@AliasFor` 重新更新 `RequestMapping`等注解，为它们内部带来了别名的功能。

## `@AliasFor` 使用方式

在 Spring 中，`@AliasFor` 可以在同一注解中使用，使用方法如 `RequestMapping` 注解。

这种方式，带来含义明确属性方法。如 `RequestMapping`，`path` 属性方法，这个属性方法含义就比较明确，不同的人理解不会有偏差。而 `value` 属性含义就不是很明确，不能一下子就将它真正含义产生联系。

> 日常开发中，我们也要避免 i,a,b 这些无意义的命名，尽量使用含义明确的命名。这样利用维护代码的人理解。

第二点，同一注解属性方法相互别名，这样就兼容之前版本用法。

`RequestMapping` 注解如果仅新增 `path` 属性，然后根据其解析 url 路径，这样就会导致升级 Spring 版本过程，运行错误的。

> 一个好软件版本需要时向前兼容，如 JDK 8 兼容 JDK 6一样。

另外 `@AliasFor` 注解还可以作用与不同注解之前，典型的如 `SpringBootApplication`注解。

![SpringBootApplication.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/SpringBootApplication-6fcc903e.png)

 `SpringBootApplication#scanBasePackages` 别名与 `ComponentScan#basePackages`。设置前者间接为后者赋值。

Spring Boot 就是使用 `@Aliasfor` 与组合注解功能，使用 `SpringBootApplication`一个注解代替 `Configuration`，`EnableAutoConfiguration`，`ComponentScan`。

## Spring 注解属性覆盖与别名

使用 `@AliasFor` 注解，可以做到别名的功能。


在 Spring 中别名可以分为以下几类：

1. 显式别名（**xplicit Aliases**）
2. 隐式别名（**Implicit Aliases**）
3. 传递隐式别名（**Transitive Implicit Aliases**）

以上三类都需要满足以下条件：

1. 属性类型相同
2. 属性方法必须存在默认值
3. 属性默认值必须相同

否则运行过程中将会出错。

### 显式别名

如果一个注解中的两个成员通过 `@AliasFor`声明后互为别名，那么它们是显式别名
。

显示别名的关系如图所示。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/image-79352e4e.png)


### 隐式别名

如果一个注解中的两个或者更多成员通过`@AliasFor`声明去覆盖同一个元注解的成员值，它们就是隐式别名
。

隐式别名如图所示。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/image-5e255ab2.png)

上图中，`@One` 的 `name` 属性与 `nameAlias` 别名与 `@Two` `nameAlias` 属性。由于 `@One` 注解中并未直接使用 `@AliasFor`,所以与 `@One` 注解隐式别名。


隐式别名类似于数学的等式。可以将其看做以下推导过程。

```
@One.name=@Two.nameAlias
@One.nameAlias=@Two.nameAlias
可以推导出
@One.name=@One.nameAlias
```

### 传递式隐式别名

如果一个注解中的两个或者更多成员通过`@AliasFor`声明去覆盖元注解中的不同成员，但是实际上因为[覆盖的传递性](https://en.wikipedia.org/wiki/Transitive_relation)导致最终覆盖的是元注解中的同一个成员，那么它们就是传递隐式别名。

传递式隐式别名如图所示。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/image-6dcc0259.png)

这种类型涉及了多个注解，`@One#name`别名了 `@Two#nameAlias`属性，然后在 `@One#nameAlias` 属性又别名了 `@Three#nameAliasThree` 属性。然后由于 `@Two#nameAlias`又别名了 `@Three#nameAliasThree` 属性，这就导致 `@One#name` 与  `@One#nameAlias` 间接才生了关系。这种依靠传递性才生别名关系，称为 传递式隐式别名。

隐式别名类似于数学的等式。大家也可以将其用上面等式推导。

## 属性覆盖

属性覆盖指的是注解的一个成员覆盖另一个成员，最后两者成员属性值一致。

属性覆盖可以分为三类:

1. 隐式覆盖（**Implicit Overrides**）
2. 显示覆盖（**Explicit Overrides**）
3. 传递式显式覆盖（**Transitive Explicit Overrides**）

### 隐式覆盖

当一个注解 `@One` 被元注解  `@Two` 标注，两个注解存在同样的属性方法 `name`。`@Two#name` 将会被 `@One#name` 属性覆盖。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/image-4b4938fe.png)

两个看似不来自不同注解的成员 name 指向了同一个成员 name。

### 显示覆盖

显示覆盖就比较简单了，使用 @AliasFor 注解之后，就成为显示覆盖。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/image-b674c38c.png)

### 传递式显式覆盖

如果注解 `@One#name`  显示覆盖了 `@Two#nameAlias`,而 `@Two#nameAlias` 显示覆盖了 `@Three#nameAlias`，最后因为传递性，`@One#name` 实际覆盖了`@Three#nameAlias`。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190713/image-7fe51148.png)


## 总结

Spring 4.2 新增 `@AliasFor`注解，带来一些特性。但是要注意的是仅仅存在 `@AliasFor` 不会执行任何语义别名。

底层原理可以参考 `AnnotationUtils`与 `AnnotatedElementUtils`。


## 帮助文档

1. [Attribute Aliases and Overrides](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model)
2. [注解编程模型~~~~](http://ifeve.com/annotation-programming-model/)
