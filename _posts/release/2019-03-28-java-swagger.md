---
layout: post
category: java
title: Swagger2边写代码边写文档
tagline: by 子悠
tags: 
  - java
published: true
---

作为一个开发人员最怕的就是写文档了，但是要想成为一个合格的程序员，写好文档也是一个必备的技能。开发中我们经常要写接口服务，既然是服务就要跟别人对接，那难免要写接口文档，那么如何”优雅“的写接口文档呢？本文介绍一个在写代码的过程就可以写完接口文档的工具——Swagger2（江湖人称丝袜哥😂）

<!--more-->

下文基于 SpringBoot + Maven 介绍 Swagger2 的使用
### 加入依赖

```java

 <!--Swagger-ui配置-->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>

```


### 编写Swagger全局配置

```java

package com.ziyou.test.admin.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.ParameterBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.schema.ModelRef;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Parameter;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.List;

/**
 * <br>
 * <b>功能：</b><br>
 * <b>作者：</b>@author 子悠<br>
 * <b>日期：</b>2019-03-28 23:35<br>
 * <b>详细说明：</b>Swagger配置类，
 * UI地址：       http://127.0.0.1:8084/swagger-ui.html
 * JSON文档地址   http://127.0.0.1:8084/v2/api-docs
 * <br>
 */
@EnableSwagger2
@Configuration
public class Swagger2Config {

    @Bean
    public Docket createRestApi() {
        //header中增加 token_id参数，没有可以去掉
        ParameterBuilder token = new ParameterBuilder();
        List<Parameter> pars = new ArrayList<Parameter>();
        token.name("token_id").description("user token")
                .modelRef(new ModelRef("string")).parameterType("header")
                .required(false).build();
        pars.add(token.build());
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // 指定controller存放的目录路径
                .apis(RequestHandlerSelectors.basePackage("com.ziyou.test.admin.controller"))
                .paths(PathSelectors.any())
                .build()
                .globalOperationParameters(pars);
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                // 文档标题
                .title("这里是文档标题")
                // 文档描述
                .description("这里是文档描述")
                .termsOfServiceUrl("这里可以增加需求文档 wiki 地址")
                .version("v1")//定义版本号
                .build();
    }

}

```

### Controller 编写

1. 类上面增加`@Api `注解

```

@Api(value = "后台管理", tags = "Administrator - 后台主入口")
public class MainController  {}

```

2. 方法上增加`@ApiOperation`注解

```

 @ApiOperation(value = "用户登录——可用", httpMethod = "POST", response = ResultMsg.class)
    public ResultMsg login(@RequestBody LoginBean loginBean) throws Exception {}
//value: 表示描述
//httpMethod: 支持的请求方式
// response: 返回的自定义的实体类

```

3. 方法参数 Model 属性自定义设置

```
package com.ziyou.test.admin.bean;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

@ApiModel(value = "用户对象", description = "用户对象model")
public class LoginBean {

    @ApiModelProperty(value = "用户登录账户", name = "loginName", required = true, example = "admin")
    private String loginName;//   账号
    @ApiModelProperty(value = "用户登录密码", name = "loginPwd", required = true, example = "123456")
    private String loginPwd;//   密码
    @ApiModelProperty(hidden = true)
    private String salt;//   盐值

    public String getLoginName() {
        return this.loginName;
    }
    public void setLoginName(String loginName) {
        this.loginName=loginName;
    }
    public String getLoginPwd() {
        return this.loginPwd;
    }
    public void setLoginPwd(String loginPwd) {
        this.loginPwd=loginPwd;
    }
    public String getSalt() {
        return this.salt;
    }
    public void setSalt(String salt) {
        this.salt=salt;
    }
}

```

> @ApiModel注解用在 model 类上
> @ApiModelProperty 用于 model 的属性上
> - 可以定义是否 required, example, hidden, value, name等参数
> - hidden 为 true 在页面上就不会显示在字段

### 界面使用
1. 启动项目，打开项目文档地址：http://ip:port/swagger-ui.html
![](/assets/images/2019/java/image_ziyou/swagger1.jpg)

2. 如图显示的内容。还可以添加 header 参数，以及在网页上进行测试。
![](/assets/images/2019/java/image_ziyou/swagger2.jpg)

### 增加密码
1. 接口文档我们往往需要让有权限的人查看，所以我们可以根据 Spring-Security增加账号密码管理。
2. 增加依赖

```java

<!--Swagger-ui 密码配置配置-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

```

3. 配置文件中增加账号密码配置

```java

Spring:
  security:
    basic:
      path: /swagger-ui.html
      enabled: true
    user:
      name: admin
      password: 123456

# swagger-ui.html账号密码配置信息，根据版本不同，可能需要如下方式
#security.basic.path=/swagger-ui.html
#security.basic.enabled=true
#security.user.name=admin
#security.user.password=123456

```

4. 增加了依赖和账号密码后重启项目，再次打开文档地址就要去输入账号和密码了
![](/assets/images/2019/java/image_ziyou/swagger3.jpg)
输入对应的账号和密码就可以登录了。

### 小坑
1. 如果在启动的时候出现如下错误，是因为新版的 Swagger2需要高版本的 guava

```

***************************
APPLICATION FAILED TO START
***************************

Description:

An attempt was made to call the method com.google.common.collect.FluentIterable.concat(Ljava/lang/Iterable;Ljava/lang/Iterable;)Lcom/google/common/collect/FluentIterable; but it does not exist. Its class, com.google.common.collect.FluentIterable, is available from the following locations:

    jar:file:/Users/silence/Work/maven-repo3/com/google/guava/guava/19.0/guava-19.0.jar!/com/google/common/collect/FluentIterable.class

It was loaded from the following location:

    file:/Users/silence/Work/maven-repo3/com/google/guava/guava/19.0/guava-19.0.jar

Action:

Correct the classpath of your application so that it contains a single, compatible version of com.google.common.collect.FluentIterable

```

2. 解决方案增加如下配置即可

```

<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>20.0</version>
</dependency>

```

3. Security 过滤特定路径
    1.  针对非文档路径的其他路径，我们需要进行过滤，采用如下方式。

 ```

package com.ziyou.test.admin.config;

import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {


    /**
     * 取消 security 对接口的拦截，只在 swaggerui 进行拦截
     *
     * @param web
     * @throws Exception
     */
    @Override
    public void configure(WebSecurity web) throws Exception {
        super.configure(web);
        //这里填写需要过滤的路径
        web.ignoring().antMatchers("/api/v1/admin/**");
    }
}


```

### 小结
Swagger2 让开发人员在编写代码的时候就完成了接口文档，方便快捷。而且随项目一起部署，不用担心丢失，需要的时候只要打开项目地址就可以了，十分方便。
在使用 Swagger2写完接口文档后，还可以直接导出 JSON 文件，访问路径`http://ip:port/v2/api-docs`可以看到JSON 文档。生成的 JSON 文档可以配合 YAPI 或者 POSTMAN 使用都是可以的。

