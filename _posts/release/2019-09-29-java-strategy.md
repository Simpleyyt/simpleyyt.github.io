---
layout: post
title: 设计模式之策略模式
tagline: by 小九
categories: java
tags: 
  - strategy
---

看了这篇文章，希望你也能写出优雅的代码

<!--more-->

### 01、简介

策略模式指的是对象具备某个行为，但是在不同的场景中，该行为有不同的实现算法。

策略模式定义了一组算法，将每个算法都封装起来，并且使它们之间可以互换。

### 02、应用场景

1. 假如系统中有很多类，而他们的区别仅仅在于他们的行为不同。
2. 一个系统需要动态地在几种算法中选择一种。

### 03、用策略模式实现选择促销活动的业务场景

这里先声明一下这个场景，现在很火的电商平台，经常会出一些活动，比如返现，优惠券，买一送一等等活动，我们用代码来模拟一下这些场景。

首先创建抽象策略角色 `Promotion` 接口：

```java
public interface Promotion {
    void execute();
}
```

然后是具体的策略角色，首先来个默认的活动 `DefaultActivity` 类：

```java
public class DefaultActivity implements Promotion {

    @Override
    public void execute() {
        System.out.println("不享受优惠,商品金额和数量不变");
    }
}
```

买一送一活动 `BuyOneGetTwo` 类:

```java
public class BuyOneGetTwo implements Promotion {

    @Override
    public void execute() {
        System.out.println("买一送一,商品数量加1");
    }
}
```

最后返现活动 `ReturnCash` 类：

```java
public class ReturnCash implements Promotion {

    @Override
    public void execute() {
        System.out.println("返现5元,实际金额=商品总金额-5");
    }
}
```

好了，准备工作做好了，现在来看下怎么用策略模式来实现。

首先我们建个 `IEnum` 接口：

```java
public interface IEnum {
    String getCode();

    String getDesc();
}
```

然后是一个枚举类 `PromotionEnum` ：

```java
public enum PromotionEnum implements IEnum {
    DEFAULT("DEFAULT","默认没有活动"),
    BUY_ONE_GET_TWO("BUYONEGETTWO","买一送一活动"),
    RETURN_CASH("RETURNCASH","返现活动");

    private String code;

    private String desc;

    PromotionEnum(String code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    @Override
    public String getCode() {
        return code;
    }

    @Override
    public String getDesc() {
        return desc;
    }
}
```

最后策略的实现类 `PromotionStrategyFactory` ，这个类我用了单例模式和工厂模式来实现：

```java
public class PromotionStrategyFactory {

    private static Map<String, Promotion> PROMOTION_STRATEGY_MAP = new HashMap<>();
    static {
        //存放了具体促销活动类的实例
        PROMOTION_STRATEGY_MAP.put(PromotionEnum.DEFAULT.getCode(),new DefaultActivity());
        PROMOTION_STRATEGY_MAP.put(PromotionEnum.RETURN_CASH.getCode(),new ReturnCash());
        PROMOTION_STRATEGY_MAP.put(PromotionEnum.BUY_ONE_GET_TWO.getCode(),new BuyOneGetTwo());
    }
	//单例模式,私有的构造方法
    private PromotionStrategyFactory(){}
	//提供获取实例的方法
    public static Promotion getPromotion(String promotionKey){
        Promotion promotion = PROMOTION_STRATEGY_MAP.get(promotionKey);
        return promotion == null ? PROMOTION_STRATEGY_MAP.get(PromotionEnum.DEFAULT.getCode()) : promotion;
    }
}
```

代码到这里就已经写完了，现在来看下测试类，首先看下不用策略模式是怎么用的。

```java
public class Test {

    public static void main(String[] args) {
        //实际的业务场景,这个promotionKey是客户端传过来
        String promotionKey = "BUYONEGETTWO";
        if(promotionKey.equals("BUYONEGETTWO")){
            //如果是买一送一活动
            new BuyOneGetTwo().execute();
        }else if(promotionKey.equals("BUYONEGETTWO")){
            //如果是返现活动
            new ReturnCash().execute();
        }else{
            //没有活动
            new DefaultActivity().execute();
        }
    }
}
```

输出结果：

```java
买一送一,商品数量加1
```

看这段代码，如果活动有很多，那么我们的 `if` 判断就会很多，代码的可读性很差，同时也不利于扩展。

现在我们来看下策略模式是怎么做的。

```java
public class Test {

    public static void main(String[] args) {
        //买一送一活动
        String promotionKey = "BUYONEGETTWO";
        Promotion promotion = PromotionStrategyFactory.getPromotion(promotionKey);
        promotion.execute();
        //返现活动
        String promotionKey1 = "RETURNCASH";
        Promotion promotion1 = PromotionStrategyFactory.getPromotion(promotionKey1);
        promotion1.execute();
        //没有活动
        String promotionKey2 = "";
        Promotion promotion2 = PromotionStrategyFactory.getPromotion(promotionKey2);
        promotion2.execute();
    }
}
```

输出结果：

```java
买一送一,商品数量加1
返现5元,实际金额=商品总金额-5
不享受优惠,商品金额和数量不变
```

结果一目了然，只用了最多两句代码就达到了我们的目的。

### 04、策略模式在 JDK 源码中的体现

首先来看一个比较常用的比较器 Comparator 接口，我们看到的一个大家常用的
compare()方法，就是一个策略抽象实现：

```java
public interface Comparator<T> {
	int compare(T o1, T o2);
	...
}
```

Comparator 抽象下面有非常多的实现类，我们经常会把 Comparator 作为参数传入作
为排序策略，例如 Arrays 类的 parallelSort 方法等：

```java
public class Arrays {
    ...
    public static <T> void parallelSort(T[] a, int fromIndex, int toIndex,
    Comparator<? super T> cmp) {
    ...
    }
    ...
}
```

### 05、策略模式的优缺点

优点：

1. 策略模式符合开闭原则。
2. 避免使用多重条件转移语句，如 if...else...语句、switch 语句
3. 使用策略模式可以提高算法的保密性和安全性。

缺点：

1. 客户端必须知道所有的策略，并且自行决定使用哪一个策略类。
2. 代码中会产生非常多策略类，增加维护难度。

