---
layout: post
categories: Java
title:  原来使用 Spring 实现策略模式可以这么简单！
tagline: by 小黑
published: false
tags: 
  - 小黑
---

Hello，大家好，我是鸭血粉丝~

最近看同事的代码时候，学到了个小技巧，在某些场景下非常挺有用的，这里分享一下给大家。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200923/517e4ac579ca411f925d78cbffff9a6b.png)

Spring 中 `@Autowired`注解，大家应该不会陌生，用过 Spring 的肯定也离不开这个注解，通过这个注解可以帮我们自动注入我们想要的 Bean。

除了这个基本功能之外，`@Autowired` 还有更加强大的功能，还可以注入指定类型的数组，List/Set 集合，甚至还可以是 Map 对象。

比如说当前应用有一个支付接口 `PayService`,分别需要对接支付宝、微信支付、银行卡，所以分别有三个不同实现类 `AliPayService`,`WechatPayservice`,`BankCardPayService`。

四个类的关系如下图所示：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200923/007S8ZIlly1gizrzoxzvgj30yi08gweo.jpg)

如果此时我需要获取当前系统类所有 `PayService`  Bean，老的方式我们只能通过 `BeanFactory`或者 `ApplicationContex`t 获取。

```java
// 首先通过 getBeanNamesForType 获取 PayService 类型所有的 Bean
String[] names = ctx.getBeanNamesForType(PayService.class);
List<PayService> anotherPayService = Lists.newArrayList();
for (String beanName : names) {
    anotherPayService.add(ctx.getBean(beanName, PayService.class));
}
// 或者通过 getBeansOfType 获取所有 PayService 类型
Map<String, PayService> beansOfType = ctx.getBeansOfType(PayService.class);
for (Map.Entry<String, PayService> entry : beansOfType.entrySet()) {
    anotherPayService.add(entry.getValue());
}
```

但是现在我们可以不用这么麻烦了，我们可以直接使用 `@Autowired` 注入 `PayService` Bean 数组，或者 `PayService` List/Set 集合，甚至，我们还可以注入 `PayService`  的 Map 集合。

```java
@Autowired
List<PayService> payServices;

@Autowired
PayService[] payServicesArray;
```

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200923/007S8ZIlly1gj0y2gn75ij30dv09tgn4.jpg)

知道了这个功能，当我们需要使用 Spring 实现策略模式就非常简单。

可能有的小伙伴不太了解策略模式，没关系，这类阿粉介绍一个业务场景，通过这个场景给大家介绍一下策略模式。

还是上面的例子，我们当前系统需要对接微信支付、支付宝、以及银行卡支付。

当接到这个需求，我们首先需要拿到相应接口文档，分析三者的共性。

假设我们这里发现，三者模式比较类似，只是部分传参不一样。

所以我们根据三者的共性，抽象出一组公共的接口 `PayService`，

```java
public interface PayService {
    PayResult epay(PayRequest request);
}
```

然后分别实现三个实现类，都继承这个接口。

那么现在问题来了，由于存在三个实现类，如何选择具体的实现类？

其实这个问题很好解决，请求参数传入一个唯一标识，然后我们根据标识选择相应的实现类。

比如说我们在请求类 `PayRequest` 搞个  `channelNo` 字段，这个代表相应支付渠道唯一标识，比如说支付宝为：**00000001**,微信支付为 **00000002**，银行卡支付为 **00000003**。

接着我们需要把唯一标识与具体实现类一一映射起来，刚好我们可以使用 `Map` 存储这种映射关系。

我们实现一个 `RouteService`，具体代码逻辑如下：

```java
@Service
public class RouteService {

    @Autowired
    Map<String, PayService> payServiceMap;

    public PayResult epay(PayRequest payRequest) {
        PayService payService = payServiceMap.get(payRequest.getChannelNo());
        return  payService.epay(payRequest);
    }

}
```

我们在 `RouteService` 自动注入  `PayService` 所有相关 Bean，然后使用唯一标识查找实现类。

这样我们对外就屏蔽了支付渠道的差异，其他服务类只要调用 `RouteService` 即可。

但是这样实现还是有点小问题，由于我们唯一标识为一串数字，如果像我们上面直接使用  `@Autowired`注入 `Map`,这就需要我们实现类的 `Bean` 名字为 **00000001** 这些。

但是这样命名不是很优雅，这样会让后来同学很难看懂，不好维护。

所以我们需要做个转换，我们可以这么实现。

首先我们改造一下 `PayService` 这个接口，增加一个方法，每个具体实现类通过这个方法返回其唯一标识。

```java
public interface PayService {

    PayResult epay(PayRequest request);

    String channel();
}
```

具体举个支付宝实现类的代码，其他实现类实现类似。

```java
@Service("aliPayService")
public class AliPayService implements PayService {

    @Override
    public PayResult epay(PayRequest request) {
        // 业务逻辑
        return new PayResult();
    }
    @Override
    public String channel() {
        return "00000001";
    }
}
```

最后我们改造一下 `RouteService`，具体逻辑如下：

```java
@Service
public class RouteService {

    @Autowired
    Set<PayService> payServiceSet;
    
    Map<String, PayService> payServiceMap;

    public PayResult epay(PayRequest payRequest) {
        PayService payService = payServiceMap.get(payRequest.getChannelNo());
        return  payService.epay(payRequest);
    }

    @PostConstruct
    public void init() {
        for (PayService payService : payServiceSet) {
            payServiceMap = new HashMap<>();
            payServiceMap.put(payService.channel(), payService);
        }
    }
}
```

上面代码首先通过自动注入 ` PayService` 一个集合，然后我们再将其转为一个 `Map`，这样内部存储刚好是唯一标识与实现类的映射了。

好了，今天的小技巧就分享到这里，学到小伙伴，不妨点个赞吧。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200923/007S8ZIlly1gj0y2j6lvlj308g08hjsu.jpg)