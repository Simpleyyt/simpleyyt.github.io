---
layout: post
categories: Java
title: 一文教你如何通过 Stream API 批量 Mock 数据
tagline: by 子悠
tags: 
  - 子悠
title: 后端程序员对于 Docker 要掌握多少才行？阿粉的答案是...
在日常开发的过程中我们经常会遇到需要 `mock` 一些数据的场景，比如说 `mock` 一些接口的返回或者说 `mock` 一些测试消息用于队列生产者发送消息，可能很多时候我们都是使用一些固定的 `case` 或者一条相同的数据重复使用。今天阿粉就教大家用 `Stream` 去构造一些伪真实的一些数据。

<!--more-->

## Mock 任意个 UUID

首先我们通过普通写法来构造 100 个 `UUID`，代码如下相信大家都会写，就不多说了。

```java
  public static List<UUID> listUUID(int size) {
    List<UUID> list = new ArrayList<>();
    for (int i = 0; i < size; i++) {
      UUID uuid = UUID.randomUUID();
      list.add(uuid);
    }
    return list;
  }
```

下面再提供 `Stream` 的写法，代码如下，一行搞定

```java
  public static List<UUID> listUUID2(int size) {
    return Stream.generate(UUID::randomUUID).limit(size).collect(Collectors.toList());
  }
```

这里我们使用了 `Stream` 的 `generate` 方法，该方法接收一个 `Supplier` 类型的参数，`Supplier` 是一个功能接口，只有一个 `get` 方法，返回一个对象，不接收任何参数，上面我们就是通过 `UUID` 静态引用的方式获得一个 `UUID` 对象，另外我们使用 `limit` 方法来进行截断只获取 100 个。

## Mock 消息

接下来我们再使用 `Stream API` 批量构造一批消息，作为队列的生产者进行数据发送

定义消息体
```java
package com.example.demo.dto;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author Java 极客技术<br>
 * <b>Date：</b>2022-09-03 11:49<br>
 * <b>Desc：</b>无<br>
 */
public class Message {
  int id;
  String message;

  public Message(int id, String message) {
    this.id = id;
    this.message = message;
  }

  @Override
  public String toString() {
    return "Message{" +
      "id=" + id +
      ", message='" + message + '\'' +
      '}';
  }
}

```

测试代码

```java
  public static void main(String[] args) {
    List<Message> messages = genMessage(10);
    messages.forEach(System.out::println);
  }

  public static List<Message> genMessage(int size) {
    AtomicInteger atomicInteger = new AtomicInteger();
    Supplier<Message> supplier = () -> {
      Message message = new Message(new Random().nextInt(), "Message : " + atomicInteger.getAndIncrement());
      System.out.println("inner:" + message.toString());
      return message;
    };
    System.out.println(99);
    return Stream.generate(supplier).limit(size).collect(Collectors.toList());
  }
```

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h5tk3opylpj219a0r078t.jpg)

先看下运行结果，我们再来分析，可以看到第一个 `case` 我们是使用静态引用来返回一个 `UUID` 对象，这个 `case` 我们通过创建 `lambda` 表达式的形式来实现一个 `Supplier`，在表达式中我们进行 `message` 对象的构造，然后进行返回。其实上文的静态引用，本质上也是一个 `lambda`，所以跟下面的实现是一个原理，只不过是一些语法糖而已。

```java
  public static List<UUID> listUUID2(int size) {
    Supplier<UUID> supplier = () -> UUID.randomUUID();
    return Stream.generate(supplier).limit(size).collect(Collectors.toList());
  }
```

如果对 `Stream` 流有理解的可以看到，我们这里有两个点需要注意，一个是我们这里的输出 99 是在` inner` 之前的，另一个是我们这里使用的 `limit` 方法，不然会一直进行输出不会停止的，这两点其实都是流的基本特性，就不多说了。

## Supplier 是个啥

上文提到 `Stream`  的 `generate` 方法接收的是一个 `Supplier` 类型的参数，那么这个 `Supplier`  是个啥呢？我们来仔细看一下。

```java
package java.util.function;

@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}

```

通过代码我们可以看到首先 `Supplier` 是个接口，既然是接口那就可以进行具体的实现，并且这个接口只有一个方法 `get` 返回指定的类型，同时该接口还有一个` @FunctionalInterface` 注解，表名这个接口是一个函数是编程的接口，函数式接口是指仅仅只包含一个抽象方法的接口。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h5tl4lhvkpj210v0u0wko.jpg)

我们看到这个注解的 `javadoc` 里面大概的意思是这个注解是用来标识一个函数接口，函数式接口只有一个抽象方法，但是如果有 `default` 方法或者覆盖了 `Object` 的 `public` 方法都不算是抽象方法。还有一句讲的是函数式接口可以通过 `lambda` 表达式，方法引用或者构造方法引用来创建。我们上面的两个例子演示了 `lambda` 表达式和方法引用，构造函数其实也一样。

所以总结来说  `Stream`  的 `generate` 方法通过接收一个 `Supplier`  类型的参数来创建一个数据流，得到数据流以后就可以进行各种流的操作了。我们这篇文章更多的是通过 `Stream` 来构造 `mock` 数据，创建一个流，对于流的各种操作就不在本文的讨论范围之内了，阿粉之前也有相应的文章介绍过 `Stream` 感兴趣的小伙伴可以去翻翻看。

## 总结

工作中 `mock` 数据在很多场景都会遇到，但是可能很多时候我们都不会太关注 `mock` 的数据的形式，虽然说一个循环也可以 `mock` 到相应的数据，但是能写的优雅一点为什么我们不写的优雅一点呢？
