---
layout: post
categories: Java
title:  微信支付、支付宝最全接入指引，看完立刻就可以上手！
tagline: by 小黑
published: true
tags: 
  - 小黑
---

Hello，大家好，我是阿粉~

前端时间阿粉在公司接手了一个支付项目，这个项目接入了微信、支付宝。这个项目开发下来，阿粉可是完完整整体验了一下微信、支付宝开发流程，也踩了一些坑。

最近正好看到有些小伙伴想接入微信、支付宝，但是不知道如何开发，所以阿粉就给大家总结一下微信支付、支付宝接入开发流程。

<!--more-->

## 总体流程

### 明确使用支付方式

**首先我们需要明确我们需要使用的支付方式。**

因为微信支付、支付宝中存在很多支付方式，不同支付方式对应的不同的业务场景，开发对接复杂度也不太一样。

以微信支付举例，其总共提供如下几种的支付方式。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkahzkc8zoj31vw0road6.jpg)

这几种支付方式对应的不同的业务场景，比如说：

- 付款码支付：商家使用扫码枪或其他扫码机具扫描用户出示的付款码，适合于线下商超收款
- Native支付：商家在系统中按微信支付协议生成支付二维码，用户扫码拉起微信收银台，确认并完成付款
- JSAPI 支付：用户在商家公众号内下单，输入密码支付，完成支付，适合于在线购物的场景
- APP 支付：用户在商家的APP中下单，跳转到微信中完成支付，支付完后跳回到商家APP内，展示支付结果
- H5 支付：用户在手机浏览器中下单，然后跳转到微信中完成支付
- 小程序支付：用户打开商家小程序下单，输入支付密码并完成支付后，返回商家小程序

如果你还是有点迷茫，不知道对接哪种支付方式，可以看下微信支付的接入指引，网址如下：

https://pay.weixin.qq.com/static/applyment_guide/applyment_index.shtml

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkahwya8k7j31b20u07a0.jpg)

这个网站里面列举上面这几种支付方式业务场景，你可以从中选择适合自己的业务场景。

### 接入角色

**第二我们需要明确以哪种身份接入。**

不同的身份将会使用不同的开发文档，微信支付、支付宝支持以下三类身份接入：

- 普通商户、企业
- 服务商
- 银行服务商

服务商是指有技术开发能力的第三方开发者为普通商户提供微信支付技术开发、营销方案，即服务商可在微信支付开放的**服务商高级接口**的基础上，为商户完成支付申请、技术开发、机具调试、活动营销等全生态链服务。

而银行服务商适合于银行类单位，其将会使用银联、网联提供的接口接入。

服务商与银行服务商可以说比较特殊了，一般企业都是以普通商户接入微信支付、支付宝。

所以本文后续流程将会以**普通商户**接入展开。

### 接入流程

当我们确定完上面两个基础条件后，我们就可以开始微信支付的接入流程，这个过程分为以下三步：

- **进件**
- **参数配置**
- **对接开发**

对于我们技术同学来讲，其实我们只需要参与后面两步，所以我们重点介绍后面两步。
第一步进件一般由市场或者财务等部门同学负责，不过在一些小公司，技术同学可能需要包办整个流程，所以这里也简单介绍一下。

## 进件

有的同学可能我当初一样，刚看到『进件』这个词，是不是很陌生，不理解这个词是什么意思。

其实进件的意思，就是将相关所需要的材料提交给微信支付、支付宝。

一般需要的材料如下：

- 营业执照

- 对公账户

- 法定代表人/个体户经营者证件

- 特殊资质证明材料

- 等等

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkb5r07bypj30cu0hgdhv.jpg)

首先我们需要明确自己公司主体类型，以微信支付为例，其仅面向**企业、个体工商户、政府及事业单位、民办非企业、社会团体、基金会**类型商户开放。

不同的主体的类型所需要的材料文件如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkb5tzfy28j30u0247e7u.jpg)


目前微信支付、支付宝都支持自主进件，在相应的网站上就提交相关资料即可。

当我们提交完材料，我们还需要进行账户验证，微信支付、支付宝将会往上面填的对公账户打入一笔小额金额，当我们收到以后，我们需要将在网站填入这个金额。

这一步的目的，主要是为了校验对公账户的真实性。完成这一步之后，进件的流程的基本完成，我们进入参数配置这一步。

## 参数配置

微信支付与支付宝参数配置又比较大区别，所以这里我们这里分别介绍一下。

### 微信支付

当我们完成上面的商户进件配置之后，其实我们只完成第一步。

由于商户的微信支付交易发起依赖于公众号、小程序、移动应用（即 **APPID** ）与微信支付商户号（**MCHID**）的绑定关系， 所以，还需在签约后登录对应 **APPID** 平台完成绑定关系确认。

如果此时还没有申请，我们还需要在相应的微信平台进行申请。这里不得不吐槽一下微信，没有一个统一申请 appid 的地方，公众号，移动应用申请在不同的网站。

这里总结一下公众号，小程序等的注册地址：

- 公众号/小程序：微信公众平台(https://mp.weixin.qq.com/)
- APP/PC 网站：微信开放平台(https://open.weixin.qq.com/)
- 企业微信：企业微信管理平(https://work.weixin.qq.com)

当注册完公众号等账号之后，我们还需要将微信支付商户号与申请公众号、小程序等进行绑定，这里放一张官方的流程指引图，详情可以点击：https://kf.qq.com/faq/1801116VJfua1801113QVNVz.html

![来源：https://kf.qq.com/faq/1801116VJfua1801113QVNVz.html](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkb6ezrxkej30u02vy7wh.jpg)

当完成上面的流程之后，接下去我们需要设置 API密钥，然后下载 API 证书。

API 证书获取流程参考：https://kf.qq.com/faq/161222NneAJf161222U7fARv.html

API 密钥设置流程参考：https://kf.qq.com/faq/180830UVRZR7180830Ij6ZZz.html

当完成上面这些步骤之后，我们将会有以下商户参数：

- APPID
- 微信支付商户号，mch_id
- API密钥
- API 证书
- Appsecret

上面这些参数我们需要妥善保管，切勿外传，以免造成资金损失。

### 支付宝

支付宝设置可能没有微信支付那么复杂，首先我们需要在支付宝开放平台（open.alipay.com），在开发者中心栏目下创建应用。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkb78juopoj31hc0i0ad1.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkb79iuj0hj30og0abmz2.jpg)

第二步我们需要在创建的应用中添加相应的支付功能，比如说需要使用扫码支付，那么需要使用当面付。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkb7cqxopoj31hc0q1wgk.jpg)



第三步，我们需要配置当前应用的公私钥信息。

首先我们需要生成一对 RSA2 公私钥，这里我们可以使用支付宝的官方工具生成。

工具下载地址：https://opendocs.alipay.com/open/291/105971

生成之后，我们需要在支付宝网站上配置支付公钥。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkb7jpuusaj30me0gc753.jpg)

这个需要注意了，一旦公钥配置完成，将不支持查看，只支持重新修改公钥，这样我们需要重新生成一对公私钥。

当应用公钥设置完成之后，我们就也可以查看支付宝的公钥，这个公钥信息，我们也需要保管起来。

此时我们手上就有支付宝的参数信息，作用如下：

- appid
- 应用私钥：适用于接口参数加密
- 支付宝公钥：适用于支付宝返回参数验签

上面我们需要注意应用公钥与支付宝公钥区别，应用公钥是需要给支付宝的，一旦在支付宝网站上设置完成，我们就用不到这个应用公钥。

完成这一步之后，我们基本上已经完成开发的参数配置，接下去就是开发。

基本开发完成之后，我们需要在支付宝处上架这个应用，上架时根据应用选择的支付能力，需要进行相应的签约。

其实这个签约就是我们上面讲到进件，需要填写商户各种资料。

## 对接开发

上面讲了这么多，终于聊到开发这一步。其实，对接开发真的不难，我们只要根据相应的文档组装接口参数，然后发送给微信支付宝即可。

微信支付文档地址：https://pay.weixin.qq.com/wiki/doc/api/index.html

支付宝文档地址：https://opendocs.alipay.com/open/194/105203

这里我们就不聊具体的源码实现方式，大家参考相应的接口文档，这里主要分享一些经验，防止大家踩坑。

### 同步接口与异步接口

同步接口：在我们调用 API 接口之后就可以同步返回具体扣款结果。

异步接口：我们调用 API 接口成功，这一步只是返回一些参数，我们需要在前台页面拼接这些参数，然后唤醒微信支付、支付宝，最后完成支付。支付结果将会通过异步通知给应用程序。

举个例子，微信支付、支付宝 API 接口中，仅**付款码支付**为同步接口，这里引用一下官方的时序图：

![付款码支付免密流程时序图](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkbo2i4q62j30nq0jxmxw.jpg)

而剩余的其他接口都为异步接口，这里引用一下微信支付 Native 时序图：

![原生支付接口模式一时序图](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkbo3mv1zoj30nu0pvtaa.jpg)



对于异步接口来说，我们除了需要开发 API 接口发送程序，我们还需要开发一个接收微信支付、支付宝的接受程序。

### 安全性

微信支付与支付宝涉及的所有 API 接口都需要进行加签，获取签名值，这一步目的主要是为了安全性。

微信支付加签方式如图所示:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkbodks7d9j30u60u0tmf.jpg)

在这一步我们就需要使用到上一个流程提到的微信支付的 AP I密钥。

除此之外，微信支付的退款接口还有点特殊，我们还需要使用其下发给我们的 API 证书。

另外查看微信支付的文档的时候，阿粉还发现，微信支付的接口还有一个 V3 版。这个版本参数格式支持 JSON，新老接口的区别如下：

![V2 VS V3](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkbo7a0w75j30uy0lodhu.jpg)

阿粉上面提到都是基于 V2 版本，如果你在搜索引擎搜索，可以看到现在市面上大部分还是在使用 V2 版本的接口。

如果大家对接 V3 版本的接口，那一定要记得查看 V3 版本的API 接口，地址如下：

https://pay.weixin.qq.com/wiki/doc/apiv3/wxpay/pages/Overview.shtml

另外对于异步通知的内容，我们一定要进行**验签**，验签的目的就是在于防止数据泄漏导致出现“假通知”，造成资金损失。

这里需要注意了，异步通知的可能会发送多次，我们的程序一定要注意幂等控制。

建议可以采用如下的流程：

**当收到通知进行处理时，首先检查对应业务数据的状态，判断该通知是否已经处理过，如果没有处理过再进行处理，如果处理过直接返回结果成功。在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱。**

### 开源工具

我们在开发过程中，可以使用一些开源工具，这些工具帮我们封装加签、验签的步骤，以及网络发送程序，极大减少开发的复杂度。

支付宝开源工具主要由支付宝官方提供，使用介绍：

https://opendocs.alipay.com/open/54/103419

其提供 Maven 依赖，如下：

```xml
<!-- https://mvnrepository.com/artifact/com.alipay.sdk/alipay-sdk-java -->
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-sdk-java</artifactId>
    <version>4.10.170.ALL</version>
</dependency>

```

对于微信来说，目前没看到官方提供的相关的工具类，这里推荐一个 Github 拥有超多 Star 的项目-**WxJava**

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkbp2z1vrrj318c0b80uj.jpg)

项目地址如下：

https://github.com/Wechat-Group/WxJava

这个开源工具，框架初始化设置 API秘钥，证书等，后续请求 API 接口只要填入相关参数即可，其他加签、验签内部程序将会自动完成。



### 沙箱环境

测试过程中，如果正式应用还没有被官方审批，我们可以先使用沙箱环境进行测试。测试完成之后，我们只要替换相关参数就可以了。

这里需要注意了，目前只有支付宝提供沙箱测试环境，微信支付暂时没有。

支付宝沙箱的地址如下：

https://openhome.alipay.com/platform/appDaily.htm

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20201104/0081Kckwly1gkctlujllbj31h10u0ahh.jpg)

同样的沙箱环境我们也需要设置相应的应用公钥，以及下载对应的支付宝公钥。

另外沙箱环境的请求地址与正式环境不同，请求地址为：

https://openapi.alipaydev.com/gateway.do

同时我们还需要使用沙箱版**支付宝 APP**才能进行正常测试。



## 最后

总体来说，阿粉觉得微信支付、支付宝的开发难度不大，相关参数只要安装接口文档拼接就可以了。

最大的难度可能在于加签、验签上，这一点新手可能比较容易踩坑。

好了，今天的文章就到此为止了，大家下篇文章见~


## 帮助文档

1. [商户号与 APPID 绑定](https://kf.qq.com/faq/1801116VJfua1801113QVNVz.html)
2. [微信接入指引](https://pay.weixin.qq.com/static/applyment_guide/applyment_index.shtml)
3. [进件](https://pay.weixin.qq.com/wiki/doc/api/mch_xiaowei.php?chapter=28_1&index=1)
4. [商户申请接入时，如何确认绑定关系？](https://kf.qq.com/faq/180910QZzmaE180910vQJfIB.html)

