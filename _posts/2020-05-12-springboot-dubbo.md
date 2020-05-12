---
layout: post
categories: springboot
title: 利用springboot+dubbo，构建分布式微服务，全程注解开发
tags: 
  - 炸鸡可乐
---

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。

<!--more-->

### 一、先来一张图
说起 Dubbo，相信大家都不会陌生！阿里巴巴公司开源的一个高性能优秀的服务框架，可以使得应用可通过高性能的 RPC 实现服务的输出和输入功能，同时可以和 Spring 框架无缝集成。

![Dubbo 架构图](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/01.jpg)

节点角色说明：

* **Provider**：暴露服务的服务提供方
* **Consumer**：调用远程服务的服务消费方
* **Registry**：服务注册与发现的注册中心
* **Monitor**：统计服务的调用次数和调用时间的监控中心
* **Container**：服务运行容器

### 二、实现思路
今天，我们以一个**用户选择商品下订单**这个流程，将其拆分成3个业务服务：**用户中心、商品中心、订单中心**，使用 Springboot + Dubbo 来实现一个小 Demo！

服务交互流程如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/02.jpg)

**本文主要是介绍 Springboot 与 Dubbo 的框架整合以及开发实践，而真实的业务服务拆分是一个非常复杂的过程，比我们介绍的这个要复杂的多，上文提到的三个服务只是为了项目演示，不必过于纠结为什么要这样拆分**！

好了，废话也不多说了，下面我们开撸！

* 1.在虚拟机创建 4 台 centos7，任意选择一台安装 zookeeper
* 2.构建微服务项目并编写代码
* 3.在 centos7 上部署微服务
* 4.远程服务调用测试

### 三、zookeeper安装
> 在使用 Dubbo 之前，我们需要一个注册中心，目前 Dubbo 可以选择的注册中心有 zookeeper、Nacos 等，一般建议使用 zookeeper！

首先在安装 Zookeeper 之前，需要安装并配置好 JDK，本机采用的是Oracle Java8 SE。

* 安装JDK（已经安装可以忽略）

```java
yum -y install java-1.8.0-openjdk
```

* 查看java安装情况

```java
java -version
```

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/03.jpg)

* JDK安装完成之后，下载安装Zookeeper

```
#创建一个zookeeper文件夹
cd /usr
mkdir zookeeper

#下载zookeeper-3.4.14版本
wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz

#解压
tar -zxvf zookeeper-3.4.14.tar.gz
```

* 创建数据、日志目录

```
#创建数据和日志存放目录
cd /usr/zookeeper/
mkdir data
mkdir log

#把conf下的zoo_sample.cfg备份一份，然后重命名为zoo.cfg
cd conf/
cp zoo_sample.cfg zoo.cfg
```

* 配置zookeeper

```
#编辑zoo.cfg文件
vim zoo.cfg
```

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/04.jpg)

* 启动Zookeeper

```
#进入Zookeeper的bin目录
cd zookeeper/zookeeper-3.4.14/bin

#启动Zookeeper
./zkServer.sh start

#查询Zookeeper状态
./zkServer.sh status

#关闭Zookeeper状态
./zkServer.sh stop
```

出现如下信息，表示启动成功！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/05.jpg)

### 四、项目介绍

* springboot版本：2.1.1.RELEASE
* zookeeper版本：3.4.14
* dubbo版本：2.7.3
* mybtais-plus版本：3.0.6
* 数据库：mysql-8
* 构建工具：maven
* 服务模块：用户中心、商品中心、订单中心

### 五、代码实践
#### 5.1、初始化数据库
首先在 mysql 客户端，创建3个数据库，分别是：`dianshang-user`、`dianshang-platform`、`dianshang-business`。

* 在 dianshang-user 数据库中，创建用户表 tb_user，并初始化数据

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/06.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/07.jpg)

* 在 dianshang-platform 数据库中，创建商品表 tb_product，并初始化数据

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/08.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/09.jpg)

* 在 dianshang-platform 数据库中，创建订单表 tb_order、订单详情表 tb_order_detail

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/10.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/11.jpg)

#### 5.2、创建工程
数据库表设计完成之后，在 IDEA 下创建一个名称为`dianshang`的`Springboot`工程。

最终的目录如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/12.jpg)

目录结构说明：

* dianshang-common：主要存放一些公共工具库，所有的服务都可以依赖使用
* dianshang-business：订单中心，其中`api`模块主要是提供dubbo服务暴露接口，`provider`模块是一个`springboot`项目，提供服务处理操作
* dianshang-user：用户中心，其中`api`模块和`provider`模块，设计与之类似
* dianshang-platform：商品中心，其中`api`模块和`provider`模块，设计与之类似


在父类`pom`文件中加入`dubbo`和`zookeeper`客户端，所有依赖的项目都可以使用。

```xml
<!-- lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.4</version>
    <scope>provided</scope>
</dependency>

<!-- Dubbo Spring Boot Starter -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.7.3</version>
</dependency>
<!-- 因为使用的是 zookeeper 作为注册中心，所以要添加 zookeeper 依赖 -->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.13</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--使用curator 作为zookeeper客户端-->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>
```
**温馨提示**：小编在搭建环境的时候，发现一个坑，工程中依赖的`zookeeper `版本与服务器的版本，需要尽量一致，例如，本例中`zookeeper`服务器的版本是`3.4.14`，那么在依赖`zookeeper`文件库的时候，也尽量保持一致，如果依赖`3.5.x`版本的`zookeeper`，项目在启动的时候会各种妖魔鬼怪的报错！

#### 5.3、创建用户中心项目
在 IDEA 中，创建`dianshang-user`子模块，并依赖`dianshang-common`模块
```xml
<dependencies>
    <dependency>
        <groupId>org.project.demo</groupId>
        <artifactId>dianshang-common</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```
同时，创建`dianshang-user-provider`和`dianshang-user-api`模块。

* dianshang-user-api：主要对其他服务提供接口暴露
* dianshang-user-provider：类似一个web工程，主要负责基础业务的`crud`，同时依赖`dianshang-user-api`模块


##### 5.3.1、配置dubbo服务
在`dianshang-user-provider`的`application.yml`文件中配置`dubbo`服务，如下：
```yml
#用户中心服务端口
server:
  port: 8080
#数据源配置
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: "jdbc:mysql://localhost:3306/dianshang-user"
      username: root
      password: 111111
#dubbo配置
dubbo:
  scan:
    # 包名根据自己的实际情况写
    base-packages: org.project.dianshang.user
  protocol:
    port: 20880
    name: dubbo
  registry:
    #zookeeper注册中心地址
    address: zookeeper://192.168.0.107:2181
```
##### 5.3.2、编写服务暴露接口以及实现类
在`dianshang-user-api`模块中，创建一个`UserApi`接口，以及返回参数对象`UserVo`！
```java
public interface UserApi {

    /**
     * 查询用户信息
     * @param userId
     * @return
     */
    UserVo findUserById(String userId);
}
```
其中`UserVo`，需要实现序列化，如下：
```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
public class UserVo implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 用户ID
     */
    private String userId;

    /**
     * 用户中文名
     */
    private String userName;
}
```
在`dianshang-user-provider`模块中，编写`UserApi`接口实现类，如下：
```java
@Service(interfaceClass =UserApi.class)
@Component
public class UserProvider implements UserApi {

    @Autowired
    private UserService userService;

    @Override
    public UserVo findUserById(String userId) {
        QueryWrapper<User> queryWrapper = new QueryWrapper<User>();
        queryWrapper.eq("user_id",userId);
        User source = userService.getOne(queryWrapper);
        if(source != null){
            UserVo vo = new UserVo();
            BeanUtils.copyProperties(source,vo);
            return vo;
        }
        return null;
    }
}
```
**其中的注解`@Service`指的是`org.apache.dubbo.config.annotation.Service`下的注解，而不是`Spring`下的注解哦**！

接着，我们继续创建商品中心项目！
#### 5.4、创建商品中心项目
与用户中心项目类似，在 IDEA 中，创建`dianshang-platform`子模块，并依赖`dianshang-common`模块
```xml
<dependencies>
    <dependency>
        <groupId>org.project.demo</groupId>
        <artifactId>dianshang-common</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```
同时，创建`dianshang-platform-provider`和`dianshang-platform-api`模块。

* dianshang-platform-api：主要对其他服务提供接口暴露
* dianshang-platform-provider：类似一个web工程，主要负责基础业务的`crud`，同时依赖`dianshang-platform-api`模块

##### 5.4.1、配置dubbo服务
在`dianshang-platform-provider`的`application.yml`文件中配置`dubbo`服务，如下：
```yml
#用户中心服务端口
server:
  port: 8081
#数据源配置
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: "jdbc:mysql://localhost:3306/dianshang-platform"
      username: root
      password: 111111
#dubbo配置
dubbo:
  scan:
    # 包名根据自己的实际情况写
    base-packages: org.project.dianshang.platform
  protocol:
    port: 20881
    name: dubbo
  registry:
    #zookeeper注册中心地址
    address: zookeeper://192.168.0.107:2181
```
##### 5.4.2、编写服务暴露接口以及实现类
在`dianshang-platform-api`模块中，创建一个`ProductApi`接口，以及返回参数对象`ProductVo`！
```java
public interface ProductApi {

    /**
     * 通过商品ID，查询商品信息
     * @param productId
     * @return
     */
    ProductVo queryProductInfoById(String productId);
}
```
其中`ProductVo`，需要实现序列化，如下：
```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
public class ProductVo implements Serializable {

    private static final long serialVersionUID = 1L;

    /**商品ID*/
    private String productId;

    /**商品名称*/
    private String productName;

    /**商品价格*/
    private BigDecimal productPrice;
}
```
在`dianshang-platform-provider`模块中，编写`ProductApi`接口实现类，如下：
```java
@Service(interfaceClass = ProductApi.class)
@Component
public class ProductProvider implements ProductApi {

    @Autowired
    private ProductService productService;

    @Override
    public ProductVo queryProductInfoById(String productId) {
        //通过商品ID查询信息
        Product source = productService.getById(productId);
        if(source != null){
            ProductVo vo = new ProductVo();
            BeanUtils.copyProperties(source,vo);
            return vo;
        }
        return null;
    }
}
```
接着，我们继续创建订单中心项目！
#### 5.5、创建订单中心项目
与商品中心项目类似，在 IDEA 中，创建`dianshang-business`子模块，并依赖`dianshang-common`模块
```xml
<dependencies>
    <dependency>
        <groupId>org.project.demo</groupId>
        <artifactId>dianshang-common</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```
同时，创建`dianshang-business-provider`和`dianshang-business-api`模块。

* dianshang-business-api：主要对其他服务提供接口暴露
* dianshang-business-provider：类似一个web工程，主要负责基础业务的`crud`，同时依赖`dianshang-business-api`模块


##### 5.5.1、配置dubbo服务
在`dianshang-business-provider`的`application.yml`文件中配置`dubbo`服务，如下：
```yml
#用户中心服务端口
server:
  port: 8082
#数据源配置
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: "jdbc:mysql://localhost:3306/dianshang-business"
      username: root
      password: 111111
#dubbo配置
dubbo:
  scan:
    # 包名根据自己的实际情况写
    base-packages: org.project.dianshang.business
  protocol:
    port: 20882
    name: dubbo
  registry:
    #zookeeper注册中心地址
    address: zookeeper://192.168.0.107:2181
```
##### 5.5.2、编写服务暴露接口以及实现类
在`dianshang-business-api`模块中，创建一个`OrderApi`接口，以及返回参数对象`OrderVo`！
```java
public interface OrderApi {

    /**
     * 通过用户ID，查询用户订单信息
     * @param userId
     * @return
     */
    List<OrderVo> queryOrderByUserId(String userId);
}
```
其中`OrderVo`，需要实现序列化，如下：
```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
public class OrderVo implements Serializable {

    private static final long serialVersionUID = 1L;

    /**订单ID*/
    private String orderId;

    /**订单编号*/
    private String orderNo;

    /**订单金额*/
    private BigDecimal orderPrice;

    /**下单时间*/
    private Date orderTime;
}
```
在`dianshang-business-provider`模块中，编写`OrderApi`接口实现类，如下：
```java
@Service(interfaceClass = OrderApi.class)
@Component
public class OrderProvider implements OrderApi {

    @Autowired
    private OrderService orderService;

    @Override
    public List<OrderVo> queryOrderByUserId(String userId) {
        QueryWrapper<Order> queryWrapper = new QueryWrapper<Order>();
        queryWrapper.eq("user_id",userId);
        List<Order> sourceList = orderService.list(queryWrapper);
        if(!CollectionUtils.isEmpty(sourceList)){
            List<OrderVo> voList = new ArrayList<>();
            for (Order order : sourceList) {
                OrderVo vo = new OrderVo();
                BeanUtils.copyProperties(order, vo);
                voList.add(vo);
            }
            return voList;
        }
        return null;
    }
}
```
至此，3个项目的服务暴露接口已经开发完成！接下来我们来编写怎么进行远程调用！
#### 5.6、远程调用
##### 5.6.1、编写创建订单服务
在`dianshang-business-provider`模块中，编写创建订单接口之前，先依赖`dianshang-business-api`和`dianshang-user-api`，如下：

```xml
<!--商品服务接口暴露 api-->
<dependency>
    <groupId>org.project.demo</groupId>
    <artifactId>dianshang-platform-api</artifactId>
    <version>1.0.0</version>
</dependency>
<!--用户服务接口暴露 api-->
<dependency>
    <groupId>org.project.demo</groupId>
    <artifactId>dianshang-user-api</artifactId>
    <version>1.0.0</version>
</dependency>
```
在`dianshang-business-provider`模块中，编写创建订单服务，如下：
```java
@RestController
@RequestMapping("/order")
public class OrderController {

	@Autowired
    private OrderService orderService;

	@Autowired
	private OrderDetailService orderDetailService;

    @Reference(check =false)
	private ProductApi productApi;

    @Reference(check =false)
    private UserApi userApi;

	/**
	 * 新增
	 */
	@JwtIgnore
	@RequestMapping(value = "/add")
	public boolean add(String productId,String userId){
	    LocalAssert.isStringEmpty(productId,"产品Id不能为空");
        LocalAssert.isStringEmpty(userId,"用户Id不能为空");
        ProductVo productVo = productApi.queryProductInfoById(productId);
        LocalAssert.isObjectEmpty(productVo,"未查询到产品信息");
        UserVo userVo = userApi.findUserById(userId);
        LocalAssert.isObjectEmpty(userVo,"未查询到用户信息");
        Order order = new Order();
        order.setOrderId(IdGenerator.uuid());
        order.setOrderNo(System.currentTimeMillis() + "");
        order.setOrderPrice(productVo.getProductPrice());
        order.setUserId(userId);
        order.setOrderTime(new Date());
        orderService.save(order);

        OrderDetail orderDetail = new OrderDetail();
        orderDetail.setOrderDetailId(IdGenerator.uuid());
        orderDetail.setOrderId(order.getOrderId());
        orderDetail.setProductId(productId);
        orderDetail.setSort(1);
        orderDetailService.save(orderDetail);
		return true;
	}
}
```
其中的`@Reference`注解，是属于`org.apache.dubbo.config.annotation.Reference`下的注解，表示远程依赖服务。

参数`check =false`表示启动服务时，不做远程服务状态检查，这样设置的目的就是为了防止当前服务启动不了，例如用户中心项目没有启动成功，但是订单中心又依赖了用户中心，如果`check=true`，此时订单中心启动会报错！
##### 5.6.2、编写用户查询自己的订单信息
同样的，在`dianshang-user-provider`模块中，编写用户查询自己的订单信息接口之前，先依赖`dianshang-business-api`和`dianshang-user-api`，如下：

```xml
<dependency>
    <groupId>org.project.demo</groupId>
    <artifactId>dianshang-business-api</artifactId>
    <version>1.0.0</version>
</dependency>
<dependency>
    <groupId>org.project.demo</groupId>
    <artifactId>dianshang-user-api</artifactId>
    <version>1.0.0</version>
</dependency>
```
在`dianshang-user-provider`模块中，编写用户查询自己的订单信息接口，如下：
```java
@RestController
@RequestMapping("/user")
public class UserController {

	@Reference(check =false)
	private OrderApi orderApi;


    /**
     * 通过用户ID，查询订单信息
     * @param userId
     * @return
     */
    @RequestMapping("/list")
    public List<OrderVo> queryOrderByUserId(String userId){
        return orderApi.queryOrderByUserId(userId);
    }
}
```
至此，远程服务调用，编写完成！
### 六、服务测试
在将项目部署在服务器之前，咱们先本地测试一下，看服务是否都可以跑通？

* 启动用户中心`dianshang-user-provider`

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/13.jpg)

* 继续启动商品中心`dianshang-platform-provider`

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/14.jpg)

* 接着启动订单中心`dianshang-business-provider`

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/15.jpg)

最后，我们来测试一下服务接口是否为我们预期的结果？

打开浏览器，输入`http://127.0.0.1:8082/order/add?productId=1&userId=1`测试创建订单接口，页面运行结果显示正常！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/16.jpg)

我们再来看看数据库，订单是否生成？

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/17.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/18.jpg)

ok！很清晰的看到，数据已经进去了，没啥问题！

我们再来测试一下在用户中心订单查询接口，输入`http://127.0.0.1:8080/user/list?userId=1`，页面运行结果如下！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/19.jpg)

**到此，本地服务测试基本通过**！
### 七、服务器部署
**在上文中，我们介绍了服务的构建、开发和测试，那如何在服务器端部署呢**？

首先，修改各个项目的`application.yml`文件，将其中的数据源地址、dubbo注册中心地址修改为线上能联通的地址，然后在`dianshang`目录下使用`maven`工具对整个工程执行如下命令进行打包！
```java
mvn clean install
```
也可以在 IDEA 环境下，通过`maven`配置`clean install`命令执行打包。

将各个项目`target`目录下的`dianshang-user-provider.jar`、`dianshang-platform-provider.jar`、`dianshang-business-provider.jar`拷贝出来。

分别上传到对应的服务器目录，本服务器采用的是 CentOS7，总共4台服务器，其中一台部署`zookeeper`，另外三台部署三个微服务项目。

登录服务器，输入如下命令，确保`JDK`已经安装完成！
```
java -version
```
关闭所有服务器的防火墙，放行端口访问！
```
#关闭防火墙
systemctl stop firewalld.service

#禁止开机启动
systemctl disable firewalld.service
```

* 启动用户中心服务，日志信息输出到`service.log`（虚拟机ip：192.168.0.108）

```java
nohup java -jar  dianshang-user-provider.jar > service.log 2>&1 &
```

* 启动商品中心服务，日志信息输出到`service.log`（虚拟机ip：192.168.0.107）

```java
nohup java -jar  dianshang-platform-provider.jar > service.log 2>&1 &
```

* 启动订单中心服务，日志信息输出到`service.log`（虚拟机ip：192.168.0.109）

```java
nohup java -jar  dianshang-business-provider.jar > service.log 2>&1 &
```

打开浏览器，输入`http://192.168.0.109:8082/order/add?productId=1&userId=1`测试创建订单接口，页面运行结果显示正常！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/20.jpg)

我们再来测试一下在用户中心订单查询接口，输入输入`http://192.168.0.108:8080/user/list?userId=1`，页面运行结果如下！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-dubbo/21.jpg)

**很清晰的看到，输出了2条信息，第二条订单是在测试环境服务器生成的，第一条是本地开发环境生成的**。

到此，服务器部署基本已经完成！

如果是生产环境，可能就需要多台`zookeeper`来保证高可用，至少2台服务器来部署业务服务，通过负载均衡来路由！

### 八、总结
整片文章比较长，主要是围绕 springboot + dubbo 的整合，通过注解开发实现远程服务调用像传统的 springmvc 开发一样轻松，当然还可以通过`xml`配置方式实现dubbo服务的调用，这个会在后期去介绍。

同时也介绍了服务器的部署，从中可以看出，开发虽然简单，但是由于分布式部署，如何保证服务高可用成了开发人员头等工作任务，所以，分布式微服务开发虽然开发简单，但是如何确保部署的服务高可用？运维方面会带来不少的挑战！
### 九、参考
1、[apache - dubbo - 官方文档](http://dubbo.apache.org/zh-cn/docs/user/preface/architecture.html)
