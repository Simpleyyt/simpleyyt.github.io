---
layout: post  
title: 神奇！明明是 Socket，被我玩成了 http！
tagline: by 江南一点雨
categories: Java  
tag: 
    - Java
---

现在，我们已经充分了解了 HTTP 和 Socket 的关系，也了解了 HTTP 报文的格式，为了让小伙伴能够加深对这两个概念的理解，本文我们来看看如何利用 Socket 模拟 HTTP 请求。如果小伙伴们对 HTTP 和 Socket 的关系、HTTP 报文格式尚不熟悉的话，可以参考前面的文章[ Http 和 Socket 到底是哪门子亲戚？](https://mp.weixin.qq.com/s/r1WlVG8cwaN3vXzoTU8bgQ)。    

由于 HTTP 是基于 TCP 协议的应用层协议，因此我们可以用更为底层的方式来访问 HTTP 服务，即直接使用 Socket 完成 HTTP 的请求和响应。我们前面说过，HTTP 的任务就是完成数据的包装， Socket 提供了网络的传输能力，所以我们只需要按照 HTTP 报文的格式来组装数据，然后利用 Socket 将数据发送出去，就能得到回应。  

## POST 请求上传数据

假设我现在有一个数据接口 `http://localhost/hello`，该接口每次接收一个参数 name ，调用成功之后返回给用户一个 `hello:name` 字符串，那我们用 Socket 来实现这样一个 HTTP 请求。  

首先，我们要先组装出 HTTP 的请求头，如下(如果小伙伴对下面这个请求头有疑问，请复习[ Http 和 Socket 到底是哪门子亲戚？](https://mp.weixin.qq.com/s/r1WlVG8cwaN3vXzoTU8bgQ)一文)：  

```
POST /hello HTTP/1.1
Accept:text/html
Accept-Language:zh-cn
Host:localhost

name=张三
```

我这里为了简单，只添加了三个请求头，然后我们通过 Socket 将上面这个字符串发送出去：

```java
Socket socket = new Socket(InetAddress.getByName("localhost"), 80);
OutputStream os = socket.getOutputStream();
String data = "name=张三";
int dataLen = data.getBytes().length;
String contentType = "Content-Length:" + dataLen+"\r\n";
os.write("POST /hello HTTP/1.1\r\n".getBytes());
os.write("Accept:text/html\r\n".getBytes());
os.write("Accept-Language:zh-cn\r\n".getBytes());
os.write(contentType.getBytes());
os.write("Host:localhost\r\n".getBytes());
os.write("\r\n".getBytes());
os.write(data.getBytes());
os.write("\r\n".getBytes());
os.flush();
```

我在 Serlvet 中接收这个请求并作简单处理，如下：  

```java
BufferedReader br = new BufferedReader(new InputStreamReader(req.getInputStream(),"UTF-8"));
StringBuffer sb = new StringBuffer();
String str;
while ((str = br.readLine()) != null) {
    sb.append(str).append("\r\n");
}
System.out.println("sb:"+sb.toString());
resp.setContentType("text/html;charset=utf-8");
PrintWriter out = resp.getWriter();
out.write(sb.toString());
out.flush();
out.close();
```  

然后通过 Socket 中的输入流我就能拿到响应结果，如下：  

```java
BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
StringBuffer sb = new StringBuffer();
String str;
while ((str = br.readLine()) != null) {
    sb.append(str).append("\r\n");
}
System.out.println(sb.toString());
```  

响应结果如下：  

```
HTTP/1.1 200 
Content-Type: text/html;charset=utf-8
Transfer-Encoding: chunked
Date: Sun, 03 Dec 2017 10:46:52 GMT

name=张三
```

这是一个简单的通过 POST 请求下载文本的案例。接下来我们再来一个 GET 请求下载图片的案例，来加深对 Socket 的理解。  

## GET 请求下载图片

这个实际上也不难，但是要实现图片的下载需要我们首先熟悉HTTP响应的数据格式，不熟悉的小伙伴可以阅读 [ Http 和 Socket 到底是哪门子亲戚？](https://mp.weixin.qq.com/s/r1WlVG8cwaN3vXzoTU8bgQ)一文。　　

下载图片，响应头是文本文件，响应数据是二进制文件，我们要想办法通过空行将这两块数据分开，分别处理。为了解决这个问题，我首先提供一个工具类，这个工具类用来实现一行一行的解析字节流，如下：　　

```java
public class BufferedLineInputStream {
    private InputStream is;

    public BufferedLineInputStream(InputStream is) {
        this.is = is;
    }

    /**
     * @param buf    将数据读入到byte数组中
     * @param offset 数组偏移量
     * @param len    数组长度
     * @return 返回值表示读取到的数据长度 -1表示数据读取完毕，0表示其他异常情况
     */
    public int readLine(byte[] buf, int offset, int len) throws IOException {
        if (len < 1) {
            return 0;
        }
        //count用来统计已经向数组中存储了多少数据
        int count = 0, c;
        while ((c = is.read()) != -1) {
            buf[offset++] = (byte) c;
            count++;
            //如果一行已经读完或者数组已满
            if (c == '\n' || count == len) {
                break;
            }
        }
        return count > 0 ? count : -1;
    }
}
```   

然后将响应中的头信息和图片分别保存在不同的文件中，数据解析的核心思路就是一行一行读取响应数据，当遇到 `\r\n` 表示头信息已经读取完了，要开始读取二进制数据了，二进制数据读取到之后，将之保存成图片即可。核心代码如下：  

```java
fos = new FileOutputStream("E:\\333.png");
pw = new PrintWriter(new OutputStreamWriter(new FileOutputStream("E:\\222.txt")));
socket = new Socket(InetAddress.getByName("localhost"), 80);
OutputStream out = socket.getOutputStream();
out.write("GET /1.png HTTP/1.1\r\n".getBytes());
out.write("Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n".getBytes());
out.write("Accept-Language:zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3\r\n".getBytes());
out.write("Host:localhost\r\n".getBytes());
out.write("\r\n".getBytes());

BufferedLineInputStream blis = new BufferedLineInputStream(socket.getInputStream());
int len;
byte[] buf = new byte[1024];
while ((len = blis.readLine(buf, 0, buf.length)) != -1) {
    String s = new String(buf, 0, len);
    System.out.println(s);
    if (s.equals("\r\n")) {//表示头信息读取完毕
        break;
    }
}
//开始解析二进制的图片数据
while ((len = blis.readLine(buf, 0, buf.length)) != -1) {
    fos.write(buf, 0, len);
}
```  

OK，Socket 模拟 HTTP 请求我们就先说到这里，两个案例，希望能够加深小伙伴对 Socket 和 HTTP 的理解。