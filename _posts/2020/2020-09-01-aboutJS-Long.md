---

layout: post
categories: java
title: 踩坑记：后端返回的Long型参数，前端js取值时精度丢失
tagline: by 炸鸡可乐
tags: 
  - 炸鸡可乐
---

最近几天一直在改造工程，采用雪花算法生成主键ID，突然踩到一个天坑，前端 JavaScript 在取 Long 型参数时，参数值有点不太对！

<!--more-->

### 一、问题描述
最近在改造内部管理系统的时候， 发现了一个巨坑，就是前端 JS 在获取后端  Long 型参数时，出现精度丢失！

起初，用 postman 模拟接口请求，都很正常，但是用浏览器请求的时候，就出现问题了！

* 问题复现

```java
@RequestMapping("/queryUser")
public List<User> queryUser(){
    List<User> resultList = new ArrayList<>();
    User user = new User();
	//赋予一个long型用户ID
    user.setId(123456789012345678L);
    resultList.add(user);
    return resultList;
}
```
打开浏览器，请求接口，结果如下！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/aboutJS-Long/01.jpg)

用 postman 模拟接口请求，结果如下！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/aboutJS-Long/02.jpg)

刚开始的时候，还真没发现这个坑，结果当进行测试的时候，才发现前端传给后端的ID，与数据库中存的ID不一致，才发现 JavaScript 还有这个天坑！

由于 JavaScript 中 Number 类型的自身原因，并不能完全表示 Long 型的数字，在 Long 长度大于`17`位时会出现精度丢失的问题。

当我们把上面的用户 ID 改成 19 位的时候，我们再来看看浏览器请求返回的结果。
```
//设置用户ID，位数为19位
user.setId(1234567890123456789l);
```
浏览器请求结果！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/aboutJS-Long/03.jpg)

当返回的结果超过17位的时候，后面的全部变成0！


### 二、解决办法
遇到这种情况，应该怎么办呢？

* 第一种办法：在后台把 long 型改为String类型，但是代价有点大，只要涉及到的地方都需要改
* 第二种办法：使用工具进行转化把 long 型改为String类型，这种方法可以实现全局转化（推荐）
* 第三种办法：前端进行处理（目前没有很好的办法，不推荐）

因为项目涉及到的代码非常多，所以不可能把 long 型改为 String 类型，而且使用 Long 类型的方法非常多，改起来风险非常大，所以不推荐使用！

最理想的方法，就是使用aop代理拦截所有的方法，对返回参数进行统一处理，使用工具进行转化，过程如下！

#### 2.1、Jackson 工具序列化对象
我们可以使用`Jackson`工具包来实现对象序列化。

* 首先在 maven 中添加必须的依赖

```
<!--jackson依赖-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.8</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.8</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.8</version>
</dependency>
```

* 编写一个转化工具类`JsonUtil`

```java
public class JsonUtil {

    private static final Logger log = LoggerFactory.getLogger(JsonUtil.class);

    private static ObjectMapper objectMapper = new ObjectMapper();
    private static final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss";

    static {
        // 对象的所有字段全部列入
        objectMapper.setSerializationInclusion(JsonInclude.Include.ALWAYS);
        // 取消默认转换timestamps形式
        objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        // 忽略空bean转json的错误
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
        //设置为东八区
        objectMapper.setTimeZone(TimeZone.getTimeZone("GMT+8"));
        // 统一日期格式
        objectMapper.setDateFormat(new SimpleDateFormat(DATE_FORMAT));
        // 反序列化时,忽略在json字符串中存在, 但在java对象中不存在对应属性的情况, 防止错误
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // 序列换成json时,将所有的long变成string
        objectMapper.registerModule(new SimpleModule().addSerializer(Long.class, ToStringSerializer.instance).addSerializer(Long.TYPE, ToStringSerializer.instance));
    }

    /**
     * 对象序列化成json字符串
     * @param obj
     * @param <T>
     * @return
     */
    public static <T> String objToStr(T obj) {
        if (null == obj) {
            return null;
        }

        try {
            return obj instanceof String ? (String) obj : objectMapper.writeValueAsString(obj);
        } catch (Exception e) {
            log.warn("objToStr error: ", e);
            return null;
        }
    }

    /**
     * json字符串反序列化成对象
     * @param str
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T strToObj(String str, Class<T> clazz) {
        if (StringUtils.isBlank(str) || null == clazz) {
            return null;
        }

        try {
            return clazz.equals(String.class) ? (T) str : objectMapper.readValue(str, clazz);
        } catch (Exception e) {
            log.warn("strToObj error: ", e);
            return null;
        }
    }

    /**
     * json字符串反序列化成对象(数组)
     * @param str
     * @param typeReference
     * @param <T>
     * @return
     */
    public static <T> T strToObj(String str, TypeReference<T> typeReference) {
        if (StringUtils.isBlank(str) || null == typeReference) {
            return null;
        }

        try {
            return (T) (typeReference.getType().equals(String.class) ? str : objectMapper.readValue(str, typeReference));
        } catch (Exception e) {
            log.warn("strToObj error", e);
            return null;
        }
    }
}
```

* 紧接着，编写一个实体类`Person`，用于测试

```java
@Data
public class Person implements Serializable {

    private static final long serialVersionUID = 1L;

    private Integer id;

    //Long型参数
    private Long uid;
    private String name;
    private String address;
    private String mobile;

    private Date createTime;
}
```

* 最后，我们编写一个测试类测试一下效果

```java
public static void main(String[] args) {
    Person person = new Person();
    person.setId(1);
    person.setUid(1111L);
    person.setName("hello");
    person.setAddress("");
    System.out.println(JsonUtil.objToStr(person));
}
```
输出结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/aboutJS-Long/04.jpg)

其中最关键一行代码，是注册了这个转换类，从而实现将所有的 long 变成 string。
```
// 序列换成json时,将所有的long变成string
SimpleModule simpleModule = new SimpleModule();
simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
simpleModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
objectMapper.registerModule(simpleModule);
```
如果想对某个日期进行格式化，可以全局设置。
```java
//全局统一日期格式
objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
```
也可以，单独对某个属性进行设置，例如对`createTime`属性格式化为`yyyy-MM-dd`，只需要加上如下注解即可。
```java
@JsonFormat(pattern="yyyy-MM-dd", timezone="GMT+8")
private Date createTime;
```
工具转化类写好之后，就非常简单了，只需要对 aop 拦截的方法返回的参数，进行序列化就可以自动实现将所有的 long 变成 string。
#### 2.2、SpringMVC 配置
如果是 SpringMVC 项目，操作也很简单。

* 自定义一个实现类，继承自`ObjectMapper `

```java
package com.example.util;

/**
 * 继承ObjectMapper
 */
public class CustomObjectMapper extends ObjectMapper {

    public CustomObjectMapper() {
        super();
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
        simpleModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
        registerModule(simpleModule);
    }
}
```

* 在 SpringMVC 的配置文件中加上如下配置

```
<mvc:annotation-driven >
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg index="0" value="utf-8" />
            <property name="supportedMediaTypes">
                <list>
                    <value>application/json;charset=UTF-8</value>
                    <value>text/plain;charset=UTF-8</value>
                </list>
            </property>
        </bean>          
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="com.example.util.CustomObjectMapper">
                    <property name="dateFormat">
						<-对日期进行统一转化->
                        <bean class="java.text.SimpleDateFormat">
                            <constructor-arg type="java.lang.String" value="yyyy-MM-dd HH:mm:ss" />
                        </bean>
                    </property>
                </bean>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```
#### 2.3、SpringBoot 配置
如果是 SpringBoot 项目，操作也类似。

* 编写一个`WebConfig`配置类，并实现自`WebMvcConfigurer`，重写`configureMessageConverters`方法

```java
/**
 * WebMvc配置
 */
@Configuration
@Slf4j
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    /**
     *添加消息转化类
     * @param list
     */
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> list) {
        MappingJackson2HttpMessageConverter jsonConverter = new MappingJackson2HttpMessageConverter();
        ObjectMapper objectMapper = jsonConverter.getObjectMapper();
		//序列换成json时,将所有的long变成string
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
        simpleModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
        objectMapper.registerModule(simpleModule);
        list.add(jsonConverter);
    }
}
```
### 三、总结
在实际的项目开发中，很多服务都是纯微服务开发，没有用到`SpringMVC`，在这种情况下，使用`JsonUtil`工具类实现对象序列化，可能是一个非常好的选择。

如果有理解不对的地方，欢迎网友批评指出！
### 四、参考
1、[CSDN - Java下利用Jackson进行JSON解析和序列化](https://blog.csdn.net/accountwcx/article/details/24585987)
