---
layout: post
categories: Java
title: 龙岭迷窟真的这么好看？今天我们就用 Java 爬取豆瓣数据好好分析一下！|原创
tagline: by 小黑
published: true
tags: 
  - 小黑
---

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge5q2u8huvj314w0h8q64.jpg)

首图来自最近热播的『鬼吹灯之龙岭迷窟』，看过上一部『鬼吹灯之怒晴湘西』同学应该能看懂这个笑点。 潘粤明老师上一部还是卸岭魁首陈玉楼，这一部摇身一变成了胡八一。

<!--more-->

好了，不扯剧情了，说会正题。鬼吹灯之龙岭迷窟』现在豆瓣评分 **8.2**，可以说是鬼吹灯系列的评分最高的一部了。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge11kk8pxmj31680j2aky.jpg)

那今天阿粉就爬取一波豆瓣短评数据，结合数据分析，看一下网友们真正的评价如何。

看完这篇文章，阿粉教大家学会一个简单的爬虫。

全文知识点如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge11gei9krj30od0djjss.jpg)

## 爬虫

首先在写爬虫程序之前，我们先来看一个爬虫整个流程：

- 分析网页结构/请求接口信息
- 获取指定数据
- 数据落库

### 分析网页结构

这次我们需要获取龙岭迷窟的短评数据，包括昵称，评分，时间，以及短评内容。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge120jq28wj31360u0tgo.jpg)

那如何获取这些数据那？

这里我们就需要使用 **xpath** 定位这些数据位置。**xpath** 可以说是爬虫必备技能，它的语法不是很难，这里阿粉就不再详细介绍 Xpath 语法，小伙伴可以在下面网站自学一下：

https://www.runoob.com/xpath/xpath-tutorial.html

这里推荐大家一个 Chrome 浏览器插件 **XPath Helper** ，可以直接在上面使用 **xpath** 语法定位我们需要数据。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge12b7qbhoj32fz0u0ndd.jpg)

> 由于**神秘因素**，很多小伙伴可能没办法连上谷歌商店下载插件，小伙伴可以在公号后台回复：『xpath』 获取插件安装包

上图中我们仅仅使用一个简单 **xpath** 语句，快速定位到我们需要短评数据内容。

### 分页爬取

豆瓣短评一个页面只有 20 条数据，这就需要我们不断爬取下一页的地址，加入爬虫待爬取的地址，这样才能实现自动爬取。这里获取下一页的地址有两个办法：

- 自动爬取下一页地址
- 手动拼接下一页地址

**自动爬取下一页地址**

这里我们可以跟上面一样，使用 **xpath** 定位下一页元素。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge26vot5z8j32lg0ms44o.jpg)

然后获取下一页中 `href` 属性，该属性代表这页请求地址。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge26xe7qplj32jm0medm3.jpg)

不过这里我们需要注意，此时我们通过 **xpath** 获取的是一个相对链接，无法直接访问。我们还需在这个基础上拼接上如下地址:

```html
https://movie.douban.com/subject/30488569/comments
```

这个还是比较麻烦，不过不用担心，下面使用阿粉介绍的开源工具，开源工具将会自动将这种相对地址拼接成可以访问的地址。

**手动拼接下一页地址**

点击下一页地址，查看地址特征：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge272gql6cj31fg0pgn2u.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge272wqggpj31d60sqgrn.jpg)

可以发现地址中只有 **start** 属性不一样，并别每一页递增 20。

所以下一页的地址我们可以根据这个规则自己拼接：

```java
https://movie.douban.com/subject/30488569/comments?start=%d&limit=20&sort=new_score&status=P
```

**注意点**

由于豆瓣限制，未登录的情况下我们只能查看 **200** 条短评数据,再继续查看，将会要求登录。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge279e8ywmj31eq0oy0wk.jpg)

不过登录的情况下也不并不能查看全部数据，最多只能查看 **500** 条数据。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge2786w0rtj31730u0q85.jpg)

所以如果自己拼接下一个地址，start 只要到 500 就可以停止。

哦，对了，这里我们还需要为爬虫解决登录问题，不然我们只能爬取 **200** 条。

登录豆瓣之后，将会在 **Cookie** 中保存登录信息。我们可以手动获取 **Cookie** 信息，设置到爬虫中。

**Cookie** 获取方式：

打开 Chrome 的开发者工具，在 Network 一栏获取生成的 **Cookie** 信息。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge27f6db6wj32ly0jcguu.jpg)

当然上面方法还是比较繁琐的，小伙伴们也可以调用豆瓣登录接口，自动获取 Cookie，这里阿粉就不详细介绍，感兴趣小伙伴可以参考下这篇文章：

https://blog.csdn.net/ityard/article/details/103554071

### Webmagic

好了，上面分析这么多，下面我们开始实战环节。

我们可以通过 HttpClient 获取网页内容,然后通过 Jsoup 解析网页正文。

不过这么做，很繁琐，对于新手也很不友好。这里阿粉推荐一个开源的简单灵活爬虫框架『**Webmagic**』，完全基于 Java 开发：

https://github.com/code4craft/webmagic

具体使用教程，小伙伴可以参考如下地址：

http://webmagic.io/docs/zh/

首先我们需要实现 `PageProcessor` 接口，实现爬虫的配置、页面元素的抽取和链接的发现，代码如下：

```java
public class DouBanCommentsProcessor implements PageProcessor {

    // 设置出错之后重试次数，休眠时间一定要设置，防止频繁抓取，豆瓣封禁 IP
    private Site site = Site.me().setRetryTimes(3).setTimeOut(10000).setSleepTime(5000);

    @SneakyThrows
    @Override
    public void process(Page page) {
        List<String> commentList = page.getHtml().xpath("//div[@class=\"comment\"]").all();
        List<DouBanCommentDO> douBanCommentDTOS = new ArrayList<>();
        // 首先获取 短评的整个 div ，然后循环遍历获取里面的内容
        for (String comment : commentList) {
            Html commentHtml = Html.create(comment);
            // 用户名
            String user = commentHtml.xpath("//h3/span[2]/a/text()").get();
            // 分数
            String star = commentHtml.xpath("//h3/span[2]/span[2]/@class").get();
            // 评分时间
            String date_time = commentHtml.xpath("//h3/span[2]/span[3]/@title").get();
            String date = commentHtml.xpath("//h3/span[2]/span[3]").get();
            // 短评内容
            String comment_text = commentHtml.xpath("//p/span/text()").get();
            DouBanCommentDO douBanCommentDTO = new DouBanCommentDO();
            douBanCommentDTO.setUser(user);
            douBanCommentDTO.setComment_text(comment_text);
            if (StringUtils.isNotBlank(date_time)) {
                // 时间格式 2020-04-01 22:17:25
                douBanCommentDTO.setComment_date_time(DateUtils.parseDate(date_time, "yyyy-MM-dd HH:mm:ss"));
            } else if (StringUtils.isNotBlank(date)) {
                douBanCommentDTO.setComment_date_time(DateUtils.parseDate(date, "yyyy-MM-dd"));
            }
            douBanCommentDTO.setStar(star);
            douBanCommentDTOS.add(douBanCommentDTO);
        }
        // 数据存储时将会通过这个 key 提取
        page.putField("comment", douBanCommentDTOS);
        // 获取下一页的连接
        String nextPageUrl = page.getHtml().xpath("//div[@id='paginator']/a[@class='next']").links().get();
        log.info("下一页地址：{}", nextPageUrl);
        if (StringUtils.isNotBlank(nextPageUrl)) {
            // 将下一页地址存入 page 这样才会继续爬取
            page.addTargetRequest(nextPageUrl);
        }
    }

    @Override
    public Site getSite() {
        site.setUserAgent("User-Agent\": 'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0)");
        site.addHeader("Cookie", "your own cookie");
        return site;
    }

}
```

这里需要注意几点:

- **Site** 一定要设置合理的休眠时间，频繁抓取，豆瓣可能会封禁 IP
- **Site** 添加 **Cookie** 信息请使用 `addHeader` 方法设置，`addCookie` 只能添加一对信息，不适合当前正常场景。

Webmagic 默认将会把爬取结果输出到控制台中，这里我们通过实现 `Pipeline`  接口，将结果保存到数据库中。

```java
public class DoubanDaoPipeline implements Pipeline {

    @Autowired
    DouBanMapper douBanMapper;

    @Override
    public void process(ResultItems resultItems, Task task) {
        List<DouBanCommentDO> douBanCommentDOS = resultItems.get("comment");
        if (CollectionUtils.isNotEmpty(douBanCommentDOS)) {
            for (DouBanCommentDO douBanCommentDO : douBanCommentDOS) {
                douBanMapper.insertBankCard(douBanCommentDO);
            }
        }

    }
}
```

> 数据存储使用 mybatis，这里不再贴具体代码。

以上就是 Webmagic 主要代码逻辑，接着我们启动爬虫，开始爬取数据。

```java
String startURL = "https://movie.douban.com/subject/30488569/comments?limit=20&sort=new_score&status=P";
Spider.create(processor)
        .addUrl(startURL)
        .addPipeline(doubanDaoPipeline)
   			// 设置爬取线程数
        .thread(1)
        //启动爬虫
        .run();
```

启动爬虫的代码，我们需要设置合理的**线程数**，这里仅设置为 1，单线程爬取。

> 完整代码请参考：https://github.com/9526xu/douban_spider

## 数据分析

爬虫获取到数据之后，我们就可以根据这些数据进行分析。这里阿粉使用一个开源 BI 工具-『**MetaBase**』，创建分析数据图表。

> **MetaBase** 是一个免费轻量级的开源工具，配置简单，安装十分方便。这个工具最大的特点，操作门槛低，容易上手，不会写 sql 也可以使用。

> **MetaBase** 安装教程可以参考这篇文章：https://zhuanlan.zhihu.com/p/52085283

这里我们通过 **MetaBase** 编辑器条件设置：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge3dgdckzoj31mf0u0ncb.jpg)

就可以快速得出一个评论数量图表:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge3ddhn9y3j31cr0u01kx.jpg)

从图中我们可以看出一些规律，**四月一号**与**四月八号**评论数最多，这是因为腾讯视频每周三会放出最新的剧集。大家看完最新剧情，然后上豆瓣评论吐槽，评论数合情合理。随着时间推荐，热度下降，评论数也递减。

接着我再来看下这部剧这几天平均星级的情况

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge3dj1znp0j31hm0u01kx.jpg)

从上图可以看到，这部剧大部分的用户评论都在四星以上，每天平均星级没有太大变化。这间接说明了这部剧的质量还是可以的。

> 上图中，四月六号打分人数过少，影响较大，所以导致曲线骤降。

## 词云

最后我们用词云展示评论区的热门词汇。这里我们使用一个开源工具：

**kumo**:https://github.com/kennycason/kumo

**kumo** 代码实现不是很难，github 上有很多示例。

实现代码如下：

```java
final FrequencyAnalyzer frequencyAnalyzer = new FrequencyAnalyzer();
// 设置词频
frequencyAnalyzer.setWordFrequenciesToReturn(600);
frequencyAnalyzer.setMinWordLength(2);
// 中文分词
frequencyAnalyzer.setWordTokenizer(new ChineseWordTokenizer());
// 过滤一些预期助词等
List stopWorlds = FileUtils.readLines(new File("stopworld/cn_stopwords.txt"));
// 过滤词
frequencyAnalyzer.setStopWords(stopWorlds);

List<DouBanCommentDO> commentDOS = douBanMapper.queryAll();

List<String> comments = commentDOS.stream().map(DouBanCommentDO::getComment_text).collect(Collectors.toList());

final List<WordFrequency> wordFrequencies = frequencyAnalyzer.load(comments);

final Dimension dimension = new Dimension(500, 312);
final WordCloud wordCloud = new WordCloud(dimension, CollisionMode.PIXEL_PERFECT);
wordCloud.setPadding(2);
// 词语背景图
wordCloud.setBackground(new PixelBoundryBackground("pic/whale_small.png"));
// 设置颜色
wordCloud.setColorPalette(new ColorPalette(new Color(0xC73939), new Color(0x10E71C), new Color(0x0984D5), new Color(0x73CB0E), new Color(0x40D3F1), new Color(0xFFC97915, true)));
wordCloud.setFontScalar(new LinearFontScalar(10, 40));
wordCloud.build(wordFrequencies);
wordCloud.writeToFile("worldCloud/test.png");
```

效果图如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge4iyinmofj30dw08oq4z.jpg)

## 总结

今天阿粉通过爬取龙岭迷窟的短评，给大家过了一下爬虫的基本流程。简单爬虫的实现真的不是很难，大家可以照着例子可以实现一下爬取豆瓣其他数据。

好了，今天就写到这了，如果还想深入了解爬虫，可以在公众号给阿粉留言，或者直接在星球提问哦~

感谢大家的关注与点赞，正因为你们，阿粉每天才有更加大动力去写好每篇文章~

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge4j6sjzffj307d06u3yh.jpg)

## 最后（点个赞吧！）

完成这篇文章的时候，『鬼吹灯之龙岭迷窟』已经播出全部剧情。听说潘粤明老师已经接了 5 部鬼吹灯系列的，期待创造一个属于鬼吹灯的宇宙和时代。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge5tfdppx6g303m01zdju.gif)

哦，对了，潘粤明老师你也千万别忘了另一个破案宇宙还在等你那~

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200425/007S8ZIlly1ge5thxyjj9j31hc0hsgw0.jpg)





