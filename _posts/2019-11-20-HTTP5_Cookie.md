---
layout: post
title: 你还拿着培训班教给你的 Cookie 去面试？确定不要进来看一下 Cookie 到底是什么样子的？
categories: HTTP系列
tags:
	- 懿
---

说实话，之前面试都是直接去背诵的面试题，关于 Cookie 的一些内容，比如说，记录浏览器端的数据信息啦， Cookie 的生命周期啦，这些内容，也从来没有研究过 Cookie 到底是怎么工作的，今天我们就来了解一下 Cookie 到底是怎么工作的。
<!--more-->

## 1.什么是Cookie 

Cookie 是当前识别用户，实现持久会话的最好方式。 前面各种技术中存在的很多问题对它们都没什么影响， 但是通常会将它们与那些技术共用， 以实现额外的价值。 Cookie 最初是由网景公司开发的， 但现在所有主要的浏览器都支持它。Cookie 非常重要， 而且它们定义了一些新的 HTTP 首部， 所以我们要比前面那些技术更详细地介绍它们。 Cookie 的存在也影响了缓存， 大多数缓存和浏览器都不允许对任何 Cookie 的内容进行缓存。 


## 2.Cookie 的类型

- 会话 Cookie

会话 Cookie 是一种临时Cookie， 它记录了用户访问站点时的设置和偏好。 用户退出浏览器时， 会话 Cookie 就被删除了。

- 持久 Cookie

持久 Cookie 的生存时间更长一些； 它们存储在硬盘上， 浏览器退出， 计算机重启时它们仍然存在。 通常会用持久 Cookie 维护某个用户会周期性访问的站点的配置文件或登录名。

那么问题来了，这两种 Cookie 的区别又在哪里，不用想，看名字，持久，会话，是不是明白了，对，没错，会话 Cookie 和持久 Cookie 之间唯一的区别就是它们的过期时间。稍后我们会看到，如果 设置了 Discard 参数，或者没有设置 Expires 或 Max-Age 参数来说明扩展的过期时 间，这个 Cookie 就是一个会话 Cookie。

没错，两个 Cookie 之间就是这么容易区别，但是很多时候，面试官问的时候，我们却把这个给忘的干干净净。

## 3.面试官：那你来说一下这个 Cookie 是怎么工作的

我们来看一幅图:

![](http://www.justdojava.com/assets/images/2019/java/image_yi/11_20/1.jpg)

图中的 a 段 ,当我们在第一次访问 Web 网站的时候， Web 服务器对我们现在过去的用户信息一无所知。

Web 服务器希望这个用户会再次回来，所以想给这个用户“拍上”一个独有的 cookie，这样以后它就可以识别出这个用户了。cookie 中包含了一个由名字 = 值（name=value）这样的信息构成的 任意列表，并通过 Set-Cookie 或 Set-Cookie2 HTTP 响应（扩展）首部将其贴到用户身上去。

但是 Cookie 中能存储的信息，绝对不会只是 ID 号，相信大家做开发的，也肯定知道 Cookie 能存什么，比如说，

```
Cookie: name="Brian Totty"; phone="555-1212"
```
我存了个这东西，它也是允许的。

浏览器会记住从服务器返回的 Set-Cookie 或 Set-Cookie2 首部中的 Cookie 内容，并将 Cookie 集存储在浏览器的 Cookie 数据库中（把它当作一个贴有不同国家贴纸的旅行箱）。将来用户返回同一站点时图中的 c 段，浏览器会挑中那个服务器贴到用户上的那些 Cookie，并在一个 Cookie 请求首部中将其传回去。

这就是 Cookie 的工作原理，Cookie 就像服务器给用户贴的“嗨，我叫”的贴纸一样。用户访问一个 Web 站点时，这个 Web 站点就可以读取那个服务器贴在用户身上的所有贴纸。

## 4.Cookie罐

说实话哈，我在看到这个名词的时候，第一反应就是，翻译书籍的作者，有点懵人呀，直接这么翻译，不太负责任呀，后来发现，这么翻译，确实也是有一定道理的，原因如下：

因为 Cookie 的基本思想就是让浏览器积累一组服务器特有的信息，每次访问服务器时都将这些信息提供给它。浏览器内部的 Cookie 罐总可以有成百上千个 Cookie，也是因为浏览器要负责存储 Cookie 信息，所以此系统被称为客户端侧状态 （client-side state）。这个 Cookie 规范的正式名称为 HTTP 状态管理机制（HTTP state management mechanism）。

我猜测因为它能够存储如此多的 Cookie ，所以当时书中的作者就翻译成了 Cookie 罐，这么想起来，也是很有道理的。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/11_20/2.jpg)

上图，是在某一个网站的时候，正在使用的 Cookie ，我的是中文的， 具体的 Cookie 的位置，以 Google 浏览器为例子，在你的安装目录下，`D:\software\360Chrome\Chrome\User Data\Default` ，里面有 Cookies 的一个文档。不同的浏览器位置是不一样的，如果各位还想看，请打开你的搜索引擎，开始你的表演把。

因为 Cookie 里面是加密的信息，所以，我就不给大家看一堆乱码了。

**不同站点使用不同的 Cookie**

浏览器内部的 cookie 罐中可以有成百上千个 cookie，但浏览器不会将每个 cookie 都发送 给所有的站点。实际上，它们通常只向每个站点发送 2 ～ 3 个 Cookie。大家看我上面的图，正在使用的可不是那么一个，是好多个。为什么？原因如下：

- 对所有这些 Cookie 字节进行传输会严重降低性能。浏览器实际传输的 Cookie 字节数要比实际的内容字节数多！
- Cookie 中包含的是服务器特有的名值对，所以对大部分站点来说，大多数 Cookie 都只是无法识别的无用数据。
- 将所有的 Cookie 发送给所有站点会引发潜在的隐私问题，那些你并不信任的站点也 会获得你只想发给其他站点的信息。

说白了，就是浏览器只向服务器发送服务器产生的那些 Cookie。

很多 Web 站点都会与第三方厂商达成协议，由其来管理广告。这些广告被做得像 Web 站 点的一个组成部分，而且它们确实发送了持久 Cookie。用户访问另一个由同一广告公司提供服务的站点时，（由于域是匹配的）浏览器就会再次回送早先设置的持久 Cookie。营销公司可以将此技术与 Referer 首部结合，暗地里构建一个用户档案和浏览习惯的详尽数 据集。现代的浏览器都允许用户对隐私特性进行设置，以限制第三方 Cookie 的使用。

## 5.Cookie 安全

这个才是本文的重点，不得不承认，Cookie是一个神奇的机制，同域内浏览器中发出的任何一个请求都会带上 Cookie ，无论请求什么资源，请求时，Cookie出现在请求头的Cookie字段中。服务端响应头的 Set-Cookie 字段可以添加、修改和删除 Cookie，大多数情况下，客户端通过 JavaScript 也可以添加、修改和删除 Cookie。

相信很多某机构出来的哥们一定想到了一个东西，跨域，是的，就是它，因为 Cookie 子域机制，其实这也是 Cookie 的domain字段的机制，设置Cookie时，如果不指定domain的值，默认就是本域。这也是很多培训机构中说的 SSO 单点登录跨域。

假如说，我们现在有一个 `a.test.com`的域，通过 JavaScript 来设置一个 Cookie，语句如下：

```
document.cookie="test=1";
```

那么，domain值默认为a.test.com，但是，`a.test.com` 域设置 Cookie 时，可以指定 domain 为父级域，比如：

```
document.cookie="test=1;domain=test.com";
```

此时，domain 就变为 test.com，这样带来的好处就是可以在不同的子域共享 Cookie，坏处也很明显，就是攻击者控制的其他子域也能读到这个 Cookie。另外，这个机制不允许设置 Cookie 的 domain 为下一级子域或其他外域。

这时候就有人会说了，Jsonp 的方式不是可以让我们很容易的实现跨域的操作了么？是的，是能实现，但是 Get 请求是可以的，那么 POST 请求呢？想没想过？那我来给大家安利一个。

**CORS**

跨域资源共享机制，它使用额外的 HTTP 头来告诉浏览器  让运行在一个 origin (domain) 上的Web应用被准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器不同的域、协议或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。

比如，站点 http://domain-a.com 的某 HTML 页面通过 <img> 的 src 请求 http://domain-b.com/image.jpg。网络上的许多页面都会加载来自不同域的 CSS 样式表，图像和脚本等资源

出于安全原因，浏览器限制从脚本内发起的跨域 HTTP 请求。 例如，XMLHttpRequest 和 Fetch API 遵循同源策略。 这意味着使用这些 API 的 We 应用程序只能从加载应用程序的同一个域请求 HTTP 资源，除非响应报文包含了正确 CORS 响应头。

跨域资源共享（ CORS ）机制允许 Web 应用服务器进行跨域访问控制，从而使跨域数据传输得以安全进行。现代浏览器支持在 API 容器中（例如 XMLHttpRequest 或 Fetch ）使用 CORS，以降低跨域 HTTP 请求所带来的风险。

**CORS流程**

- 请求发起时,浏览器先判断当前是否是跨域的AJAX;

- 如果是,判断是否是普通类型请求(GET类型,无自定义头数据)；

- 普通请求,直接发起GET到服务端,在响应头中寻找 Access-Contro-Alow- Origin,如果有且允许,处理响应结果；

- 不是普通请求(非GET类型,或有自定义头), 先 PreFlight(即发起一个 method= OPTIONS)的请求；

- 要求返回 Access-Control-Allow- Methods 和 Access-Control-Allow- Headers, 内容体为空；

- PreFlight 正确执行后, 再发起GET请求, 获得响应结果, 并处理结果.

好了关于 HTTP 中的 Cookie 机制就说这些把，说的太多了，恐怕大家会烦，下一篇文章，我们来详细说 HTTP 的加密解密。很有意思呦，敬请期待！

文章参考：

《图解 HTTP》

《HTTP 权威指南》

我是懿，一个正在被打击却努力前进的码农。