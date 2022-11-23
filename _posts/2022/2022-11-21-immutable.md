---
layout: post
categories: Java
title: Java 中如何实现一个像 String 一样不可变的类？
tagline: by 子悠
tags: 
  - 子悠

---

如果问你在日常开发中用到的最多的一个 `Java` 类是什么，阿粉敢打赌绝对是 `String.class`。说到 `String` 大家都知道 `String` 是一个不可变的类；虽然用的很多，那不知道小伙伴们有没有想过怎么样创建一个自己的不可变的类呢？这篇文章阿粉就带大家来实践一下，创建一个自己的不可变的类。

<!--more-->

## 特性

在手动编写代码之前，我们先了解一下不可变类都有哪些特性，

1. 定义类的时候需要使用 `final` 关键字进行修饰：之所以使用 `final` 进行修饰是因为这样可以避免被其他类继承，一旦有了子类继承就会破坏父类的不可变性机制；
2. 成员变量需要使用 `fina`l 关键词修饰，并且需要是 `private` 的：避免属性被外部修改；
3. 成员变量不可提供 `setter` 方法，只能提供 `getter` 方法：避免被外部修改，并且避免返回成员变量本身；
4. 提供所有字段的构造函数；

## 实操

知道了不可变类的一些基本特性之后，我们来实际写代码操作一下，以及我们会验证一下，如果不按照上面的要求来编写的话，会出现什么样的问题。

这里我们定义一个 `Teacher` 类来测试一下，按照我们上面提到的几点，我们给类和属性的定义都加上 `final` 代码如下所示。

```java
package com.example.demo.immutable;

import java.util.List;
import java.util.Map;

public final class Teacher {
  private final String name;
  private final List<String> students;
  private final Address address;
  private final Map<String, String> metadata;

  public Teacher(String name, List<String> students, Address address, Map<String, String> metadata) {
    this.name = name;
    this.students = students;
    this.address = address;
    this.metadata = metadata;
  }

  public String getName() {
    return name;
  }


  public List<String> getStudents() {
    return students;
  }

  public Address getAddress() {
    return address;
  }

  public Map<String, String> getMetadata() {
    return metadata;
  }
}

```

```java
package com.example.demo.immutable;

public class Address {
  private String country;
  private String city;

  public String getCountry() {
    return country;
  }

  public void setCountry(String country) {
    this.country = country;
  }

  public String getCity() {
    return city;
  }

  public void setCity(String city) {
    this.city = city;
  }
}

```

我们思考一下，上面的代码是否真正的做到了不可变，好，我们思考三秒钟，心里默默的数三下。为了回答这个问题，我们看下下面的测试代码。

```java
package com.example.demo;

import com.example.demo.immutable.Address;
import com.example.demo.immutable.Teacher;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author Silence<br>
 * <b>Date：</b>2022-11-22 21:17<br>
 * <b>Desc：</b>无<br>
 */
public class ImmutableDemo {

  public static void main(String[] args) {
    List<String> students = new ArrayList<>();
    students.add("鸭血粉丝 1");
    students.add("鸭血粉丝 2");
    students.add("鸭血粉丝 3");
    Address address = new Address();
    address.setCountry("中国");
    address.setCity("深圳");
    Map<String, String> metadata = new HashMap<>();
    metadata.put("hobby", "篮球");
    metadata.put("age", "29");
    Teacher teacher = new Teacher("Java极客技术", students, address, metadata);
    System.out.println(teacher.getStudents().size());
    System.out.println(teacher.getMetadata().size());
    System.out.println(teacher.getAddress().getCity());

    // 修改属性
    teacher.getStudents().add("小明");
    teacher.getMetadata().put("weight", "120");
    teacher.getAddress().setCity("广州");

    System.out.println(teacher.getStudents().size());
    System.out.println(teacher.getMetadata().size());
    System.out.println(teacher.getAddress().getCity());
  }

}

```

运行的结果如下截图所示，通过测试我们可以发现，简单的只添加 `final` 关键字是不能解决不可变性的，我们当前的 `teacher` 实例已经被外层修改掉了成员变量。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8e97nshnoj31420u0jw5.jpg)

为了解决这个问题，我们还需要对我们的 `Teacher` 类进行改造，首先我们可以想到的就是需要将 `students` 和 `metadata` 两个成员变量不能直接返回给外层，否则外层的修改会直接影响到我们的不可变类，那么我们就可以修改 `getter` 方法，拷贝一下成员变量进行返回，而不是直接返回，修改代码如下

```java
  public List<String> getStudents() {
    return new ArrayList<>(students);
    //return students;
  }
    public Map<String, String> getMetadata() {
    return new HashMap<>(metadata);
		//return metadata;
  }
```

我们再次运行上面的测试代码，可以看到这次的返回数据如下，这次我们的 `students` 和 `metadate` 成员变量并没有被外层修改掉了。但是我们的 `address` 成员变量还是有问题，没关系，我们接着往下看。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8e98d3qpsj317t0u043k.jpg)

很自然的为了解决 `address` 的问题，我们想到了也是进行一个拷贝，再调用 `getter` 方法的时候返回一个拷贝对象，而不是直接返回成员变量。那我们就需要改造 `Address` 类，将其变成 `Cloneable` 的即可，我们实现 接口，然后覆盖一个 `clone` 方法，代码如下

```java
package com.example.demo.immutable;

public class Address implements Cloneable{
  ...// 省略
  @Override
  public Address clone() {
    try {
      return (Address) super.clone();
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}

```

再修改 `Teacher` 的 `getAddress` 方法

```java
  public Address getAddress() {
		//return address;
    return address.clone();
  }
```

接下来我们再运行一下测试代码，结果如下，可以看到这次我们的 `teacher` 实例的成员变量并没有被修改掉了，至此我们完成了一个不可变对象的创建！

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8e9c2rxlcj315z0u0jwi.jpg)

## String 的实现

前面我们看的是自定义实现不可变类的操作，接下来我们简单看一下 `String` 类是如何实现不可变的，通过源码我们可以看到 `String` 也使用了关键字 `final` 来避免被子类继承，以及对应存放具体值的成员变量也使用了 `final` 关键字。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8e9l1tchfj31ba0u0q7z.jpg)

并且对外提供的方法 `substring` 也是通过复制的形式对外提供的新的 `String` 对象。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8e9mo2rccj31pk0h6q63.jpg)

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8e9n02xxqj31o60k8dip.jpg)

> 注意阿粉这里的 JDK 版本是 19 所以可能大家版本不一致具体的实现不太一样，但是本质上都是一样的。
