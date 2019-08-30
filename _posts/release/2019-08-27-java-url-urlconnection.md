---
layout: post
category: 程序员
title: Java 网络编程：必知必会的 URL 和 URLConnection
tagline: by 沉默王二
tags: 
  - 网络编程
---

`java.net.URL` 类将 URL 地址进行了封装，并提供了解析 URL 地址的基本方法，比如获取 URL 的主机名和端口号。`java.net.URLConnection` 则代表了应用程序和 URL 之间的通信链接，可用于读取和写入此 URL 引用的资源。

URLConnection 看起来只是比 URL 多了一个 Connection，它们之间的关系也仅限于此吗？

<!--more-->
### 01、什么是 URL

为了搞清楚什么是 URL，需要引入另外两个概念 URI 和 URN。

什么鬼，URL 都没搞清楚，又来两个搞不清楚的？别担心，我能像变了魔法一样让大家把三个都搞清楚。

- URI = Universal Resource Identifier，中文释义为统一资源标志符
- URL = Universal Resource Locator，中文释义为统一资源定位符
- URN = Universal Resource Name，中文释义为统一资源名称

它们之间的关系如下图所示：

![](https://static.xmt.cn/4807d477c6c949d38506729633eab847.png)

这图啥意思啊，怎么办呢？张小敬有问题就去问葛佬，咱不会就去问“维基百科”啊。

>URI 可以分为 URL 和 URN，或者是 URL 和 URN 的结合体（同时具备 Locator 和 Name）。URN 就好像一个人的名字，URL 就像一个人的地址。换句话说：URN 确定了身份，URL 提供了找到它的方式。

概念清晰了吧？URI 是一个纯粹的句法结构，用于指定标识 Web 资源的字符串的各个不同部分。URL 是 URI 的一个特例，包含了定位 Web 资源的足够多的信息。URI 是统一资源标识符，而 URL 是统一资源定位符。URL 是 URI 的一种，比如：http://www.itmind.net/。但不是所有的 URI 都是 URL，因为 URI 可能包括一个子集，即统一资源名称 (URN，命名了资源但不指定如何定位资源)，比如说：mailto：qing_gee@163.com。

吧啦吧啦说这么多挺累的，来一发实例吧，用于获取 URL 的主机名和端口号。

```java
URL url = new URL("http://www.itmind.net/category/payment-selection/zhishixingqiu-jingxuan/");

System.out.println("host: " + url.getHost());
System.out.println("port: " + url.getPort());
System.out.println("uri_path: " + url.getPath());

// 输出
// host: www.itmind.net
// port: -1
// uri_path: /category/payment-selection/zhishixingqiu-jingxuan/
```

1）创建 `java.net.URL` 对象的方法非常简单，只需要一行代码。

```java
URL url = new URL(URL地址);
```

URL 对象是不可变的，因为 URL 类是 final 类型的，这样的好处就是保证它是"线程安全"的。

2）有了 `java.net.URL` 对象后，就可以获取 URL 相关的主机名、端口、路径等等。

```java
url.getHost()
url.getPort()
url.getPath()
```

### 02、什么是 URLConnection

URLConnection 是一个抽象类，代表应用程序和 URL 之间的通信链接。它的实例可用于读取和写入此 URL 引用的资源。该类提供了比 Socket 类更易于使用、更高级的网络连接抽象。

怎么获取 URLConnection 对象呢？通过 URL 对象的 `openConnection()` 方法，示例如下。

```java
URL url = new URL("http://www.itmind.net");
URLConnection connection = url.openConnection();
```

如果 URL 协议为  HTTP 的话，返回的连接为 URLConnection 的子类 HttpURLConnection。

有了 `URLConnection` 对象后，可以通过 `getInputStream()` 返回一个 InputStream，由此读取 URL 所引用的资源数据（如果读取 ASCII 文本则为 ASCII；如果读取 HTML 文件则为原始 HTML，如果读取图像文件则为二进制图片数据等）。

我们来尝试读取一下小白学堂首页的内容，代码示例如下。

```java
URL url = new URL("http://www.itmind.net");
URLConnection connection = url.openConnection();

try (InputStream in = connection.getInputStream();) {

	ByteArrayOutputStream output = new ByteArrayOutputStream();
	byte[] buffer = new byte[1024];
	int len = -1;
	while ((len = in.read(buffer)) != -1) {
		output.write(buffer, 0, len);
	}

	System.out.println(new String(output.toByteArray()));

} catch (IOException e) {
	e.printStackTrace();
}
```

可以使用 `try-with-resource` 获取 `InputStream`，该类实现了 `AutoCloseable` 接口，可以在内容读取完毕后自动关闭输入流。

打印的内容如下图所示（部分）：

![](https://static.xmt.cn/172a38adb8e242a68cf37478cfea93b4.png)

如果你想读取某个 URL 的内容，上述方法是一个不错的方案，赶快去试试吧！

### 03、URL 和 URLConnection 的不同

URL 和 URLConnection 最大的不同在于：

- URLConnection 提供了对 HTTP 头部的访问；
- URLConnection 可以配置发送给某个 URL 的请求参数；
- URLConnection 不仅可以读取 URL 定位的资源，还可以向其写入数据。

获取 HTTP 头部的方法有以下一些：

- getContentType，返回 Content-type 头字段的值，即数据的 MIME 内容类型。若类型不可用，则返回 null。如果内容类型是文本，则 Content-type 首部可能会包含一个标识内容编码方式的字符集，例如：`Content-type:text/html; charset=UTF-8`

- getContentLength()，返回 Content-length 头字段的值，即内容的字节数。

- getContentEncoding()，返回 Content-encoding 头字段的值，即内容的编码方式（不同于字符编码方式），例如：x-gzip。

- getDate()，返回 date 头字段的值，即请求的发送时间。

- getExpiration()，返回 expires（过期时间） 头字段的值。如果返回 0，表示不过期，永远缓存。

- getLastModified()，返回 last-modified（上次修改日期） 头字段的值。

代码示例如下。

```java
URL url = new URL("http://www.itmind.net");
URLConnection connection = url.openConnection();
System.out.println(connection.getContentType());
System.out.println(connection.getContentLength());
System.out.println(connection.getContentEncoding());
System.out.println(connection.getDate());
System.out.println(connection.getExpiration());
System.out.println(connection.getLastModified());

// 输出
// text/html; charset=UTF-8
// -1
// null
// 1566886980000
// 0
// 0
```

### 04、最后

利用 URLConnection 还可以向指定的 URL 请求发送 POST 请求，但这种作法已经很少有人在用了，所以这篇文章也不打算进行介绍了。如果非常感兴趣，想自己试一试，可以在公众号后台回复「666」获取我们为你精心准备的高清带书签的 PDF——《Java 网络编程（第4版）》，第 7 章对此进行了详细的介绍。