---
layout: post
categories: Java
title: 阿粉教你如何使用爬虫来对比某东上的数据
tagline: by 懿
tags: 
  - 懿
---

自从阿粉经历过上次的大数据杀熟事件之后，明显感觉现在的平台对于用户非常的不友好呀，只要你高频的搜索某些关键词的同时，却往往是越对比，直接就买在了最高峰，就和买股票一样，每次总感觉能抄底，殊不知买在了天台。于是阿粉想了个办法，把所有的数据扒拉下来，我自己做对比，也不去搜索了，省的平台上总是根据我的搜索内容去进行推荐。

<!--more-->

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/10-15/1.jpg)

### Java如何做爬虫

大家在想到爬虫的时候，一定想说，爬虫，这东西不是学Python的人员才能做的么？我们Java能做呢？阿粉想告诉大家的是，可以，Java语言这么多年，历时这么久，怎么可能没有这些内容呢，于是阿粉就开始了学习了 Java 的爬虫道路。

#### Jsoup

阿粉在介绍这个类之前，肯定先得说说我们通常看到的内容是由什么组成的，现在比如说我们做开发的都知道，至少我们在电脑端访问某东，某宝的数据的时候，他们给我们反馈的数据都是通过 HTML 来进行展示的，比如说这个样子：

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/10-15/2.jpg)

在开发的肯定都是知道，这些都是些什么意思，阿粉在这里我们就不再进行详细的介绍，说这个 HTML 到底是个啥东西了，阿粉需要介绍的是 Jsoup ,然后告诉大家怎么使用 Jsoup 这个类爬取京东的数据。

正如官方文档所给我们提示的内容，怎么去解析一段 HTML 代码 ：

```
String html = "<html><head><title>First parse</title></head>"
  + "<body><p>Parsed HTML into a doc.</p></body></html>";
  
Document doc = Jsoup.parse(html);

```

而这个 Document是什么呢？我们可以输出一下看一眼，顺带着看看源码解释，毕竟嘛，开发人员不看这个类是干嘛的，就不是个合格的程序员不是，

输出内容：
```
<html>
 <head>
  <title>First parse</title>
 </head>
 <body>
  <p>Parsed HTML into a doc.</p>
 </body>
</html>

```

其实可以看出这里，Document实际上是给我们输出了一个新的文档，而且是整理之后的，相当于为之后的分析 HTML 做了专业的准备。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/10-15/3.jpg)

而我们在看源码的注释的时候，不难看出，Jsoup不单单是能解析我们给的这个字符串，还可以是一个URL，也可以是一个文件。

它把我们给他的 HTML 字符串转换成了一个对象，这个对象就是我们上面看到的 Document，然后我们就可以顺利成章的去使用 Document 对象里面的元素了。

上面是解析字符串，那我们看下面这个解析 URL 的存在：

```
 public static void main(String[] args) {
        try {
            Document doc = Jsoup.connect("https://www.jd.com/?cu=true&utm_source=baidu-pinzhuan&utm_medium=cpc&utm_campaign=t_288551095_baidupinzhuan&utm_term=0f3d30c8dba7459bb52f2eb5eba8ac7d_0_f38cf584e9fb4328a3e0d2bb515e1458").get();
            String title = doc.title();
            System.out.println(title);
        }catch (IOException e){
            e.printStackTrace();
        }
    }
    
```

大家执行以下的话，就一定能够看到这个 title 到底是什么，而结果是这个样子的：

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/10-15/4.jpg)

和我们在百度搜索的时候是不是不太一样了，因为这个是进入之后的主页。

#### Element 

而在我们看源码的时候，我们能清晰的看到，Document 是继承了 Element 的类，那么必然可以调用 Element 里面的方法，比如说：

```
getElementById(String id); //是不是有点眼熟，像不像Js里面的ID选择器

getElementsByTag(String tagName)；// 通过标签来选择

getAllElements()；//获取所有的Element的元素
```

关于方法，阿粉就不再一一的进行叙述了，大家有兴趣的可以去看看官方文档，或者去看看这个源码，包名送上 `package org.jsoup.nodes`

有人肯定开始烦了，说阿粉，你就别介绍了，那你说了太多废话了，赶紧介绍爬京东，好的，这就开始，

我们在爬取之前肯定先分析京东的网址，比如说我搜索硬盘：

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/10-15/5.jpg)

下面就出来了一堆数据，而我们则需要解析的就是在 HTML 种最有用的那一部分，比如：

```
<div class="p-price">
    <strong class="J_54994027563" data-done="1">
        <em>￥</em><i>879.00</i>
    </strong>
</div>

```

在这里我们就记下了这个价格，然后我们去找我们要的名字

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/10-15/6.jpg)

看，`p-name`就是我们需要的名字，那么我们就可以写代码了。

```
     //这是京东的搜索网址，我们把这个keyword关键词提取出来，注意中英文，中文要处理一下
     String url = "https://search.jd.com/Search?keyword=" + keyword;
     url = url + "&enc=utf-8";
     Document document = Jsoup.parse(new URL(url), 40000);
    
     //我们先找这个 List，然后一层一层的遍历
     Element element = document.getElementById("J_goodsList");
     Elements elements = element.getElementsByTag("li");
     for (Element el : elements) {
         String img = el.getElementsByTag("img").eq(0).attr("source-data-lazy-img");
         String price = el.getElementsByClass("p-price").eq(0).text();
         String title = el.getElementsByClass("p-name").eq(0).text();
         String shop = el.getElementsByClass("p-shop").eq(0).text();
            System.out.println("=========================");
            System.out.println("标题:" + title);
            System.out.println("图片url:" + img);
            System.out.println("店铺:" + shop);
            System.out.println("价格:" + price);
      } 

```

大家看执行的效果图：

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/10-15/7.jpg)

如果你还有兴趣的话，你可以直接在for循环里面新建一个对象，弄一个List集合，然后在最后的的时候，执行一下插入数据库的方法，这样是不是就能完整的把数据都保存下来了呢？

### 写在最后

为什么阿粉介绍爬取某东，而不去爬取某宝，因为某东是允许你爬取我的数据的，他没有做任何的反扒机制，而某宝则不行，正是应了那句话

“某东：我卖的是真货，你说我骗人，我赔给你，某宝：我卖假货，但是我不承认，你能拿我怎么办？某多多：亲，假一赔十，结果发过来11件假货，”

阿粉想问大家，你有没有被大数据杀熟过呢？
