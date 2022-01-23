---
layout: post
categories: java
title: 实体类的属性映射怎么可以少了它？
tagline: by 淼淼之森
tags: 
    - 淼淼之森
---

我们都知道，随着一个工程的越来越成熟，模块划分会越来越细，其中实体类一般存于 domain 之中，但 domain  工程最好不要被其他工程依赖，所以其他工程想获取实体类数据时就需要在各自工程写 model，自定义 model 可以根据自身业务需要映射相应的实体属性。这样一来，这个映射工程貌似并不简单了。阿粉差点就犯难了……
<!--more-->

## 序　
所以阿粉今天就要给大家安利一款叫 `mapstruct` 的插件，它就是专门用来处理 domin 实体类与 model 类的属性映射的，我们只需定义 mapper 接口，mapstruct 在编译的时候就会自动的帮我们实现这个映射接口，避免了麻烦复杂的映射实现。

那可能有的小伙伴就要问了？为啥不用 `BeanUtils` 的  `copyProperties` 方法呢？不也照样可以实现属性的映射么？

这个啊，阿粉我开始也是好奇，所以就和 `BeanUtils` 深入交流了一番，最后才发现，`BeanUtils` 就是一个大老粗，只能同属性映射，或者在属性相同的情况下，允许被映射的对象属性少；但当遇到被映射的属性数据类型被修改或者被映射的字段名被修改，则会导致映射失败。而 `mapstruct` 就是一个巧媳妇儿了，她心思细腻，把我们可能会遇到的情况都给考虑到了（要是阿粉我也能找一个这样的媳妇儿该多好，内心笑出了猪声）

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/04-01/5.gif)

如下是这个插件的开源项目地址和各种例子：
- Github地址：https://github.com/mapstruct/mapstruct/
- 使用例子：https://github.com/mapstruct/mapstruct-examples



## 一、准备工作
接下来，阿粉将和大家一起去解开这个巧媳妇儿的真正面纱，所以我们还需要做一点准备工作。

### 1.1、了解`@Mapper` 注解
从 mybatis3.4.0 开始加入的 @Mapper 注解，目的就是为了不再写mapper映射文件。

我们只需要在 dao 层定义的接口上使用注解就可以实现sql语句的编写，例如：
```
@Select("select * from user where name = #{name}")
public User find(String name);

```
如上就是一个简单的使用，虽然简单，但也确实体现出了这个注解的优越性，至少少写了一个xml文件。

但阿粉我今天可不是想跟你探讨 `@Mapper` 注解，我主要是想去看我的巧媳妇儿 `mapstruct` ,所以我就只是想说下 `@Mapper` 注解的 `componentModel` 属性，`componentModel` 属性用于指定自动生成的接口实现类的组件类型，这个属性支持四个值：

- default: 这是默认的情况，mapstruct 不使用任何组件类型, 可以通过Mappers.getMapper(Class)方式获取自动生成的实例对象。
- cdi: the generated mapper is an application-scoped CDI bean and can be retrieved via @Inject
- spring: 生成的实现类上面会自动添加一个@Component注解，可以通过Spring的 @Autowired方式进行注入
- jsr330: 生成的实现类上会添加@javax.inject.Named 和@Singleton注解，可以通过 @Inject注解获取

### 1.2、依赖包
首先需要把依赖包导入，主要由两个包组成：
- `org.mapstruct:mapstruct`：包含了一些必要的注解，例如@Mapping。r若我们使用的JDK版本高于1.8，当我们在pom里面导入依赖时候，建议使用坐标是：`org.mapstruct:mapstruct-jdk8`，这可以帮助我们利用一些Java8的新特性。  
- `org.mapstruct:mapstruct-processor`：注解处理器，根据注解自动生成mapper的实现。

```
    <dependency>
        <groupId>org.mapstruct</groupId>
        <!-- jdk8以下就使用mapstruct -->
        <artifactId>mapstruct-jdk8</artifactId>
        <version>1.2.0.Final</version>
    </dependency>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct-processor</artifactId>
        <version>1.2.0.Final</version>
    </dependency>
```
好了，准备工作做完了，接下来我们就看看巧媳妇儿巧在什么地方吧。

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/04-01/6.jpg)

## 二、先简单玩一把
### 2.1、定义实体类以及被映射类 
```
// 实体类
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    private Integer id;
    private String name;
    private String createTime;
    private LocalDateTime updateTime;
}

// 被映射类VO1:和实体类一模一样
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserVO1 {
    private Integer id;
    private String name;
    private String createTime;
    private LocalDateTime updateTime;
}

// 被映射类VO1:比实体类少一个字段
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserVO2 {
    private Integer id;
    private String name;
    private String createTime;

}
```

### 2.2、定义接口：
当实体类和被映射对象属性相同或者被映射对象属性值少几个时：
```
@Mapper(componentModel = "spring")
public interface UserCovertBasic {
    UserCovertBasic INSTANCE = Mappers.getMapper(UserCovertBasic.class);

    /**
     * 字段数量类型数量相同，利用工具BeanUtils也可以实现类似效果
     * @param source
     * @return
     */
    UserVO1 toConvertVO1(User source);
    User fromConvertEntity1(UserVO1 userVO1);

    /**
     * 字段数量类型相同,数量少：仅能让多的转换成少的，故没有fromConvertEntity2
     * @param source
     * @return
     */
    UserVO2 toConvertVO2(User source);
}
```
从上面的代码可以看出：接口中声明了一个成员变量INSTANCE，母的是让客户端可以访问 Mapper 接口的实现。

### 2.3、使用
```
@RestController
public class TestController {

    @GetMapping("convert")
    public Object convertEntity() {
        User user = User.builder()
                .id(1)
                .name("张三")
                .createTime("2020-04-01 11:05:07")
                .updateTime(LocalDateTime.now())
                .build();
        List<Object> objectList = new ArrayList<>();

        objectList.add(user);

        // 使用mapstruct
        UserVO1 userVO1 = UserCovertBasic.INSTANCE.toConvertVO1(user);
        objectList.add("userVO1:" + UserCovertBasic.INSTANCE.toConvertVO1(user));
        objectList.add("userVO1转换回实体类user:" + UserCovertBasic.INSTANCE.fromConvertEntity1(userVO1));
        // 输出转换结果
        objectList.add("userVO2:" + " | " + UserCovertBasic.INSTANCE.toConvertVO2(user));
        // 使用BeanUtils
        UserVO2 userVO22 = new UserVO2();
        BeanUtils.copyProperties(user, userVO22);
        objectList.add("userVO22:" + " | " + userVO22);

        return objectList;
    }
}
``` 
### 2.4、查看编译结果
通过IDE的反编译功能查看编译后自动生成 `UserCovertBasic` 的实现类 `UserCovertBasicImpl` ，内容如下：
```
@Component
public class UserCovertBasicImpl implements UserCovertBasic {
    public UserCovertBasicImpl() {
    }

    public UserVO1 toConvertVO1(User source) {
        if (source == null) {
            return null;
        } else {
            UserVO1 userVO1 = new UserVO1();
            userVO1.setId(source.getId());
            userVO1.setName(source.getName());
            userVO1.setCreateTime(source.getCreateTime());
            userVO1.setUpdateTime(source.getUpdateTime());
            return userVO1;
        }
    }

    public User fromConvertEntity1(UserVO1 userVO1) {
        if (userVO1 == null) {
            return null;
        } else {
            User user = new User();
            user.setId(userVO1.getId());
            user.setName(userVO1.getName());
            user.setCreateTime(userVO1.getCreateTime());
            user.setUpdateTime(userVO1.getUpdateTime());
            return user;
        }
    }

    public UserVO2 toConvertVO2(User source) {
        if (source == null) {
            return null;
        } else {
            UserVO2 userVO2 = new UserVO2();
            userVO2.setId(source.getId());
            userVO2.setName(source.getName());
            userVO2.setCreateTime(source.getCreateTime());
            return userVO2;
        }
    }
}
```

### 2.5、浏览器查看结果
![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/04-01/1.png)

好了，一个流程就走完了，是不是感觉贼简单呢？

而且呀，阿粉温馨提醒：
如果是要转换一个集合的话，只需要把这里的实体类换成集合就行了，例如：
```
    List<UserVO1> toConvertVOList(List<User> source);
```

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/04-01/7.jpg)

## 三、不简单的情况
上面已经把整个流程都给过了一遍了，相信大家对 `mapstruct` 也有了一个基础的了解了，所以接下来的情况我们就不展示全部代码了，毕竟篇幅也有限，所以就直接上关键代码（因为不关键的和上面内容一样，哈哈）
### 3.1、类型不一致
实体类我们还是沿用 `User`；被映射对象 `UserVO3` 改为：
```
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserVO3 {
    private String id;
    private String name;
    // 实体类该属性是String
    private LocalDateTime createTime;
    // 实体类该属性是LocalDateTime
    private String updateTime;
}
```

那么我们定义的接口就要稍稍修改一下了：
```
    @Mappings({
            @Mapping(target = "createTime", expression = "java(com.java.mmzsblog.util.DateTransform.strToDate(source.getCreateTime()))"),
    })
    UserVO3 toConvertVO3(User source);

    User fromConvertEntity3(UserVO3 userVO3);
```
上面 `expression` 指定的表达式内容如下：
```
public class DateTransform {
    public static LocalDateTime strToDate(String str){
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyy-MM-dd HH:mm:ss");
        return LocalDateTime.parse("2018-01-12 17:07:05",df);
    }

}
```

通过IDE的反编译功能查看编译后的实现类，结果是这样子的：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/04-01/2.png)

从图中我们可以看到，编译时使用了expression中定义的表达式对目标字段 `createTime` 进行了转换；然后你还会发现 `updateTime` 字段也被自动从 LocalDateTime 类型转换成了 String 类型。

**阿粉小结**：

当字段类型不一致时，以下的类型之间是 `mapstruct` 自动进行类型转换的:
- 1、基本类型及其他们对应的包装类型。
此时 `mapstruct` 会自动进行拆装箱。不需要人为的处理
- 2、基本类型的包装类型和string类型之间

除此之外的类型转换我们可以通过定义表达式来进行指定转换。

### 3.2、字段名不一致
实体类我们还是沿用 `User`；被映射对象 `UserVO4` 改为：
```
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserVO4 {
    // 实体类该属性名是id
    private String userId;
    // 实体类该属性名是name
    private String userName;
    private String createTime;
    private String updateTime;
}
```

那么我们定义的接口就要稍稍修改一下了：
```
    @Mappings({
            @Mapping(source = "id", target = "userId"),
            @Mapping(source = "name", target = "userName")
    })
    UserVO4 toConvertVO(User source);
    
    User fromConvertEntity(UserVO4 userVO4);
```

通过IDE的反编译功能查看编译后的实现类，编译后的结果是这样子的：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/04-01/3.png)

**阿粉小结**：

当字段名不一致时，通过使用 `@Mappings` 注解指定对应关系，编译后即可实现对应字段的赋值。

很明显， `mapstruct` 通过读取我们配置的字段名对应关系，帮我们把它们赋值在了相对应的位置上，可以说是相当优秀了，但这也仅仅是优秀，而更秀的还请继续往下看：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/04-01/8.jpg)

### 3.3、属性是枚举类型
实体类我们还是改用 `UserEnum`:
```
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserEnum {
    private Integer id;
    private String name;
    private UserTypeEnum userTypeEnum;
}
```

被映射对象 `UserVO5` 改为：
```
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserVO5 {
    private Integer id;
    private String name;
    private String type;
}
```
枚举对象是：
```
@Getter
@AllArgsConstructor
public enum UserTypeEnum {
    Java("000", "Java开发工程师"),
    DB("001", "数据库管理员"),
    LINUX("002", "Linux运维员");
    
    private String value;
    private String title;

}
```
那么我们定义的接口还是照常定义，不会受到它是枚举就有所变化：
```
    @Mapping(source = "userTypeEnum", target = "type")
    UserVO5 toConvertVO5(UserEnum source);

    UserEnum fromConvertEntity5(UserVO5 userVO5);
```

通过IDE的反编译功能查看编译后的实现类，编译后的结果是这样子的：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/04-01/4.png)
很明显， `mapstruct` 通过枚举类型的内容，帮我们把枚举类型转换成字符串，并给type赋值，可谓是小心使得万年船啊。看来这巧媳妇儿不仅仅优秀还心细啊……

文章中的所有例子已上传github：[https://github.com/mmzsblog/mapstructDemo](https://github.com/mmzsblog/mapstructDemo)