---
layout: post
categories: Java
title: 还不会用lombok？你手不痛吗
tagline: by 小九
tags: 
  - lombok
---

hello~各位读者新年好，我是鸭血粉丝（大家可以称呼我为「阿粉」），一位立志要成为码王的男人！虽然阿粉现在天天都在努力写bug，咳咳。。不是，是努力的在搬砖。因为代码量比较大，所有有bug很正常，流汗中。。。恩，阿粉可不是在解释哦。

好了，阿粉今天要带着大家来看看阿粉在搬砖途中遇到的有趣的事情



## 事件回顾

阿粉不是一个啰嗦的人，所以直接来看下代码

```java
@Data
public class DataSourceConditionVo extends BaseConditionVo{
    private Long id;
    private String sourceName;
    private Integer status;
    private Long driverId;
    public DataSourceConditionVo(Long id,Integer status){
        this.id = id;
        this.status = status;
    }
}
```

大家看出有什么问题吗?阿粉悄悄的告诉大家,阿粉少写了一个无参的构造函数,导致 controller 将客户端传过来的json转成实体对象的时候报错了。

大家可不要笑话阿粉犯这么低级的错误，其实之前阿粉是有写无参构造函数的，只是看到 `@Data` 这个注解,阿粉以为会自动的生成无参构造函数,所以就把无参构造函数去掉了。汗。。

为了防止继续犯错，所以阿粉把lombok（lombok是什么？你还不知道？那你还不搬个板凳，买包瓜子，仔细往下面看）常用的注解都看了一遍，现在分享给大家，希望大家不要犯和阿粉一样的错误。

## lombok

首先，阿粉先和大家普及一下lombok（了解过的可以快进）的相关知识。

#### 1 什么是lombok ####

官方的介绍我就不复制了（是因为懒吗），主要是以简单的注解形式来简化一些没有技术含量并且又不得不写的代码，比如 get\set ，构造方法等等，提高了代码的简洁性。

#### 2 lombok的使用

lombok的使用非常简单，只需要引入 jar 包,然后 idea 安装一个 lombok 插件。

```xml
 <dependency>
     <groupId>org.projectlombok</groupId>
     <artifactId>lombok</artifactId>
     <version></version>
</dependency>
```

version选择最新的就可以了。idea 安装插件就不介绍了，相信在座的各位都是大佬。

#### 2.1 @Data

@Data注解在类上，会为类的所有属性自动生成setter/getter、equals、canEqual、hashCodetoString方法。下面通过简单的代码看下效果:

```java
@Data
public class User {
    private String name;
    private Integer sex;
}
```

现在看下编译之后的 class 文件：

```java
public class User {
    private String name;
    private Integer sex;
    public User() {
    }
    public String getName() {
        return this.name;
    }
    public Integer getSex() {
        return this.sex;
    }
    public void setName(final String name) {
        this.name = name;
    }
    public void setSex(final Integer sex) {
        this.sex = sex;
    }
    public boolean equals(final Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof User)) {
            return false;
        } else {
            User other = (User)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
                Object this$name = this.getName();
                Object other$name = other.getName();
                if (this$name == null) {
                    if (other$name != null) {
                        return false;
                    }
                } else if (!this$name.equals(other$name)) {
                    return false;
                }
                Object this$sex = this.getSex();
                Object other$sex = other.getSex();
                if (this$sex == null) {
                    if (other$sex != null) {
                        return false;
                    }
                } else if (!this$sex.equals(other$sex)) {
                    return false;
                }
                return true;
            }
        }
    }
    protected boolean canEqual(final Object other) {
        return other instanceof User;
    }
    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $name = this.getName();
        int result = result * 59 + ($name == null ? 43 : $name.hashCode());
        Object $sex = this.getSex();
        result = result * 59 + ($sex == null ? 43 : $sex.hashCode());
        return result;
    }
    public String toString() {
        return "User(name=" + this.getName() + ", sex=" + this.getSex() + ")";
    }
}
```

#### 2.2 @Getter/@Setter

这个就不演示了，注解在类或字段上面，就是生成 get 和 set 方法，具体看 @Data 里面生产的 get/set 方法。

#### 2.3 @ToString

这个也不演示了，注解在类上面，生成 toString 方法。

#### 2.4 @EqualsAndHashCode

这个也不演示了，注解在类上面，生成 hashCode 和 equals 方法。

#### 2.5 @NoArgsConstructor

注解在类，生成无参的构造方法。这个看下生成的代码，阿粉上面的代码就是加了这个注解就可以了。

```java
public class User {
    private String name;
    private Integer sex;

    public User() {
    }
}
```

#### 2.6 @RequiredArgsConstructor

注解在类上，生成包含final和@NonNull注解的成员变量的构造器。

```java
public class User {
    private String name;
    private final Integer sex;

    public User(final Integer sex) {
        this.sex = sex;
    }
}
```

#### 2.7 @AllArgsConstructor

注解在类，生成包含类中所有字段的构造方法。

```java
public class User {
    private String name;
    private final Integer sex;

    public User(final String name, final Integer sex) {
        this.name = name;
        this.sex = sex;
    }
}
```

#### 2.8 @Slf4j

这个也是用的比较多的，注解在类上，生成log常量

```java
public class User {
    private static final Logger log = LoggerFactory.getLogger(User.class);
    private String name;
    private Integer sex;

    public User() {
    }
}
```

### 3 Lombok的优缺点

1. 优点:

   a. 能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、toString等方法，提高了一定的开发效率。

   b. 让代码变得简洁，不用过多的去关注相应的方法。

   c. 属性做修改时，也简化了维护为这些属性所生成的getter/setter方法等。

2. 缺点:

   a. 不支持多种参数构造器的重载。

   b. 虽然代码变得简洁，但也大大降低了代码的可读性和完整性。

#### 4 总结

Lombok的这些知识点虽然简单，但是用得好却能大大的提高开发效率。不管别人用不用，反正阿粉早就开始用起来了。真香

一个小问题，却让阿粉了解了这么多知识，果然阿粉有成为码王的潜质。阿粉一定会努力学习，完成自己 目标。

好了，今天阿粉的分享就到这里了，我们下期再见。喜欢阿粉的伙伴记的点个赞哦！爱你，么么哒！