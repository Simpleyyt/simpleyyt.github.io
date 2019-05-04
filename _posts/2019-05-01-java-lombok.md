---
layout: post
title: 你干啥的？Lombok
tagline: by 沉默王二
categories: java
tag:
    - Java Lombok
---

![](https://upload-images.jianshu.io/upload_images/1179389-168fc10b5d7c16fe.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 01、Lombok 的自我介绍

`Lombok` 在官网是这样作自我介绍的：

>Project Lombok makes java a spicier language by adding 'handlers' that know how to build and compile simple, boilerplate-free, not-quite-java code.

说实话，我英文不太好（不是找借口，真的），但借助金山词霸，大致知道了这段英文的意思：`Lombok` 是个好类库，可以为 Java 代码添加一些“处理程序”，让其变得更简洁、更优雅。

<!--more-->

据我已有的经验来看，`Lombok` 最大的好处就在于通过注解的形式来简化 Java 代码，简化到什么程度呢？

我相信你一定写过不少的 `getter / setter`，尽管可以借助 IDE 来自动生成，可一旦 `Javabean` 的属性很多，就免不了要产生大量的 `getter / setter`，这会让代码看起来不够简练，就像老太婆的裹脚布一样，又臭又长。

```java
class Cmower {
	private int age;
	private String name;
	private BigDecimal money;

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public BigDecimal getMoney() {
		return money;
	}

	public void setMoney(BigDecimal money) {
		this.money = money;
	}
}
```

`Lombok` 可以通过注解的方式，在编译的时候自动为 `Javabean` 的属性生成 `getter / setter`，不仅如此，还可以生成构造方法、`equals`、`hashCode`，以及 `toString`。注意是在编译的时候哦，源码当中是没有 `getter / setter` 等等的。

```java
@Getter
@Setter
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;
}
```

哎呀，源码看起来苗条多了，对不对？

### 02、添加 Lombok 的依赖

如果项目使用 `Maven` 构建的话，添加`Lombok` 的依赖就变得轻而易举了。

```xml
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.18.6</version>
	<scope>provided</scope>
</dependency>
```

其中 `scope=provided`，就说明 `Lombok` 只在编译阶段生效。也就是说，`Lombok` 会在编译期静悄悄地将带 `Lombok` 注解的源码文件正确编译为完整的 `class` 文件。

温馨提示：只在项目中追加 `Lombok` 的依赖还不够，还要为 IDE 添加 `Lombok` 支持，否则 `Javabean` 的 `getter / setter` 就无法自动编译，也就不能被调用。

### 03、为 Eclipse 添加 Lombok 支持

第一步，下载 `Lombok` 的 jar 包。下载地址如下：

http://central.maven.org/maven2/org/projectlombok/lombok/1.18.6/lombok-1.18.6.jar

第二步，双击运行该 jar 包。

![](https://upload-images.jianshu.io/upload_images/1179389-d46b5d1cc00f46a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三步，点击「Install / Update」进行安装。

![](https://upload-images.jianshu.io/upload_images/1179389-0e06edb7a7c2b03d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第四步，重启 Eclipse，完成项目的重新编译。

可以通过 Outline 视图查看已经编译好的 `getter / setter`。是不是感觉很奇妙？

![](https://upload-images.jianshu.io/upload_images/1179389-6555a30055b9a6bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候，我们就可以使用 `Lombok` 注解过的 `Javabean` 了。

![](https://upload-images.jianshu.io/upload_images/1179389-6049323cd3b9d771.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 04、使用 Jad 查看 Lombok 字节码

曾经有一段时间，每个人选择的反编译工具都是 `Jad`。虽然 `Jad` 已经死了，不再更新了，但仍然有许多人需要它。比如说我就是其中一个。甚至在我的心目中，`Jad` 是最佳的 Java 反编译工具，排名在 `JD-GUI` 之前。

Jad 的下载地址如下，包含各种平台的版本：
[http://www.javadecompilers.com/jad](http://www.javadecompilers.com/jad)

下载完成后解压，并不需要任何的安装步骤。怎么使用 Jad 呢？

```cmd
jad CmowerLombok.class
// Parsing CmowerLombok.class... Generating CmowerLombok.jad
```

执行完以上命令后，会生成一个新的文件，后缀为 .jad，使用文本编辑器打开后，内容如下：

```java
// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3) 
// Source File Name:   CmowerLombok.java

package com.cmower.java_demo.lombok;

import java.math.BigDecimal;

class CmowerLombok
{

    CmowerLombok()
    {
    }

    public int getAge()
    {
        return age;
    }

    public String getName()
    {
        return name;
    }

    public BigDecimal getMoney()
    {
        return money;
    }

    public void setAge(int age)
    {
        this.age = age;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public void setMoney(BigDecimal money)
    {
        this.money = money;
    }

    private int age;
    private String name;
    private BigDecimal money;
}
```

嘿嘿，果然 `getter / setter` 就在里面，这真是一件令人开心的事情，开心得我一巴掌拍在桌子上，差一点没把手拍骨折，也不知道桌子疼不疼。

很早就有朋友劝我使用 `Lombok`，但我总觉得增加一个能够产生任何现代 IDE 都能轻易产生的代码的类库没有多大的价值（句子有点长，注意断句）。现在我再也不会这么觉得了，`Lombok` 为我节省了大量的生成样板代码的时间。

PS：需要注明一点的是，我首次查看 class 文件的时候遇到了巨坑，`getter / setter` 竟然不在其中，但是可以调用。试了很多的反编译工具都不行。

于是我不得不在很多个群里面发起了咨询，很多大神都请教了，最后的结论是 Eclipse 的 Lombok 插件可能出了 bug。为此，我还下载了程序员开发利器——IntelliJ IDEA，但我用起来蹩手蹩脚，最后还是放弃了。

折腾了大概 3 个多小时候后，没办法，我只得重启了 Eclipse（解决编译问题的终极杀招），class 文件中莫名其妙地又出现了 `getter / setter`（还记得我拍桌子的兴奋劲吗？）。由此我得出的结论是，不管别人有没有写 `Lombok` 的教程，自己一定要亲身实践一番。

### 05、使用其他反编译工具查看 Lombok 字节码

既然说到反编译工具，我觉得有必要介绍另外一款优秀的反编译工具——`Enhanced Class Decompiler`。

它将 `JD`、`Jad`、`FernFlow`、`CFR`、`Procyon` 与 `Eclipse` 无缝集成，并且允许 Java 开发人员直接调试类文件而不需要源代码。这还不算完啊，它还集成了 Eclipse 类编辑器 M2E 插件，支持 Javadoc、参考搜索、库源附加、字节码视图和 `JDK 8 lambda` 表达式的语法。

总之，`Enhanced Class Decompiler` 要取代 Jad 在我心目中的位置了。

第一步，在 `Eclipse Marketplace` 搜索 jad。

![](https://upload-images.jianshu.io/upload_images/1179389-6ee5f54f02444228.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二步，直接点击「Installed」按钮进行安装。安装完成后重启 Eclipse。

第三步，右键 target，选择「Show In Remote Systems View」。

![](https://upload-images.jianshu.io/upload_images/1179389-233b213249673fe5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第四步，右键需要查看的 class 文件，依次选择「Open With」→「Class Decompiler Viewer」。

![](https://upload-images.jianshu.io/upload_images/1179389-9524cba03abeb9e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看到反编译后的代码如下所示。

```java
package com.cmower.java_demo.lombok;

import java.math.BigDecimal;

class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;

	public int getAge() {
		return this.age;
	}

	public String getName() {
		return this.name;
	}

	public BigDecimal getMoney() {
		return this.money;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public void setName(String name) {
		this.name = name;
	}

	public void setMoney(BigDecimal money) {
		this.money = money;
	}
}
```

PS：如果你想把 `Enhanced Class Decompiler` 设置为默认的 class 文件视图的话，可以参照下图哦。

![](https://upload-images.jianshu.io/upload_images/1179389-20bf79699811bb11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 06、常用的 Lombok 注解

1）**@Getter / @Setter**

`@Getter / @Setter` 用起来很灵活，比如说像下面这样：

```java
class CmowerLombok {
	@Getter @Setter private int age;
	@Getter private String name;
	@Setter private BigDecimal money;
}
```

字节码文件反编译后的内容是：

```java
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;

	public int getAge() {
		return this.age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public String getName() {
		return this.name;
	}

	public void setMoney(BigDecimal money) {
		this.money = money;
	}
}
```

2）**@ToString**

打印日志的好帮手哦。

```java
@ToString
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;
}
```

字节码文件反编译后的内容是：

```java
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;

	public String toString() {
		return "CmowerLombok(age=" + this.age + ", name=" + this.name + ", money=" + this.money + ")";
	}
}
```

3）**val**

在编写 JavaScript 代码时，我一直觉得 `var` 这个变量声明类型用起来特别方便。`Lombok` 也提供了一个类似的。

```java
class CmowerLombok {
	public void test() {
		val names = new ArrayList<String>();
		names.add("沉默王二");
		val name = names.get(0);
		System.out.println(name);
	}
}
```

字节码文件反编译后的内容是：

```java
class CmowerLombok {
	public void test() {
		ArrayList<String> names = new ArrayList();
		names.add("沉默王二");
		String name = (String) names.get(0);
		System.out.println(name);
	}
}
```

4）**@Data**

@Data 注解可以生成 `getter / setter`、`equals`、`hashCode`，以及 `toString`，是个总和的选项。

```java
@Data
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;
}
```

字节码文件反编译后的内容是：

```java
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;

	public int getAge() {
		return this.age;
	}

	public String getName() {
		return this.name;
	}

	public BigDecimal getMoney() {
		return this.money;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public void setName(String name) {
		this.name = name;
	}

	public void setMoney(BigDecimal money) {
		this.money = money;
	}

	public boolean equals(Object o) {
		if (o == this) {
			return true;
		} else if (!(o instanceof CmowerLombok)) {
			return false;
		} else {
			CmowerLombok other = (CmowerLombok) o;
			if (!other.canEqual(this)) {
				return false;
			} else if (this.getAge() != other.getAge()) {
				return false;
			} else {
				Object this$name = this.getName();
				Object other$name = other.getName();
				if (this$name == null) {
					if (other$name != null) {
						return false;
					}
				} else if (!this$name.equals(other$name)) {
					return false;
				}

				Object this$money = this.getMoney();
				Object other$money = other.getMoney();
				if (this$money == null) {
					if (other$money != null) {
						return false;
					}
				} else if (!this$money.equals(other$money)) {
					return false;
				}

				return true;
			}
		}
	}

	protected boolean canEqual(Object other) {
		return other instanceof CmowerLombok;
	}

	public int hashCode() {
		int PRIME = true;
		int result = 1;
		int result = result * 59 + this.getAge();
		Object $name = this.getName();
		result = result * 59 + ($name == null ? 43 : $name.hashCode());
		Object $money = this.getMoney();
		result = result * 59 + ($money == null ? 43 : $money.hashCode());
		return result;
	}

	public String toString() {
		return "CmowerLombok(age=" + this.getAge() + ", name=" + this.getName() + ", money=" + this.getMoney() + ")";
	}
}
```

5）更多的 `Lombok` 注解，待你解锁哦。

### 07、Lombok 的处理流程

一图胜千言，我就先上图了。

![](https://upload-images.jianshu.io/upload_images/1179389-1db057ef5f229da6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- javac 对源代码进行分析，生成一棵抽象语法树（AST）

- javac 编译过程中调用实现了JSR 269 的 Lombok 程序

- Lombok 对 AST 进行处理，找到 Lombok 注解所在类对应的语法树（AST），然后修改该语法树，增加 Lombok 注解定义的相应树节点（所谓代码）

- javac 使用修改后的抽象语法树生成字节码文件
