---
layout: post
categories: springboot
title: 灵魂拷问，如何防止重复提交？
tags: 
  - 炸鸡可乐
---

平时开发项目的时候，你是否遇到这样的困惑，用户不停的点击按钮向后端提交数据，而你却束手无策！

<!--more-->

### 一、故事
记得以前面试的时候，面试官抛出来这么一个问题，**就是后端如何防止重复提交订单**？

当时的我刚工作一年多，工作经历也不是很丰富，脑子里第一个想到的就是，这个前端就可以解决吧，然后面试官说必须要在后台处理这个问题，之后这场面试也就凉了。

面试结束之后，就开始百度查询资料，除了广告占头条比较吸引人以外，也没找到啥可行的答案，然后请教各路大佬之后，终算是有了一个比较可靠的解决方案。（后文会详细分享）

前些天在群里也看到有个朋友在讨论这个问题，这让我也想起了之前的那段经历，今天小编就和大家一起来讨论一下**如何防止重复提交**这个问题！

### 二、问题场景
**重复提交**，从名字上看，顾名思义，就是多次提交数据，例如支付的时候，假如同一笔订单多次支付，就会造成多次扣款，其后果可想而知！

像这样的案例比比皆是，如果将场景进行归纳，我们会发现主要有两类：

* 第一类：由于用户误操作或者网络卡顿，可能会造成多次点击表单提交按钮或者刷新提交页面，就会造成重复提交；
* 第二类：黑客或恶意用户使用postman、jmeter等工具重复恶意提交表单，攻击网站，从而造成重复提交；

这两类严重的时候，甚至会直接造成系统宕机！
### 三、解决方案
说了这么多，那如何防止重复提交数据呢？

毫无疑问，肯定是从前端、后端同时入手！

#### 3.1、前端解决方法
通过 JavaScript 来屏蔽提交按钮，当用户点击提交按钮后，屏幕弹出遮罩层**提示数据加载中....**！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/submit-token-data/01.gif)

直到后端返回结果或者前端请求超时时，再将其遮罩层关闭，从而实现防止表单重复提交！

#### 3.2、后端解决方法
虽然前端通过屏蔽操作按钮，防止用户重复提交数据，但是如果黑客直接绕过前端给后端提交数据时，那么后端肯定也必须要做防止重复提交的验证。
##### 方案一：给数据库增加唯一键约束（不推荐）
起初，最开始想到的就是，在控制层给数据做验证，例如用户注册，当用户手机号或者邮箱已经存在，则直接提示提交失败。
```java
@RequestMapping(value = "/register")
public boolean register(@RequestBody UserDto userDto) throws Exception {
    //检查邮件是否已经注册
    QueryWrapper<User> queryWrapper = new QueryWrapper();
    queryWrapper.eq("user_email",userDto.getUserEmail());
    User dbUser = userService.getOne(queryWrapper);
    if(dbUser ! = null){
        throw new CommonExecption("当前邮箱已被注册，请使用新的邮箱注册或者通过密码找回操作！");
    }
    return userService.insert(userDto);
}
```
如果想更加安全一点，可以**在数据库中给关键字段增加唯一键约束**，如果用户邮箱已经插入到数据库，会直接抛异常，提示当前邮箱已经注册！
```java
try {
    userService.insert(userDto);
} catch (Exception e) {
    log.error("用户插入失败",e);
    throw new CommonExecption("当前邮箱已被注册，请使用新的邮箱注册！");
}
```
这种方案在某些场景下是有效果的，例如请求不是非常频繁，可以采用这种方式。

那如果请求非常频繁，而且服务层需要处理的逻辑非常多的时候，这种方案就会遇到很大的瓶颈。

以订单支付为例，当用户支付时，首先会对订单数据做各种基础验证，接着走风控系统，鉴别是否是机器人操作，风控系统通过之后，再对接银行系统查询用户金额是否充足，如果充足就申请扣款，扣款成功之后，更新订单状态，同时将订单的数据推送给中心仓库，等待发货。

当然这个只是一个基础的流程，实际的处理逻辑比这个要复杂的多，此时我们也不能像上面介绍的那样对某个关键字做唯一约束，同时整个处理逻辑所需的时间也相对比较长，假如有几个请求同时过来，其结果可想而知！

##### 方案二：利用缓存ID防止重复提交（推荐）
设想一下，前端在请求后端的时候，先从后端缓存中获取一个唯一的ID，在请求提交数据的时候带上这个唯一的ID，后端检查缓存中是否存在这个ID，如果存在，就进行业务处理，处理完毕之后，从缓存中将这个ID移除掉，如果在处理过程中，前端又再次提交，此时缓存中的ID状态还没有被移除，直接提示：**数据处理中，不要重复提交....**，具体流程如下！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/submit-token-data/02.jpg)

* 先编写一个缓存工具类

```java
/**
 * 缓存工具类
 */
public class CacheUtil {

    //hashMap线程安全类
    private static Map<String,Object> cacheMap = new ConcurrentHashMap<>();

    /**
     * 添加缓存
     * @param key
     * @param value
     */
    public static void addCache(String key,Object value){
        cacheMap.put(key, value);
    }

    /**
     * 设置缓存
     * @param key
     * @param value
     */
    public static void setValue(String key,Object value){
        cacheMap.put(key, value);
    }

    /**
     * 获取缓存
     * @param key
     * @return
     */
    public static Object getValue(String key){
        return cacheMap.get(key);
    }

    /**
     * 判断key是存在
     * @param key
     * @return
     */
    public static boolean containKey(String key){
        return cacheMap.containsKey(key);
    }

    /**
     * 移除缓存
     * @param key
     */
    public static void removeCache(String key){
        cacheMap.remove(key);
    }

}
```

* 再编写一个获取唯一ID的方法

```java
@PostMapping("/getSubmitToken")
public Object getSubmitToken(){
    String submitToken = UUID.randomUUID().toString();
    //将事务请求唯一ID放入缓存池
    CacheUtil.addCache(submitToken, "false");
	//将ID返回给前端
    JSONObject result = new JSONObject();
    result.put("submitToken", submitToken);
    return result;
}
```

* 接着编写一个注解，用于需要验证重复提交的方法上

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface SubmitToken {

    boolean value() default true;
}
```

* 然后编写一个拦截器，用于类或者方法上有`@SubmitToken`注解的验证处理

```java
/**
 * 重复提交拦截器
 */
public class SubmitTokenInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //如果不是映射到方法，直接通过
        if(!(handler instanceof HandlerMethod)){
            return true;
        }
        //如果类或者方法有SubmitToken注解，则进行重复提交验证
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        if (handlerMethod.getBeanType().isAnnotationPresent(SubmitToken.class) || handlerMethod.getMethod().isAnnotationPresent(SubmitToken.class)) {
            final String submitToken = request.getParameter("submitToken");
            if(StringUtils.isEmpty(submitToken)){
                throw new CommonException("submitToken不能为空！");
            }
            if(!CacheUtil.containKey(submitToken)){
                throw new CommonException("submitToken失效，请重新获取！");
            }
            Object value = CacheUtil.getValue(submitToken);
            if(!"false".equals(value)){
                throw new CommonException("数据正在处理，请不要重复提交");
            }
            //验证通过之后，将submitToken对应的值设置为正在处理
            CacheUtil.setValue(submitToken, "true");
        }
        return true;
    }


    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        //业务处理完毕之后，将submitToken从缓存中移除
        final String submitToken = request.getParameter("submitToken");
        if(StringUtils.isNotEmpty(submitToken)){
            CacheUtil.removeCache(submitToken);
        }
    }
}
```

* 最后将`@SubmitToken`注解用于需要进行重复提交的方法或者类上

```java
/**
 * 将SubmitToken用于增、删、改的方法或者类上
 */
@SubmitToken
@RequestMapping(value = "/register")
public boolean register(@RequestBody UserDto userDto) throws Exception {
    //......
}
```

在开发的时候，我们只需将`@SubmitToken`用于增、删、改的方法上即可，当前端在提交数据的时候，先通过`/getSubmitToken`接口获取一个`submitToken`也就是唯一ID，然后再提交请求的时候，带上这个参数即可！

当你真正在使用的时候，对于缓存类你会发现还有很大的优化空间，本例采用的是`ConcurrentHashMap`作为缓存类，随着提交请求量越来越多，缓存类所占用的空间也越来越大，最后很有可能会OOM。

因此有两种解决办法：

* 第一种：编写一个缓存实体类，里面存放有效期，然后弄一个线程来扫描缓存map，到达过期的数据就将其移除。
* 第二种：将需要缓存的数据写入到redis，同时设置过期时间。

如果是小项目，第一种方法就基本可以解决，如果是中大型项目，那么推荐使用 redis 搭建高可用的缓存集群，同时一定要注意 key 的设计，最好采用单独的前缀，例如`submittoken-uuid- `+ `项目名称`作为前缀，方便后期扩展的时候缓存数据迁移！

### 四、总结
本文主要围绕**后端如何防止重复提交数据问题**进行一些总结，可能也有遗漏的地方，欢迎网友点评、吐槽！
