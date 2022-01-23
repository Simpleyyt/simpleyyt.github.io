---
layout: post
categories: springboot
title: 关于参数合法性验证，阿粉有话要说
tags: 
  - 炸鸡可乐
---

如何优雅的进行参数验证，需要了解的都在这里！

<!--more-->

### 一、介绍
关于参数合法性验证的重要性就不多说了，即使前端对参数做了基本验证以外，后端依然还需要进行验证，以防不合规的数据直接进入后端，**严重的甚至会造成系统直接崩溃**！

本文结合自己在项目中的实际使用经验，**主要以实用为主，对数据合法性验证做一次总结**，不了解的朋友可以学习一下，同时可以立马实践到项目上去。

下面我们通过几个示例来演示如何判断参数是否合法，不多说直接开撸！
### 二、断言验证
对于参数的合法性验证，最初的做法比较简单，自定义一个异常类。
```java
public class CommonException extends RuntimeException {

    /**错误码*/
    private Integer code;

    /**错误信息*/
    private String msg;

    //...set/get

    public CommonException(String msg) {
        super(msg);
        this.msg = msg;
    }

    public CommonException(String msg, Throwable cause) {
        super(msg, cause);
        this.msg = msg;
    }

}
```
当判断某个参数不合法的时候，直接抛异常！
```java
@RestController
public class HelloController {

	@RequestMapping("/upload")
	public void upload(MultipartFile file) {
		if (file == null) {
			throw new CommonException("请选择上传文件！");
		}
		
		//.....
	}
}
```
然后写一个统一异常拦截器，对抛异常的程序进行处理。

这种做法比较直观，**如果当前参数既要判断是否为空，又要判断长度是否超过最大长度的时候，代码就显得有点多了**！

于是，程序界的大佬想到了一个更加优雅又能节省代码的方式，创建一个断言类工具类，专门用来判断参数的是否合法，如果不合法，就抛异常！
```java
/**
 * 断言工具类
 */
public abstract class LocalAssert {
	
	public static void isTrue(boolean expression, String message) throws CommonException {
		if (!expression) {
			throw new CommonException(message);
		}
	}
	public static void isStringEmpty(String param, String message) throws CommonException{
		if(StringUtils.isEmpty(param)) {
			throw new CommonException(message);
		}
	}

	public static void isObjectEmpty(Object object, String message) throws CommonException {
		if (object == null) {
			throw new CommonException(message);
		}
	}

	public static void isCollectionEmpty(Collection coll, String message) throws CommonException {
		if (coll == null || (coll.size() == 0)) {
			throw new CommonException(message);
		}
	}
}
```
当我们需要对参数进行验证的时候，直接通过这个类就可以完成基本操作，方式如下：
```java
@RestController
public class HelloController {

	@RequestMapping("/save")
	public void save(String name, String email) {
		LocalAssert.isStringEmpty(name, "用户名不能为空！");
		LocalAssert.isStringEmpty(email, "邮箱不能为空！");
		
		//.....
	}
}
```
相比上个步骤，当要判断的参数比较多时，代码明显简洁多了！

类似这样的工具类，`spring`也提供了一个名为`Assert`的断言工具类，在开发的时候，可以直接使用！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/01.jpg)

### 三、注解验证
使用注解对数据进行合法性验证，可以说是 java 界一项非常伟大的创新，使用这种方式不仅使的代码变得很简洁，而且阅读起来非常令人赏心悦目！

#### 3.1、依赖包引入
下面我们一起来看看具体的实践方式，以`Spring Boot`工程为例，如果需要使用注解校验，直接引入`spring-boot-starter-web`依赖包即可，会自动将注解验证相关的依赖包打入工程！
```xml
<!-- spring boot web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
下面在创建实体类的时候，还会用到`lombok`插件，因此还需要引入`lombok`依赖包！
```xml
<!-- lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.4</version>
    <scope>provided</scope>
</dependency>
```
如果是普通的`Java`工程，引入以下几个依赖包即可！
```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.9.Final</version>
</dependency>
<dependency>
     <groupId>javax.el</groupId>
     <artifactId>javax.el-api</artifactId>
     <version>3.0.0</version>
 </dependency>
 <dependency>
    <groupId>org.glassfish.web</groupId>
    <artifactId>javax.el</artifactId>
    <version>2.2.6</version>
 </dependency>
```
#### 3.2、注解校验请求对象
紧接着我们来创建一个实体`User`，用于模拟用户注册时的请求实体对象！
```
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
public class User {

    @NotBlank(message = "用户名不能为空！")
    private String userName;

    @Email(message = "邮箱格式不正确")
    @NotBlank(message = "邮箱不能为空！")
    private String email;

    @NotBlank(message = "密码不能为空！")
    @Size(min = 8, max = 16,message = "请输入长度在8～16位的密码")
    private String userPwd;

    @NotBlank(message = "确认密码不能为空！")
    private String confirmPwd;
}
```
在`web`层创建一个`register()`注册接口方法，同时在请求参数上添加`@Valid`，如下：
```java
@RestController
public class UserController {

    @RequestMapping("/register")
    public boolean register(@RequestBody @Valid User user){
        if(!user.getUserPwd().equals(user.getConfirmPwd())){
            throw new CommonException("确认密码与密码不相同，请确认！");
        }
		//业务处理...
        return true;
    }
}
```
最后自定义一个异常全局处理器，用于处理异常消息，如下：
```java
@Slf4j
@Configuration
public class GlobalWebMvcConfig implements WebMvcConfigurer {

    /**
     * 统一异常处理
     * @param resolvers
     */
    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new HandlerExceptionResolver() {
            @Override
            public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object o, Exception e) {
                log.error("【统一异常拦截】请求出现异常，内容如下：",e);
                ModelAndView mv = new ModelAndView(new MappingJackson2JsonView());
                String uri = request.getRequestURI();
                if(e instanceof CommonException){
                    //CommonExecption为自定义异常类抛出的异常
                    printWrite(((CommonException) e).getMsg(),((CommonException) e).getData(), uri, mv);
                } else if(e instanceof MethodArgumentNotValidException){
                    //MethodArgumentNotValidException为注解校验异常类
                    //获取注解校验异常信息
                    String error = ((MethodArgumentNotValidException) e).getBindingResult().getFieldError().getDefaultMessage();
                    printWrite(error,null, uri, mv);
                } else {
                    printWrite(e.getMessage(),null, uri, mv);
                }
                return mv;
            }
        });
    }


    /**
     * 异常封装相应结果
     * @param object
     */
    private void printWrite(String msg, Object object, String uri, ModelAndView mv){
        ResResult resResult = new ResResult(uri, object);
        if(msg != null){resResult.setMsg(msg);}
        if(log.isDebugEnabled()){
            log.debug("【response】异常输出结果:" + JSONObject.toJSONString(resResult, SerializerFeature.WriteMapNullValue));
        }
        Map resultMap = BeanToMapUtil.beanToMap(resResult);
        mv.addAllObjects(resultMap);
    }
}
```
下面我们启动项目，使用`postman`来测试一把，看看效果如何？

* 测试字段是否为空

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/02.jpg)

* 测试邮箱是否合法

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/03.jpg)

* 测试密码长度是否符合要求

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/04.jpg)

* 测试密码与确认密码是否相同

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/05.jpg)

#### 3.3、注解校验请求参数
上面我们介绍了请求对象的验证方式，那如果直接在方法上对请求参数进行验证是否同样有效呢？

为了眼见为实，下面我们就来模拟在方法上对请求参数进行验证，看看结果如何。

新建一个查询接口`query`，如下
```java
@RestController
public class UserController {

    @PostMapping("/query")
    public boolean query(@RequestParam("userId") @Valid @NotBlank(message = "用户ID不能为空") String userId ){
        return true;
    }

}
```
使用`postman`请求试一试，默认给`userId`参数为`null`，结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/06.jpg)

**很清晰的看到，`query()`方法中的参数注解验证无效**！

当我们在`UserController`类上加上`@Validated`注解！
```java
@RestController
@Validated
public class UserController {

    @PostMapping("/query")
    public boolean query(@RequestParam("userId") @Valid @NotBlank(message = "用户ID不能为空") String userId ){
        return true;
    }

}
```
使用`postman`请求再试一试，结果如下！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/07.jpg)

很清晰的看到，注解进行了验证，同时还抛出异常`ConstraintViolationException`！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/08.jpg)

`@Validated`参数作用于类上时，表示告诉`Spring`可以对方法中请求参数进行校验！

所有在实际开发的时候，我们可以使用`@Validated`和`@Valid`注解的组合来对方法中的**请求参数**和**请求对象**进行校验！

同时，**`@Validated`和`@Valid`注解不仅仅只是验证控制器级别，可以验证任何`Spring`组件**，例如`Service`层方法入参的验证！
```java
@Service
@Validated
public class UserService {

    public void saveUser(@Valid User user){
        //dao插入
    }
}
```
#### 3.4、自定义注解验证
默认的情况下，依赖包已经给我们提供了非常多的校验注解，如下！

* JSR提供的校验注解！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/09.jpg)

* Hibernate Validator提供的校验注解

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/10.jpg)

但是某些情况，例如性别这个参数可能需要我们自己去验证，同时我们也可以自定义一个注解来完成参数的校验，实现方式如下！

* 新创建一个`Sex`注解，其中`SexValidator`类指的是具体的参数验证类

```java
@Target({FIELD})
@Retention(RUNTIME)
@Constraint(validatedBy = SexValidator.class)
@Documented
public @interface Sex {

    String message() default "性别值不在可选范围内";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

* `SexValidator`类，实现自`ConstraintValidator`接口

```java
public class SexValidator implements ConstraintValidator<Sex, String> {

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        Set<String> sexSet = new HashSet<String>();
        sexSet.add("男");
        sexSet.add("女");
        return sexSet.contains(value);
    }
}
```
最后在`User`实体类上加入一个性别参数，使用自定义注解进行校验！
```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
public class User {

    @NotBlank(message = "用户名不能为空！")
    private String userName;

    @Email(message = "邮箱格式不正确")
    @NotBlank(message = "邮箱不能为空！")
    private String email;

    @NotBlank(message = "密码不能为空！")
    @Size(min = 8, max = 16,message = "请输入长度在8～16位的密码")
    private String userPwd;

    @NotBlank(message = "确认密码不能为空！")
    private String confirmPwd;

    /**
     * 自定义注解校验
     */
    @Sex(message = "性别输入有误！")
    private String sex;
}
```
使用`postman`来请求试一试，结果如下！

* 不传`sex`参数

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/11.jpg)

很清晰的看到，已经生效！

#### 3.5、手动进行注解校验
某些时候呢，假如有100个类需要用到校验注解，此时我们可能在每个类会加上注解`@Validated`或者`@Valid`，再增加100个这样的类，就会造成很多大量的重复工作。

**而此时，我们的诉求是想对有校验注解的实体类进行全局参数验证**！

解决办法就会用到`Validator`提供的手动注解校验证工具类，实现方法如下！

* 新建一个注解验证工具类

```java
/**
 * 注解校验工具类
 */
public class ValidatorUtils {

    /**
     * 获取对象中所有注解校验证异常信息
     * @param object
     * @return
     */
    public static String validated(Object object){
        List<String> errorMessageList = new ArrayList<>();
		//获取注解校验工厂
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        Validator validator = factory.getValidator();
        Set<ConstraintViolation<Object>> violations = validator.validate(object);

        for (ConstraintViolation<Object> constraintViolation : violations) {
            errorMessageList.add(constraintViolation.getMessage());
        }
        return errorMessageList.toString();
    }
}
```

* 使用`ValidatorUtils`工具类，对参数进行验证

```java
@Test
public void testUser(){
    User user = new User();
    System.out.println(ValidatorUtils.validated(user));
}
```
执行之后，结果如下！
```
[邮箱不能为空！, 用户名不能为空！, 密码不能为空！, 确认密码不能为空！, 性别输入有误！]
```
当然你还可以对`ValidatorUtils`类进行改造，当有异常信息的时候，直接抛异常！

同时，你还可以通过`@Autowired`直接注入的方式来获取`Validator`对象！

```java
@Autowired
Validator validator
```
#### 3.6、spring 注解校验原理
如果你对`springmvc`的方法参数解析器（`HandlerMethodArgumentResolver`）了解的话，就可能会想到参数校验这块肯定是在对应的方法参数解析器里执行的。

直接定位到`resolveArgument`这个方法，先通过`WebDataBinder`进行入参属性绑定，然后再进行校验！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/12.png)

`validateIfApplicable`方法逻辑，会遍历当前参数`methodParam`所有的注解，如果注解是`@Validated`或者注解的名字以`Valid`开头，则使用`WebDataBinder`对象执行校验逻辑。

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/13.png)

方法参数解析器只针对接口请求时入参进行验证，如果想对任何组件中方法进行注解校验，似乎还缺了点什么！

而当需要对一个类中的方法参数使用注解校验时，在类上加上`@Validated`就是为了告诉`Spring`去校验方法参数！

底层核心是通过切面代理类并配合`MethodValidationPostProcessor`这个后置处理器进行处理！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/springboot-validate/14.jpg)

### 四、总结
参数验证，在开发中使用非常频繁，如何优雅的进行验证，让代码变得更加可读，是业界大佬一直在追求的目标！

本文主要是对自己在项目中的实际使用到参数验证方式加一整理，希望能帮助到各位网友！
### 五、参考
1、SpringMVC源码

2、[JavaGuide - 如何在 Spring/Spring Boot 中做参数校验？](https://juejin.im/post/5dc8bc745188254e7a155ba0#heading-14)

3、[胡峻峥 - SpringMvc @Validated注解执行原理](https://www.cnblogs.com/hujunzheng/p/12570921.html)