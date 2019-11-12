---
layout: post
title: Java：接口和抽象类，傻傻分不清楚？
tagline: by 沉默王二
categories: java基础
tags:
    - 沉默王二
---

![](https://upload-images.jianshu.io/upload_images/1179389-57dc87b6a2322e67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 01、

来看网络上对接口的一番解释：

>接口（英文：Interface），在 Java 编程语言中是一个抽象类型，是抽象方法的集合。一个类通过继承接口的方式，从而来继承接口的抽象方法。

<!--more-->


兄弟们，你们怎么看，这段解释把我绕得晕乎乎的，好像喝过一斤二锅头。到底是解释抽象类呢还是接口呢？傻傻分不清楚。

搞不清楚要用抽象类还是接口，就先来看看两者之间的区别。来，抽象类和接口，你俩过来比比身高。

1. 抽象类中的方法可以有方法体，能实现方法具体要实现的功能，但是接口中的方法不行，没有方法体。
2. 抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是 `public static final` 类型的，并且是隐式的，缺省的。
3. 接口中不能含有静态代码块以及静态方法(用 `static` 修饰的方法)，而抽象类是可以有静态代码块和静态方法的。
4. 一个类只能继承一个抽象类，而一个类却可以实现多个接口。

### 02、

好像知道了两者之间的区别，但印象还是有些模糊。没关系，我们进一步深入。

**抽象类**

抽象类体现了数据抽象的思想（不然呢），是实现多态的一种机制。抽象类定义了一组抽象的方法，至于这组抽象方法的具体表现形式由子类来继承实现。

抽象类就是用来继承的，否则它就没有存在的任何意义。举个例子，我们来定义一个抽象的作者类。

```java
abstract class Author {
	abstract void write ();
	
	public void sleep () {
		System.out.println("吃饭睡觉打豆豆");
	}
}
```

作为一名作者，本职工作就是搞写作的，其他时间就吃饭睡觉打豆豆；但至于能写出什么样的作品，就要看是哪一个作者了。比如说，沉默王二能写出的作品一定是幽默风趣的。

```java
public class Wanger extends Author {

	@Override
	void write() {
		System.out.println("沉默王二的作品《Web 全栈开发进阶之路》，读起来轻松惬意");
	}
	
}
```

注意到了没？抽象类是可以有自己的方法的，但继承它的子类可以忽视。

**接口**

接口是一种比抽象类更加抽象的“类”，毕竟是用关键字 `interface` 声明的，不是用 `class`。

接口只是一种形式，就好像一纸契约，自身不能做任何事情。但只要某个类实现了这个接口，就必须按照这纸契约来办事：接口里提到的方法必须全部实现，少一个都不行（抽象类的子类可以忽视非抽象方法）。举个例子，我们来定义一个北航出版合同的接口。

```java
interface ContractBeihang {
	void scriptBeihang();
}
```

一旦作者签订了合同，那么就必须定期完成一定量的书稿。

```java
public class Wanger extends Author implements ContractBeihang {

	@Override
	void write() {
		System.out.println("作品《Web 全栈开发进阶之路》，读起来轻松惬意的技术书");
	}

	@Override
	public void scriptBeihang() {
		System.out.println("一年内完成书稿啊，不然要交违约金的哦。");
	}
	
}
```

接口是抽象类的补充，Java 为了保证数据的安全性不允许多重继承，也就是说一个类同时只允许继承一个父类（为什么呢？请搜索关键字“菱形问题”）。

但是接口不同，一个类可以同时实现多个接口，这些接口之间可以没有多大的关系（弥补了抽象类不能多重继承的缺陷）。比如说，沉默王二不仅签了北航出版社的合同，还和 51CTO 签了付费课程的合同。

```java
public class Wanger extends Author implements ContractBeihang, Contract51 {

	@Override
	void write() {
		System.out.println("作品《Web 全栈开发进阶之路》，读起来轻松惬意的技术书");
	}

	@Override
	public void scriptBeihang() {
		System.out.println("一年内完成书稿啊，不然要交违约金的哦。");
	}

	@Override
	public void script51() {
		System.out.println("王老师，先把 Java 云盘的大纲整理出来。");
	}
	
}
```

### 03、

通过上面举的例子，是不是对接口和抽象类有比较清晰的认知了？如果还没有，来来来，我们再来比较一下接口和抽象类之间的差别。

![](https://upload-images.jianshu.io/upload_images/1179389-e1837c25fd88d254.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

究竟什么时候使用接口，什么时候使用抽象类呢？

1、抽象类表示了一种“is-a”的关系，而接口表示的是“like-a”的关系。也就是说，如果 B 类是 A（沉默王二是一个作者），则 A 应该用抽象类。如果 B 类只是和 A 有某种关系，则 A 应该用接口。

2、 如果要拥有自己的成员变量和非抽象方法，则用抽象类。接口只能存在静态的不可变的成员变量（不过一般都不在接口中定义成员变量）。

3、为接口添加任何方法（抽象的），相应的所有实现了这个接口的类，也必须实现新增的方法，否则会出现编译错误。对于抽象类，如果添加了非抽象方法，其子类却可以坐享其成，完全不必担心编译会出问题。

4、抽象类和接口有很大的相似性，请谨慎判断。Java 从1.8版本开始，尝试向接口中引入了默认方法和静态方法，以此来减少抽象类和接口之间的差异。换句话说，两者之间越来越难区分了。

### 04、

在实际的开发应用当中，抽象类我用得不多（这可真是大实话）；接口我倒是用得蛮多的，就像下面这样子：

```java
public interface CityMapper {

	@Select("select * from city")
	List<City> getCitys();

}
```


`@Insert`、`@Update`、`@Delete`、`@Select` 被称为 `Mybatis` 的注射器注解。

是不是突然感觉有点懵？之前还在谈接口和抽象类，怎么一下子跳跃到 `Mybatis` 上面了呢？还有什么映射器注解？

嗯，这就对了。所有的理论知识都要应用于实践，否则也就没有了存在价值。在我的实践应用当中，接口用得最多的就是 `Mybatis` 的 `Mapper` 接口。

`MyBatis` 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。`MyBatis` 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。`MyBatis` 可以使用简单的 XML 或注解（就是你在前面见到的增删改查四大注解）来配置和映射原生类型、接口和 `Java` 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

当我们配置好了 `MyBatis` 环境后，可以直接通过以下语句来调用注射器接口。

```java
@Service
public class CityService {
	@Autowired
	private CityMapper cityMapper;

	public void init() {
			List<City> citys = cityMapper.getCitys();
		}
	}
}
```

在注射器接口中，也只会存在那些与数据库查询相关的抽象方法，就像你看到的 `List<City> getCitys();`。一个注射器接口 + 注射器注解就可以增删改查数据库，是不是感觉很神奇？

### 05、

这篇文章的目的是帮助更多的读者了解和掌握抽象类、接口的特点，以及不同的使用场景，通过我整篇文章的努力，我相信你一定若有所获——这也是我写作的最强动力。最后，感谢各位的阅读哦。
