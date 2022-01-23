---
layout: post
title: 后端JAVAWeb工程师必须掌握的三个内容！！
tagline: by 懿
categories: javaWeb
tags: 
    - 懿
---

我们都是作为一个JAVA开发，之前有好几次出去面试，面试官都问我，JAVAWeb掌握的怎么样，我当时就不知道怎么回答，Web，日常开发中用的是什么？今天我们来说说JAVAWeb最应该掌握的三个内容。
<!--more-->

## JAVAWeb知识点一 --Cookie

作为一个进入职场一年的年轻人来说，可能还没有很强的能力达到从前端的HTML，CSS和JS再到后端的JAVA代码通通都是自己写，但是至少在前后端交互的过程中，我们应该了解他们之间交互的一些东西，比如：我下面所说的内容一

`Cookie`

在Web应用中，HTTP请求是无状态的。也就是说用户第一次发起请求，与服务器建立连接并登录成功后，为了避免每次打开一个页面都需要登录一下，就出现了Cookie，Session。

我们先来说说这个Cookie。

Cookie翻译成中文的意思是‘小甜饼’，是由W3C组织提出，最早由Netscape社区发展的一种机制。目前Cookie已经成为标准，所有的主流浏览器如IE、Netscape、Firefox、Opera等都支持Cookie。

服务器单从网络连接上无从知道客户身份。怎么办呢？就给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理。

Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。Cookie存储的数据量有限，且都是保存在客户端浏览器中。不同的浏览器有不同的存储大小，但一般不超过4KB。因此使用Cookie实际上只能存储一小段的文本信息。

说到Cookie技术是将用户的数据存储到客户端的技术，那么我们的服务器端怎样将一个Cookie发送到客户端？服务器端又是怎样接受客户端携带的Cookie？

服务器端怎样将一个Cookie发送到客户端？

我们来写一段小小的代码

(1)创建Cookie：

```

    Cookie cookie = new Cookie(String cookieName,String cookieValue);

示例：
    Cookie cookie = new Cookie("username"，"zhangsan");
    那么该cookie会以响应头的形式发送给客户端：
    Set-Cookie:"name=zhangsan"

```

(2)设置Cookie在客户端的持久化时间：

```
    cookie.setMaxAge(int seconds); ---时间秒
    
    注意：如果不设置持久化时间，cookie会存储在浏览器的内存中，浏览器关闭
    cookie信息销毁（会话级别的cookie），如果设置持久化时间，cookie信息会	被持久化到浏览器的磁盘文件里

示例：
    cookie.setMaxAge(10*60);
    设置cookie信息在浏览器的磁盘文件中存储的时间是10分钟，过期浏览器自动删除该cookie信息
```

(3)设置Cookie的携带路径：

```
    cookie.setPath(String path);
    注意：如果不设置携带路径，那么该cookie信息会在访问产生该cookie的web资源所在的路径都携带cookie信息
示例：
    cookie.setPath("/match01");
    代表访问match01应用中的任何资源都携带cookie
    cookie.setPath("/match01/cookieServlet");
    代表访问match01中的cookieServlet时才携带cookie信息
```

(4)向客户端发送cookie：

```
    response.addCookie(Cookie cookie);
```

(5)删除客户端的cookie

```
    如果想删除客户端的已经存储的cookie信息，那么就使用同名同路径的持久化时	间为0的cookie进行覆盖即可
```

我们再来说说服务器端怎么接受客户端携带的Cookie？

cookie信息是以请求头的方式发送到服务器端的：Cookie：“name=zhangsan”

(1)通过request获得所有的Cookie;

```
Cookie[] cookies = request.getCookies();
```

(2)遍历Cookie数组，通过Cookie的名称获得我们想要的Cookie

```
    for(Cookie cookie : cookies){
        if(cookie.getName().equal(cookieName)){
        String cookieValue = cookie.getValue();
    }
}
```

上面的简单代码就是对Cookie的一些简单的操作，具体情况肯定要根据具体情况来进行分析，怎么使用，那就得全靠大家自己来进行实际操作了。Cookie还有很多操作的方法，有兴趣的可以去jar包中研究一下他的源码，Cookie类位于包javax.servlet.http.*的下面。

注意：Cookie功能需要浏览器的支持。如果浏览器不支持Cookie（如大部分手机中的浏览器）或者把Cookie禁用了，Cookie功能就会失效。不同的浏览器采用不同的方式保存Cookie。IE浏览器会在“C:\Documents and Settings\你的用户名\Cookies”文件夹下以文本文件形式保存，一个文本文件保存一个Cookie。

Cookies最典型的应用是判定注册用户是否已经登录网站，用户可能会得到提示，是否在下一次进入此网站时保留用户信息以便简化登录手续，这些都是Cookies的功用。另一个重要应用场合是“购物车”之类处理。用户可能会在一段时间内在同一家网站的不同页面中选择不同的商品，这些信息都会写入Cookies，以便在最后付款时提取信息。

## JAVAWeb知识点二 --Session

`Session`

除了使用Cookie，Web应用程序中还经常使用Session来记录客户端状态。Session技术是将数据存储在服务器端的技术，会为每个客户端都创建一块内存空间	存储客户的数据，但客户端需要每次都携带一个标识ID去服务器中寻找属于自己的内	存空间。所以说Session的实现是基于Cookie，Session需要借助于Cookie存储客	户的唯一性标识JSESSIONID。

我们从三个方面来分析这个Session。

1.怎样获得属于本客户端的session对象（内存区域）？

2.怎样向session中存取数据（session也是一个域对象）？

3.session对象的生命周期？

(1)获得Session对象

```
    HttpSession session = request.getSession();
    此方法会获得专属于当前会话的Session对象，如果服务器端没有该会话的Session对象会创建一个新的Session返回，
    如果已经有了属于该会话的Session直接将已有	的Session返回（实质就是根据JSESSIONID判断该客户端是否在服务器上已经存在	session了）
```

(2)怎样向session中存取数据（session也是一个域对象）

```
Session也是存储数据的区域对象，所以session对象也具有如下三个方法：
    session.setAttribute(String name,Object obj);
    session.getAttribute(String name);
    session.removeAttribute(String name);
```

(3)Session对象的生命周期

第三点也是面试里面会经常问到的内容，不论是笔试还是面试，都经常会出现这个问题。

创建：第一次执行request.getSession()时创建

销毁：
1）服务器（非正常）关闭时

2）session过期/失效（默认30分钟）也有人说是20分钟，我专门去tomcat的conf目录下面的web.xml看过了，我可以肯定是30分钟。

问题：时间的起算点 从何时开始计算30分钟？
从不操作服务器端的资源开始计时

可以在工程的web.xml中进行配置
```
    <session-config>
            <session-timeout>30</session-timeout>
    </session-config>
```
3）手动销毁session
```
session.invalidate();
```

作用范围：默认在一次会话中，也就是说在，一次会话中任何资源公用一个session对象

面试题：浏览器关闭，session就销毁了？ 不对.

Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，并维护该Session。为防止内存溢出，服务器会把长时间内没有活跃的Session从内存删除。这个时间就是Session的超时时间。如果超过了超时时间没访问过服务器，Session就自动失效了。

## JAVAWeb知识点二 --Token

`Token`

什么是token？

token的意思是“令牌”，是服务端生成的一串字符串，作为客户端进行请求的一个标识。

当用户第一次登录后，服务器生成一个token并将此token返回给客户端，以后客户端只需带上这个token前来请求数据即可，无需再次带上用户名和密码。

简单token的组成；uid(用户唯一的身份标识)、time(当前时间的时间戳)、sign（签名，token的前几位以哈希算法压缩成的一定长度的十六进制字符串。为防止token泄露）。

Token的作用是什么呢？

(1)防止表单重复提交

防止表单重复提交一般还是使用前后端都限制的方式，比如：在前端点击提交之后，将按钮置为灰色，不可再次点击，然后客户端和服务端的token各自独立存储，客户端存储在Cookie或者Form的隐藏域（放在Form隐藏域中的时候，需要每个表单）中，服务端存储在Session（单机系统中可以使用）或者其他缓存系统（分布式系统可以使用）中。

(2)用来作身份验证,一般情况下用在APP中。

用户在登录APP时，APP端会发送加密的用户名和密码到服务器，服务器验证用户名和密码，如果验证成功，就会生成相应位数的字符产作为token存储到服务器中，并且将该token返回给APP端。

以后APP再次请求时，凡是需要验证的地方都要带上该token，然后服务器端验证token，成功返回所需要的结果，失败返回错误信息，让用户重新登录。其中，服务器上会给token设置一个有效期，每次APP请求的时候都验证token和有效期。

而我们又怎么去存储这个Token而且防止Token泄露呢？

Token的存储方式：

Token可以存到数据库中，但是有可能查询token的时间会过长导致token丢失（其实token丢失了再重新认证一个就好，但是别丢太频繁，别让用户没事儿就去认证）。

为了避免查询时间过长，可以将token放到内存中。这样查询速度绝对就不是问题了，也不用太担心占据内存，就算token是一个32位的字符串，应用的用户量在百万级或者千万级，也是占不了多少内存的。

Token的加密

在存储的时候把token进行对称加密存储，用到的时候再解密。我们可以将请求URL、时间戳、token三者合并，通过算法进行加密处理。至于这个算法，等待之后的文章中去专门研究这个加密和解密的操作。

还有一点，在网络层面上Token使用明文传输的话是非常危险的，所以一定要使用HTTPS协议。

我是懿，一个正在被打击还在努力前进的码农。欢迎大家关注我们的公众号，加入我们的知识星球，我们在知识星球中等着你的加入。

