---
layout: post
categories: springboot
title: 手把手教你，使用JWT实现单点登录
tags: 
  - 炸鸡可乐
---

JSON Web Token（JWT）是目前最流行的跨域身份验证解决方案之一，今天我们一起来揭开它神秘的面纱！

<!--more-->

### 一、故事起源
说起 JWT，我们先来谈一谈基于传统`session`认证的方案以及瓶颈。

传统`session`交互流程，如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/01.png)

当浏览器向服务器发送登录请求时，验证通过之后，会将用户信息存入`seesion`中，然后服务器会生成一个`sessionId`放入`cookie`中，随后返回给浏览器。

当浏览器再次发送请求时，会在请求头部的`cookie`中放入`sessionId`，将请求数据一并发送给服务器。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/02.jpg)

服务器就可以再次从`seesion`获取用户信息，整个流程完毕！

通常在服务端会设置`seesion`的时长，例如 30 分钟没有活动，会将已经存放的用户信息从`seesion`中移除。
```java
session.setMaxInactiveInterval(30 * 60);//30分钟没活动，自动移除
```
同时，在服务端也可以通过`seesion`来判断当前用户是否已经登录，如果为空表示没有登录，直接跳转到登录页面；如果不为空，可以从`session`中获取用户信息即可进行后续操作。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/03.png)

在单体应用中，这样的交互方式，是没啥问题的。

但是，假如应用服务器的请求量变得很大，而单台服务器能支撑的请求量是有限的，这个时候就容易出现请求变慢或者`OOM`。

解决的办法，要么给单台服务器增加配置，要么增加新的服务器，通过负载均衡来满足业务的需求。

如果是给单台服务器增加配置，请求量继续变大，依然无法支撑业务处理。

显而易见，增加新的服务器，可以实现无限的水平扩展。

但是增加新的服务器之后，不同的服务器之间的`sessionId`是不一样的，可能在`A`服务器上已经登录成功了，能从服务器的`session`中获取用户信息，但是在`B`服务器上却查不到`session`信息，此时肯定无比的尴尬，只好退出来继续登录，结果`A`服务器中的`session`因为超时失效，登录之后又被强制退出来要求重新登录，想想都挺尴尬～～

面对这种情况，几位大佬于是合起来商议，想出了一个`token`方案。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/04.png)

将各个应用程序与内存数据库`redis`相连，对登录成功的用户信息进行一定的算法加密，生成的`ID`被称为`token`，将`token`还有用户的信息存入`redis`；等用户再次发起请求的时候，将`token`还有请求数据一并发送给服务器，服务端验证`token`是否存在`redis`中，如果存在，表示验证通过，如果不存在，告诉浏览器跳转到登录页面，流程结束。

`token`方案保证了服务的无状态，所有的信息都是存在分布式缓存中。基于分布式存储，这样可以水平扩展来支持高并发。

当然，现在`springboot`还提供了`session`共享方案，类似`token`方案将`session`存入到`redis`中，在集群环境下实现一次登录之后，每个服务器都可以获取到用户信息。
### 二、JWT是什么
上文中，我们谈到的`session`还有`token`的方案，在集群环境下，他们都是靠第三方缓存数据库`redis`来实现数据的共享。

那有没有一种方案，不用缓存数据库`redis`来实现用户信息的共享，以达到一次登录，处处可见的效果呢？

答案肯定是有的，就是我们今天要介绍的`JWT`！

`JWT`全称`JSON Web Token`，实现过程简单的说就是用户登录成功之后，将用户的信息进行加密，然后生成一个`token`返回给客户端，与传统的`session`交互没太大区别。

交互流程如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/05.png)

**唯一的不同点就是**：`token`存放了用户的基本信息，更直观一点就是将原本放入`redis`中的用户数据，放入到`token`中去了！

这样一来，客户端、服务端都可以从`token`中获取用户的基本信息，既然客户端可以获取，**肯定是不能存放敏感信息的**，因为浏览器可以直接从`token`获取用户信息。

#### JWT具体长什么样呢？

JWT是由三段信息构成的，将这三段信息文本用`.`链接一起就构成了`JWT`字符串。就像这样:
```java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

* 第一部分：我们称它为头部（header），用于存放token类型和加密协议，一般都是固定的；
* 第二部分：我们称其为载荷（payload），用户数据就存放在里面；
* 第三部分：是签证（signature），主要用于服务端的验证；

##### 1、header
JWT的头部承载两部分信息：

* 声明类型，这里是JWT；
* 声明加密的算法，通常直接使用 HMAC SHA256；

完整的头部就像下面这样的JSON：
```java
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```
使用`base64`加密，构成了第一部分。
```java
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```
##### 2、playload
载荷就是存放有效信息的地方，这些有效信息包含三个部分：

* 标准中注册的声明；
* 公共的声明；
* 私有的声明；

**其中，标准中注册的声明 (建议但不强制使用)包括如下几个部分** ：

* iss: jwt签发者；
* sub: jwt所面向的用户；
* aud: 接收jwt的一方；
* exp: jwt的过期时间，这个过期时间必须要大于签发时间；
* nbf: 定义在什么时间之前，该jwt都是不可用的；
* iat: jwt的签发时间；
* jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击；

**公共的声明部分**：
公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息，但不建议添加敏感信息，因为该部分在客户端可解密。

**私有的声明部分**：
私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为`base64`是对称解密的，意味着该部分信息可以归类为明文信息。

定义一个payload：
```java
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
然后将其进行`base64`加密，得到`Jwt`的第二部分：
```java
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```
##### 3、signature
jwt的第三部分是一个签证信息，这个签证信息由三部分组成：

* header (base64后的)；
* payload (base64后的)；
* secret (密钥);

这个部分需要`base64`加密后的`header`和`base64`加密后的`payload`使用`.`连接组成的字符串，然后通过`header`中声明的加密方式进行加盐`secret`组合加密，然后就构成了`jwt`的第三部分。
```java
//javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, '密钥');
```
加密之后，得到`signature`签名信息。
```java
TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

将这三部分用`.`连接成一个完整的字符串，就构成了最终的jwt：

```java
//jwt最终格式
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
这个只是通过`javascript`实现的一个演示，`JWT`的签发和密钥的保存都是在服务端来完成。

**`secret`用来进行`jwt`的签发和`jwt`的验证，所以，在任何场景都不应该流露出去**。

### 三、实战
介绍了这么多，怎么实现呢？废话不多说，下面我们直接开撸！

* 创建一个`springboot`项目，添加`JWT`依赖库

```xml
<!-- jwt支持 -->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.4.0</version>
</dependency>
```

* 然后，创建一个用户信息类，将会通过加密存放在`token`中

```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
public class UserToken implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 用户ID
     */
    private String userId;

    /**
     * 用户登录账户
     */
    private String userNo;

    /**
     * 用户中文名
     */
    private String userName;
}
```

* 接着，创建一个`JwtTokenUtil`工具类，用于创建`token`、验证`token`

```java
public class JwtTokenUtil {

	//定义token返回头部
    public static final String AUTH_HEADER_KEY = "Authorization";

	//token前缀
    public static final String TOKEN_PREFIX = "Bearer ";

	//签名密钥
    public static final String KEY = "q3t6w9z$C&F)J@NcQfTjWnZr4u7x";
	
	//有效期默认为 2hour
    public static final Long EXPIRATION_TIME = 1000L*60*60*2;


    /**
     * 创建TOKEN
     * @param content
     * @return
     */
    public static String createToken(String content){
        return TOKEN_PREFIX + JWT.create()
                .withSubject(content)
                .withExpiresAt(new Date(System.currentTimeMillis() + EXPIRATION_TIME))
                .sign(Algorithm.HMAC512(KEY));
    }

    /**
     * 验证token
     * @param token
     */
    public static String verifyToken(String token) throws Exception {
        try {
            return JWT.require(Algorithm.HMAC512(KEY))
                    .build()
                    .verify(token.replace(TOKEN_PREFIX, ""))
                    .getSubject();
        } catch (TokenExpiredException e){
            throw new Exception("token已失效，请重新登录",e);
        } catch (JWTVerificationException e) {
            throw new Exception("token验证失败！",e);
        }
    }
}
```

* 编写配置类，允许跨域，并且创建一个权限拦截器

```java
@Slf4j
@Configuration
public class GlobalWebMvcConfig implements WebMvcConfigurer {
	   /**
     * 重写父类提供的跨域请求处理的接口
     * @param registry
     */
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // 添加映射路径
        registry.addMapping("/**")
                // 放行哪些原始域
                .allowedOrigins("*")
                // 是否发送Cookie信息
                .allowCredentials(true)
                // 放行哪些原始域(请求方式)
                .allowedMethods("GET", "POST", "DELETE", "PUT", "OPTIONS", "HEAD")
                // 放行哪些原始域(头部信息)
                .allowedHeaders("*")
                // 暴露哪些头部信息（因为跨域访问默认不能获取全部头部信息）
                .exposedHeaders("Server","Content-Length", "Authorization", "Access-Token", "Access-Control-Allow-Origin","Access-Control-Allow-Credentials");
    }

    /**
     * 添加拦截器
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //添加权限拦截器
        registry.addInterceptor(new AuthenticationInterceptor()).addPathPatterns("/**").excludePathPatterns("/static/**");
    }
}
```

* 使用`AuthenticationInterceptor`拦截器对接口参数进行验证

```java
@Slf4j
public class AuthenticationInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		// 从http请求头中取出token
        final String token = request.getHeader(JwtTokenUtil.AUTH_HEADER_KEY);
        //如果不是映射到方法，直接通过
        if(!(handler instanceof HandlerMethod)){
            return true;
        }
        //如果是方法探测，直接通过
        if (HttpMethod.OPTIONS.equals(request.getMethod())) {
            response.setStatus(HttpServletResponse.SC_OK);
            return true;
        }
        //如果方法有JwtIgnore注解，直接通过
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method=handlerMethod.getMethod();
        if (method.isAnnotationPresent(JwtIgnore.class)) {
            JwtIgnore jwtIgnore = method.getAnnotation(JwtIgnore.class);
            if(jwtIgnore.value()){
                return true;
            }
        }
        LocalAssert.isStringEmpty(token, "token为空，鉴权失败！");
        //验证，并获取token内部信息
        String userToken = JwtTokenUtil.verifyToken(token);
		
		//将token放入本地缓存
        WebContextUtil.setUserToken(userToken);
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        //方法结束后，移除缓存的token
        WebContextUtil.removeUserToken();
    }
}
```

* 最后，在`controller`层用户登录之后，创建一个`token`，存放在头部即可

```java
/**
 * 登录
 * @param userDto
 * @return
 */
@JwtIgnore
@RequestMapping(value = "/login", method = RequestMethod.POST, produces = {"application/json;charset=UTF-8"})
public UserVo login(@RequestBody UserDto userDto, HttpServletResponse response){
    //...参数合法性验证

    //从数据库获取用户信息
    User dbUser = userService.selectByUserNo(userDto.getUserNo);

    //....用户、密码验证

    //创建token，并将token放在响应头
    UserToken userToken = new UserToken();
    BeanUtils.copyProperties(dbUser,userToken);

    String token = JwtTokenUtil.createToken(JSONObject.toJSONString(userToken));
    response.setHeader(JwtTokenUtil.AUTH_HEADER_KEY, token);


    //定义返回结果
    UserVo result = new UserVo();
    BeanUtils.copyProperties(dbUser,result);
    return result;
}
```
**到这里基本就完成了！**


其中`AuthenticationInterceptor`中用到的`JwtIgnore`是一个注解，用于不需要验证`token`的方法上，例如验证码的获取等等。
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface JwtIgnore {

    boolean value() default true;
}
```

而`WebContextUtil`是一个线程缓存工具类，其他接口通过这个方法即可从`token`中获取用户信息。

```java
public class WebContextUtil {

    //本地线程缓存token
    private static ThreadLocal<String> local = new ThreadLocal<>();

    /**
     * 设置token信息
     * @param content
     */
    public static void setUserToken(String content){
        removeUserToken();
        local.set(content);
    }

    /**
     * 获取token信息
     * @return
     */
    public static UserToken getUserToken(){
        if(local.get() != null){
            UserToken userToken = JSONObject.parseObject(local.get() , UserToken.class);
            return userToken;
        }
        return null;
    }

    /**
     * 移除token信息
     * @return
     */
    public static void removeUserToken(){
        if(local.get() != null){
            local.remove();
        }
    }
}
```

最后，启动项目，我们来用`postman`测试一下，看看头部返回结果。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/06.jpg)

我们把返回的信息提取处理，使用浏览器的`base64`对前两个部分进行解密。

* 第一部分，也就是header，结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/07.jpg)

* 第二部分，也就是playload，结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/08.jpg)

可以很清晰的看到，头部、载荷的信息都可以通过`base64`解密出来。

**所以，一定别在`token`中存放敏感信息**！

当我们需要请求其它服务接口时，只需要在请求头部`headers`中加入`Authorization`参数即可。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-jwt/09.jpg)

当权限拦截器验证通过之后，在接口方法中只需要通过`WebContextUtil`工具类就可以获取用户信息。
```java
//获取用户token信息
UserToken userToken = WebContextUtil.getUserToken();
```
### 四、总结
`JWT`相比`session`方案，因为`json`的通用性，所以`JWT`是可以进行跨语言支持的，像`JAVA`、`JavaScript`、`PHP`等很多语言都可以使用，而`session`方案只针对`JAVA`。

因为有了`payload`部分，所以`JWT`可以存储一些其他业务逻辑所必要的非敏感信息。

同时，保护好服务端`secret`私钥非常重要，因为私钥可以对数据进行验证、解密！

如果可以，请使用`https`协议！

### 五、参考
1、[简书 - 什么是 JWT -- JSON WEB TOKEN](https://www.jianshu.com/p/576dbf44b2ae)

2、[博客园 - 基于session和token的身份认证方案](https://www.cnblogs.com/xiangkejin/p/9011007.html)

