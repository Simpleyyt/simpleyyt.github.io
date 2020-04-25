---
layout: post
title: 你的个税APP退税还好么？从个税APP看并发
catgories: java
tags:
  - 懿
---

> hello~各位读者好，我是鸭血粉丝（大家可以称呼我为「阿粉」），今天是周六了，大家的个税都完成退税操作了么？

<!--more-->

前几天真的是堪称个税APP史上最魔幻的一天，一个抖音发的退税的视频，愣是把个税APP给搞得各种出小差，然后经过了2天的抢修，终于大家可以如愿以偿的进行退税的操作了，而阿粉在看到视频的时候，视频的的浏览量已经几万了，而阿粉在意的确实并发，这次并发操作，实实在在的体验到什么叫容易出问题了。

关于并发，咱们就来掰扯一下这个并发的事情，高并发这个词汇一直是大公司面试的时候问的内容，比如某东，某宝，某度，某讯，不说这些超级大厂，就是一些做教育的互联网公司，都会问这个。

比如面试官很多就会问：

面试官：

1. 在服务器的带宽，cpu，内存，I/O 都正常的情况下。程序没有接收到请求，你会从哪方面查原因？

2. 那假如有 1 百万 /s 的请求，nginx 的队列满了，后面的请求只能排队怎么办？

3. 你之前的项目并发量在多少呢？你们是怎么处理这些内容的？

如果面试的时候你说你对这个不熟悉，那么基本属于“凉凉”，那么阿粉给大家总结了之前的关于并发的文章，想让大家看看，对大家有没有帮助呢？

解决并发的方式有很多种，阿粉给大家区分出了一些内容，文章放给大家，

[并发程序设计，懂不懂？](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247484489&idx=2&sn=e1a340189ffcca10ffb79d8a43b3f50a&chksm=fd0164feca76ede8c46c139d82161cf722f661d16368a24a1495cc0c0b5439d23f0f27381467&token=1081080156&lang=zh_CN#rd)

[Java：并发不易，先学会用](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247483720&idx=1&sn=11b34c31fea5cebac55656551d6a9e07&chksm=fd0161ffca76e8e9f80d77594198186908038cb8fc66949133d123d41b70a97dc312f53c8209&token=1081080156&lang=zh_CN#rd)

[不知道如何实现服务的动态发现？快来看看 Dubbo 是如何做到的](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485418&idx=2&sn=af1722112a042cacfb6d582a6d59a644&chksm=fd01675dca76ee4b19da5e434855d7ead27dbb7c1b6a79112a505c91c1c8aed2db6a8771af7f&token=1081080156&lang=zh_CN#rd)

[并发神器之 CopyOnWriteArrayList](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486705&idx=2&sn=13f81906a0ca3c771c93b58b060a2c98&chksm=fd016c46ca76e55091846f75b40c549caddeb646e672f77de9038a8ce53ac43cb94c01279598&token=1081080156&lang=zh_CN#rd)

[微服务异步架构---MQ之RocketMQ](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247483842&idx=1&sn=1ef0c7d2e8d3b4d5507c625a7051f65a&chksm=fd016175ca76e863e868edd69788aa9a19ab693a0780723a66d6324dcd61a9db8e8df65c0b06&token=1081080156&lang=zh_CN#rd)

### 1.使用缓存

[Redis 小白入门以及基础搭建](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486997&idx=1&sn=aefa3bc0ef9c80d7d50736b89095e1e7&chksm=fd016ea2ca76e7b4f3e67532effe802a4f2becb63b3cb634e2b387290719e8c79562139a487e&token=1081080156&lang=zh_CN#rd)

[Redis 中的 “SOS”，不对，是 SDS](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486487&idx=1&sn=dcecbdc97785c5c9b18a91c5e0e6b099&chksm=fd016ca0ca76e5b615beaa56ba64a5d707a41ee579610d76049ac507ff0c935d1141139cb7ab&token=1121137251&lang=zh_CN&scene=21#wechat_redirect)

[一文带你了解 Redis 的发布与订阅的底层原理](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486427&idx=1&sn=6ed4e315f9ae87acd54fa72c2c48de28&chksm=fd016b6cca76e27a8ac468361f3f89a670e10c164f876396d277539f2e24763ba32b10355cd7&token=1121137251&lang=zh_CN&scene=21#wechat_redirect)

[一文看懂 Redis 的内存回收策略和 Key 过期策略](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486090&idx=1&sn=ccd917cb07e91f44722ad57fd8b9d3f1&chksm=fd016a3dca76e32b0685fb85668ec7f8ea03b972eebe32bbb9ec38e30e31d79e9e8f5631adc1&token=1121137251&lang=zh_CN&scene=21#wechat_redirect)

[来看看我们是怎么玩儿 Redis 的](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485784&idx=2&sn=9f10fcc792076f48efe4caf440f9addd&chksm=fd0169efca76e0f93c042d7b1d24931606be64575bab56d89aa3b9322fbb347f979f3a12ebfa&token=1121137251&lang=zh_CN&scene=21#wechat_redirect)

[还不知道 Redis 分布式锁的背后原理？还不赶快学习一下](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247487506&idx=1&sn=a8b529bf0f4d2f0a4da3c86d099ed245&chksm=fd0170a5ca76f9b3582a55a3f97e07c1c599358e08f361ecf0461bd27fc1d68b75461cf29f8e&token=1081080156&lang=zh_CN#rd)

### 2.应用程序和静态资源文件进行分离

### 3.集群与分布式

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/04-04/1.jpg)

[分布式下必备神器之分布式锁](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247484721&idx=1&sn=e70194f7832dd1900a508e17b3279fad&chksm=fd016586ca76ec9032afbb56d51a5df1d342404202e5ff6e29e6b56fc565439634e7c8a6fbe8&scene=21#wechat_redirect)

[你知道什么是分布式事务吗](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485784&idx=1&sn=d66f4319ec407bbf49055b50e6fa8d3b&chksm=fd0169efca76e0f9a792c1a085bea26f5a205e879f9288128f5f4d23740d417a385a2e8719dc&token=1656096563&lang=zh_CN&scene=21#wechat_redirect)

[分布式面试题：服务注册中心如何选型？](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485402&idx=1&sn=43c7a49b1705c81b483c6bd133d7aca8&chksm=fd01676dca76ee7b1acb97a05113c600549265d554bca5858b04a57babe3073edb18a9d03d2a&token=1081080156&lang=zh_CN#rd)

[告诉你Dubbo 的底层原理，面试不再怕！](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485337&idx=1&sn=5977d7995b607c1a539143107ab6210a&chksm=fd01672eca76ee3828a172bb373ec1da614e7167d663d0b16fe7ee0464319bdd78174b825190&token=1081080156&lang=zh_CN#rd)

[还不知道事务消息吗？这篇文章带你全面扫盲！](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247487317&idx=1&sn=b85f7036084731efccabfc70200a20ae&chksm=fd016fe2ca76e6f484e247dde9750fd536fdfb249ee92df8c2f54b060ecfe3ae0c18ef175645&token=1081080156&lang=zh_CN#rd)

[谈谈对分布式事务的一点理解和解决方案](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247484136&idx=1&sn=98d9350aae3ace35b4ad697392c22820&chksm=fd01625fca76eb49f6a75bd259b3873343b9c3d3a5e45e334ab30dc6199abccc7bf408cff6ec&token=1081080156&lang=zh_CN#rd)

[面试必备的分布式事物方案](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247483963&idx=1&sn=a0802da8add5b5d197eab667486dc8b6&chksm=fd01628cca76eb9a1b67addd3f2519b0dbc78d168a2261468495a73b0c1bf18876354b8ca525&token=1081080156&lang=zh_CN#rd)

[如果有人问你 Dubbo 中注册中心工作原理，就把这篇文章给他](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485153&idx=1&sn=decd4ee3e88c4e35c30e7299dfba57dc&chksm=fd016656ca76ef4099fef37539e993daf0d02c411bd4008b517bcefc3886ffc81750ebc4d52e&token=1081080156&lang=zh_CN#rd)


### 4.反向代理和负载均衡


负载均衡其实在面试的时候，一般先不要去死命的去说定义，可以结合自己的理解，之后再做详细的阐述，说的直白一点，在服务器做集群的时候，需要有一台服务器充当调度者的角色，而这个服务器在接受到请求的时候，他作为调度者，可以给其他的服务器进行合理的分配，性能高的我给你多一点，性能略低的我给你少一点，当然说这么一点肯定是不够的，接下来就请看下面的文章呢！


[因为没有网关，我的服务器被 DDoS 了](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247487169&idx=1&sn=1b59676f3a32e32367256317f320aef6&chksm=fd016e76ca76e760bb21e143a6bfa17b08943baf504e32751807080d6b96d7ebcdbb7cc8398c&token=1081080156&lang=zh_CN#rd)

[面试官：负载均衡的算法你了解不？](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486408&idx=1&sn=b3595f8cba397856a64872a9bf34e4bf&chksm=fd016b7fca76e269d4d3c91f400ca7c2a9c84224659da802a506e8645da2d5680ddc75f3ba48&token=1081080156&lang=zh_CN#rd)

[面试官问：HTTP 的负载均衡你了解么？你不是说了你们用的Nginx么？说一下把。](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486281&idx=1&sn=d733bd69be95124d3c1d6a506954dc91&chksm=fd016bfeca76e2e8d6a988d66b98f8e84e66b378e33ad980981580c8c5c6aa2cd47d675174da&token=1081080156&lang=zh_CN#rd)

[手把手教你，使用 Nginx 搭配 Tomcat 实现负载均衡!](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485581&idx=1&sn=d79a57d4b606c6297095f130d765ce30&chksm=fd01683aca76e12c948f9614bfa798e680b8285a94d442f3c8c080567e66711f826875163a10&token=1081080156&lang=zh_CN#rd)

[关于 Nginx，你还在背诵那些培训机构教给你的内容么？](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485426&idx=1&sn=ce7d9b6124634fe870dbdeb21f3c6995&chksm=fd016745ca76ee53db7e647330872cce6f8a47b0a0267590a6de479020df52508b3659db2dd9&token=1081080156&lang=zh_CN#rd)

[关于 HTTP 代理，你还需要了解这些，不然面试你是过不去的！](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486049&idx=1&sn=c3e68d8e6c74a701975a9af366a1f61f&chksm=fd016ad6ca76e3c0511c27d45b62f6fc997516b82f0a1e42e0f09b7fe9408be69142339a4829&token=1081080156&lang=zh_CN#rd)

而且阿粉也不光为大家准备了关于这个并发的，还有大家最喜欢的数据结构系列，也放送给大家，毕竟金三银四面试季，虽然几年的行情不太好，但是不能阻挡我们对涨工资的热情不是么？关于数据结构请看下面：

[如果有人再问你 Java IO，把这篇文章砸他头上](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486423&idx=1&sn=aa9ee8044961bb5dd7a345f434336cb8&chksm=fd016b60ca76e276770260d3045dcc2b7999cbebdb0963fcedfe23eb518ae8055fcee5eca8ab&token=1656096563&lang=zh_CN#rd)

[面试必问之 ConcurrentHashMap 线程安全的具体实现方式](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486419&idx=1&sn=28bfd3b9516ace6fe9b5d09d88d48fdf&chksm=fd016b64ca76e272607fe84e50464a1fc4f01e2386d94ac944cffa6033611c1ee2f60dd3c7e6&token=1656096563&lang=zh_CN#rd)

[HashMap 在多线程环境下操作可能会导致程序死循环](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486396&idx=1&sn=df0e0a10016384fe54d5989c613ee35b&chksm=fd016b0bca76e21d2e51477bb337cfcf8df234f460ff394080eec9594143a3b83120f3658f56&token=1656096563&lang=zh_CN#rd)

[你应该知道的 PriorityQueue ——深入浅出分析 PriorityQueue](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486212&idx=1&sn=289e40f4a330ad3e7d239d01db54c4f7&chksm=fd016bb3ca76e2a5037a0ae6d980725045fc6e17d46d7c653a1d5f2f1ed753f60e83fe89484a&token=1656096563&lang=zh_CN#rd)

[集合系列- 深入浅出分析 ArrayDeque](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486190&idx=1&sn=840b4b230eac4d647d30781c7ffdb2ba&chksm=fd016a59ca76e34f14a1a24630f13f507d2253cefa9f8b4d84d8b1f613ceb932bae3c95aa575&token=1656096563&lang=zh_CN#rd)

[深入浅出的分析 Set集合](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486044&idx=1&sn=ee2a29c755c5d9bbf08d1bc06324d157&chksm=fd016aebca76e3fd1edb75d3c98c8910a79562f2ca7ddcc69c7522510cedde36d7aa9490621f&token=1656096563&lang=zh_CN#rd)

[深入浅出的分析 Properties](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247486017&idx=1&sn=a6cdd98d21a502b70ecb4173adf49f5d&chksm=fd016af6ca76e3e0082545f5ca58974036664232ee2554ec4a99e90af4a1b996e5b44925aa34&token=1656096563&lang=zh_CN#rd)

[带你深入浅出的分析 HashTable 源码](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485988&idx=1&sn=c5c540346a0229ae87d2457661468297&chksm=fd016a93ca76e385feddb1310f034c9dc1929f96397893ad3c717d6f1ce508be214332957c10&token=1656096563&lang=zh_CN#rd)

[深入浅出分析 IdentityHashMap](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485803&idx=2&sn=3851ae92c49d9c5fb7f4c599416115ed&chksm=fd0169dcca76e0ca46bc0d226be0547b7738429cf83b81497a1cb2f2474cf6f142f825c8ae59&token=1656096563&lang=zh_CN#rd)

[深入浅出分析 LinkedHashMap](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485799&idx=2&sn=ad2ef8e0e32e648667f2c46774e8d744&chksm=fd0169d0ca76e0c67e08f37a1bfd482949adaee1f8a2c4f2851db4584f1043b67be2e151c3a2&token=1656096563&lang=zh_CN#rd)

[深入浅出的分析 TreeMap](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485767&idx=2&sn=7c1e45bf62b2505a2ec28b03d8e01283&chksm=fd0169f0ca76e0e65c85c83f9608cfffeb0317cb1d396f696e42058ba8fd23f2753e40e5c47b&token=1656096563&lang=zh_CN#rd)

[深入浅出分析 Collection 中的 List 接口](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485682&idx=1&sn=0def1ba786b3c4ccd1b828af0b1797d4&chksm=fd016845ca76e153b4a015e0746bce0e65f5f5d12fd6d0301fdbf96b111bb9181c3598501cd7&token=1656096563&lang=zh_CN#rd)

[一文看懂 HashMap 中的红黑树实现原理](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485642&idx=1&sn=87686ada46171453fbf0775e9f79eb8e&chksm=fd01687dca76e16bcacbbe2f002adb7fadaac9781d77c4860b39f58cbb5f59d0963b0b17722f&token=1656096563&lang=zh_CN#rd)

[掌握 HashMap 看这一篇文章就够了](https://mp.weixin.qq.com/s?__biz=MzU3NzczMTAzMg==&mid=2247485627&idx=1&sn=d1caf10fcd41fd93a3ef713bf30b52c2&chksm=fd01680cca76e11a38b79b0216ff4a0923104f193b361fcd7bdb1e2a3b09c4422c894903a927&token=1656096563&lang=zh_CN#rd)

在这个金三银四跳槽的好时间，阿粉祝愿大家找到一个心仪的工作，信心十足的开启新的生活。









