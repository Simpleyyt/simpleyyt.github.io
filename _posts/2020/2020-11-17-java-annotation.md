---
layout: post
categories: Java
title: 天天都在使用的 Java 注解，你真的了解它吗？
tagline: by 子悠
tags: 
  - 子悠

---

Hello，大家好，我是阿粉，Java 的注解相信大家天天都在用，但是关于注解的原理，大家都了解吗？这篇文章通过意见简单的示例给大家演示一下注解的使用和原理。

<!--more-->

# Java 元注解

注解（Annotation）是一种可以放在 Java 类上，方法上，属性上，参数前面的一种特殊的注释，用来注释注解的注解叫做元注解。元注解我们平常不会编写，只需要添加到我们自己编写的注解上即可，。

Java 自带的常用的元注解有`@Target`，`@Retention`，`@Documented`，`@Inherited` 分别有如下含义

1. `@Target`：标记这个注解使用的地方，取值范围在枚举 `java.lang.annotation.ElementType`：`TYPE,FIELD,METHOD,PARAMETER,CONSTRUCTOR,LOCAL_VARIABLE,ANNOTATION_TYPE,PACKAGE,TYPE_PARAMETER,TYPE_USE`。
2. `@Retention` ：标识这个注解的生命周期，取值范围在枚举 `java.lang.annotation.RetentionPolicy `，`SOURCE,CLASS,RUNTIME`，一般定义的注解都是在运行时使用，所有要用 `@Retention(RetentionPolicy.RUNTIME)`;
3. `@Documented`：表示注解是否包含到文档中。
4. `@Inherited` ：使用`@Inherited`定义子类是否可继承父类定义的`Annotation`。`@Inherited`仅针对`@Target(ElementType.TYPE)`类型的`annotation`有效，并且仅针对`class`的继承，对`interface`的继承无效。

# 定义注解

上面介绍了几个元注解，下面我们定义一个日志注解来演示一下，我们通过定义一个名为`OperationLog`  的注解来记录一些通用的操作日志，比如记录什么时候什么人查询的哪个表的数据或者新增了什么数据。编写注解我们用的是 `@interface` 关键字，相关代码如下：

```java
package com.api.annotation;

import java.lang.annotation.*;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author 子悠<br>
 * <b>Date：</b>2020-11-17 22:10<br>
 * <b>Desc：</b>用于记录操作日志<br>
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface OperationLog {

    /**
     * 操作类型
     *
     * @return
     */
    String type() default OperationType.SELECT;

    /**
     * 操作说明
     *
     * @return
     */
    String desc() default "";

    /**
     * 请求路径
     *
     * @return
     */
    String path() default "";

    /**
     * 是否记录日志，默认是
     *
     * @return
     */
    boolean write() default true;

    /**
     * 是否需要登录信息
     *
     * @return
     */
    boolean auth() default true;
  	/**
     * 当 type 为 save 时必须
     *
     * @return
     */
    String primaryKey() default "";

    /**
     * 对应 service 的 Class
     *
     * @return
     */
    Class<?> defaultServiceClass() default Object.class;
}

```

### 说明

上面的注解，我们增加了`@Target({ElementType.METHOD})` , `@Retention(RetentionPolicy.RUNTIME)`, `@Documented` 三个元注解，表示我们这个注解是使用在方法上的，并且生命周期是运行时，而且可以记录到文档中。然后我们可以看到定义注解采用的u是`@interface`  关键字，并且我们给这个注解定义了几个属性，同时设置了默认值。主要注意的是平时我们编写的注解一般必须设置`@Target`和`@Retention`，而且 `@Retention`一般设置为`RUNTIME`，这是因为我们自定义的注解通常要求在运行期读取，另外一般情况下，不必写`@Inherited`。

## 使用

上面的动作只是把注解定义出来了，但是光光定义出来是没有用的，必须有一个地方读取解析，才能提现出注解的价值，我们就采用 Spring 的 AOP 拦截这个注解，将所有携带这个注解的方法所进行的操作都记录下来。

```java
package com.api.config;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.*;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author 子悠<br>
 * <b>Date：</b>2020-11-17 14:40<br>
 * <b>Desc：</b>aspect for operation log<br>
 */
@Aspect
@Component
@Order(-5)
@Slf4j
public class LogAspect {
    /**
     * Pointcut for methods which need to record operate log
     */
    @Pointcut("within(com.xx.yy.controller..*) && @annotation(com.api.annotation.OperationLog)")
    public void logAspect() {
    }

    /**
     * record log for Admin and DSP
     *
     * @param joinPoint parameter
     * @return result
     * @throws Throwable
     */
    @Around("logAspect()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        Object proceed = null;
        String classType = joinPoint.getTarget().getClass().getName();
        Class<?> targetCls = Class.forName(classType);
        MethodSignature ms = (MethodSignature) joinPoint.getSignature();
        Method targetMethod = targetCls.getDeclaredMethod(ms.getName(), ms.getParameterTypes());
        OperationLog operation = targetMethod.getAnnotation(OperationLog.class);
        if (null != operation && operation.write()) {
            SysMenuOpLogEntity opLogEntity = new SysMenuOpLogEntity();
            StringBuilder change = new StringBuilder();
            if (StrUtil.isNotBlank(operation.type())) {
                switch (operation.type()) {
                    case OperationType.ADD:
                        proceed = joinPoint.proceed();
                        String addString = genAddData(targetCls, operation.defaultServiceClass(), joinPoint.getArgs());
                        opLogEntity.setAfterJson(addString);
                        change.append(OperationType.ADD);
                        break;
                    case OperationType.DELETE:
                        String deleteString = autoQueryDeletedData(targetCls, operation.primaryKey(), operation.defaultServiceClass(), joinPoint.getArgs());
                        opLogEntity.setBeforeJson(deleteString);
                        change.append(OperationType.DELETE);
                        proceed = joinPoint.proceed();
                        break;
                    case OperationType.EDIT:
                        change.append(OperationType.EDIT);
                        setOpLogEntity(opLogEntity, targetCls, operation.primaryKey(), operation.defaultServiceClass(), joinPoint.getArgs());
                        proceed = joinPoint.proceed();
                        break;
                    case OperationType.SELECT:
                        opLogEntity.setBeforeJson(getQueryString(targetCls, operation.defaultServiceClass(), joinPoint.getArgs()));
                        change.append(operation.type());
                        proceed = joinPoint.proceed();
                        break;
                    case OperationType.SAVE:
                        savedDataOpLog(opLogEntity, targetCls, operation.primaryKey(), operation.defaultServiceClass(), joinPoint.getArgs());
                        change.append(operation.type());
                        proceed = joinPoint.proceed();
                        break;
                    case OperationType.EXPORT:
                    case OperationType.DOWNLOAD:
                        change.append(operation.type());
                        proceed = joinPoint.proceed();
                        break;
                    default:
                }
                opLogEntity.setExecType(operation.type());
            }
            StringBuilder changing = new StringBuilder();
            if (StrUtil.isNotBlank(opLogEntity.getExecType())) {
                if (operation.auth()) {
                    LoginUserVO loginUser = getLoginUser();
                    if (null != loginUser) {
                        opLogEntity.setUserId(loginUser.getUserId());
                        opLogEntity.setUserName(loginUser.getUserName());
                        changing.append(loginUser.getUserName()).append("-");
                    } else {
                        log.error("用户未登录");
                    }
                }
                opLogEntity.setCreateTime(DateUtils.getCurDate());
                opLogEntity.setRemark(getOperateMenuName(targetMethod, operation.desc()));
                opLogEntity.setPath(getPath(targetMethod, targetMethod.getName()));
                opLogEntity.setChanging(changing.append(change).toString());
                menuOpLogService.save(opLogEntity);
            }
        }
        return proceed;
    }

    /**
     * query data by userId
     *
     * @param targetCls           class
     * @param defaultServiceClass default service class
     * @return
     * @throws Exception
     */
    private String queryByCurrentUserId(Class<?> targetCls, Class<?> defaultServiceClass) throws Exception {
        BaseService baseService = getBaseService(targetCls, defaultServiceClass);
        LoginUserVO loginUser = dspBaseService.getLoginUser();
        if (null != loginUser) {
            Object o = baseService.queryId(loginUser.getUserId());
            return JsonUtils.obj2Json(o);
        }
        return null;
    }

    /**
     * return query parameter
     *
     * @param targetCls           class
     * @param args                parameter
     * @param defaultServiceClass default service class
     * @return
     * @throws Exception
     */
    private String getQueryString(Class<?> targetCls, Class<?> defaultServiceClass, Object[] args) {
        if (args.length > 0) {
            Class<?> entityClz = getEntityClz(targetCls, defaultServiceClass);
            for (Object arg : args) {
                if (arg.getClass().equals(entityClz) || arg instanceof BaseModel) {
                    return JsonUtils.obj2Json(arg);
                }
            }
        }
        return null;
    }

    /**
     * save record log while OperatorType is SAVE
     *
     * @param opLogEntity         entity
     * @param targetCls           class
     * @param primaryKey          primaryKey
     * @param defaultServiceClass default service class
     * @param args                parameter
     * @throws Exception
     */
    private void savedDataOpLog(SysMenuOpLogEntity opLogEntity, Class<?> targetCls, String primaryKey, Class<?> defaultServiceClass, Object[] args) throws Exception {
        Class<?> entityClz = getEntityClz(targetCls, defaultServiceClass);
        BaseService baseService = getBaseService(targetCls, defaultServiceClass);
        for (Object arg : args) {
            if (arg.getClass().equals(entityClz)) {
                if (StrUtil.isNotBlank(primaryKey)) {
                    Field declaredField = entityClz.getDeclaredField(primaryKey);
                    declaredField.setAccessible(true);
                    Object primaryKeyValue = declaredField.get(arg);
                    //if primary key is not null that means edit, otherwise is add
                    if (null != primaryKeyValue) {
                        //query data by primary key
                        Object o = baseService.queryId(primaryKeyValue);
                        opLogEntity.setBeforeJson(JsonUtils.obj2Json(o));
                    }
                }
                opLogEntity.setAfterJson(JsonUtils.obj2Json(arg));
            }
        }
    }

    /**
     * set parameter which edit data
     *
     * @param opLogEntity         entity
     * @param targetCls           class
     * @param primaryKey          primaryKey
     * @param defaultServiceClass default service class
     * @param args                parameter
     * @throws Exception
     */
    private void setOpLogEntity(SysMenuOpLogEntity opLogEntity, Class<?> targetCls, String primaryKey, Class<?> defaultServiceClass, Object[] args) throws Exception {
        Map<String, String> saveMap = autoQueryEditedData(targetCls, primaryKey, defaultServiceClass, args);
        if (null != saveMap) {
            if (saveMap.containsKey(ASPECT_LOG_OLD_DATA)) {
                opLogEntity.setBeforeJson(saveMap.get(ASPECT_LOG_OLD_DATA));
            }
            if (saveMap.containsKey(ASPECT_LOG_NEW_DATA)) {
                opLogEntity.setBeforeJson(saveMap.get(ASPECT_LOG_NEW_DATA));
            }
        }
    }

    /**
     * query data for edit and after edit operate
     *
     * @param targetCls           class
     * @param primaryKey          primaryKey
     * @param defaultServiceClass default service class
     * @param args                parameter
     * @return map which data
     * @throws Exception
     */
    private Map<String, String> autoQueryEditedData(Class<?> targetCls, String primaryKey, Class<?> defaultServiceClass, Object[] args) throws Exception {
        if (StrUtil.isBlank(primaryKey)) {
            throw new Exception();
        }
        Map<String, String> map = new HashMap<>(16);
        Class<?> entityClz = getEntityClz(targetCls, defaultServiceClass);
        BaseService baseService = getBaseService(targetCls, defaultServiceClass);
        for (Object arg : args) {
            if (arg.getClass().equals(entityClz)) {
                Field declaredField = entityClz.getDeclaredField(primaryKey);
                declaredField.setAccessible(true);
                Object primaryKeyValue = declaredField.get(arg);
                //query the data before edit
                if (null != primaryKeyValue) {
                    //query data by primary key
                    Object o = baseService.queryId(primaryKeyValue);
                    map.put(ASPECT_LOG_OLD_DATA, JsonUtils.obj2Json(o));
                    map.put(ASPECT_LOG_NEW_DATA, JsonUtils.obj2Json(arg));
                    return map;
                }
            }
        }
        return null;
    }

    /**
     * return JSON data which add operate
     *
     * @param targetCls           class
     * @param args                parameter
     * @param defaultServiceClass default service class
     * @return add data which will be added
     * @throws Exception
     */
    private String genAddData(Class<?> targetCls, Class<?> defaultServiceClass, Object[] args) throws Exception {
        List<Object> parameter = new ArrayList<>();
        for (Object arg : args) {
            if (arg instanceof HttpServletRequest) {
            } else {
                parameter.add(arg);
            }
        }
        return JsonUtils.obj2Json(parameter);
    }

    /**
     * query delete data before delete operate
     *
     * @param targetCls           class
     * @param primaryKey          primaryKey
     * @param defaultServiceClass default service class
     * @param ids                 ids
     * @return delete data which will be deleted
     * @throws Throwable
     */
    private String autoQueryDeletedData(Class<?> targetCls, String primaryKey, Class<?> defaultServiceClass, Object[] ids) throws Throwable {
        if (StrUtil.isBlank(primaryKey)) {
            throw new OriginException(TipEnum.LOG_ASPECT_PRIMARY_KEY_NOT_EXIST);
        }
        //get service
        BaseService baseService = getBaseService(targetCls, defaultServiceClass);
        //get entity
        Class<?> entityClz = getEntityClz(targetCls, defaultServiceClass);
        //query deleted data by primary key
        Query query = new Query();
        WhereOperator whereOperator = new WhereOperator(entityClz);
        Set<Object> set = new HashSet<>(Arrays.asList((Object[]) ids[0]));
        whereOperator.and(primaryKey).in(set.toArray());
        query.addWhereOperator(whereOperator);
        List list = baseService.queryList(query);
        return JsonUtils.obj2Json(list);
    }


    /**
     * return service by targetCls
     *
     * @param targetCls           current controller class
     * @param defaultServiceClass default service class
     * @return service instance
     * @throws Exception
     */
    private BaseService getBaseService(Class<?> targetCls, Class<?> defaultServiceClass) throws Exception {
        //根据类名拿到对应的 service 名称
        String serviceName = getServiceName(targetCls, defaultServiceClass);
        BaseService baseService;
        if (null != defaultServiceClass) {
            baseService = (BaseService) ApplicationContextProvider.getBean(serviceName, defaultServiceClass);
        } else {
            Class<?> type = targetCls.getDeclaredField(serviceName).getType();
            baseService = (BaseService) ApplicationContextProvider.getBean(serviceName, type);
        }
        return baseService;
    }

    /**
     * return service name
     *
     * @param targetCls           current controller class
     * @param defaultServiceClass default service class
     * @return service name
     */
    private String getServiceName(Class<?> targetCls, Class<?> defaultServiceClass) {
        if (null != defaultServiceClass && Object.class != defaultServiceClass) {
            return StrUtil.left(defaultServiceClass.getSimpleName(), 1).toLowerCase() + defaultServiceClass.getSimpleName().substring(1);
        }
        return StrUtil.left(targetCls.getSimpleName(), 1).toLowerCase() + targetCls.getSimpleName().substring(1).replace("Controller", "Service");
    }


    /**
     * return entity class
     *
     * @param targetCls           current controller class
     * @param defaultServiceClass default service class
     * @return entity class
     * @throws Exception
     */
    private Class<?> getEntityClz(Class<?> targetCls, Class<?> defaultServiceClass) {
        try {
            Class<?> type;
            if (null != defaultServiceClass && Object.class != defaultServiceClass) {
                type = defaultServiceClass;
            } else {
                type = targetCls.getDeclaredField(getServiceName(targetCls, null)).getType();
            }
            String entityName = type.getName().replace("service", "entity").replace("Service", "Entity");
            Class<?> entityClz = Class.forName(entityName);
            return entityClz;
        } catch (Exception e) {
            log.error("获取 class 失败");
        }
        return null;
    }


    /**
     * require path
     *
     * @param targetMethod target method
     * @param defaultPath  default require path
     * @return require path
     */
    private String getPath(Method targetMethod, String defaultPath) {
        String path = defaultPath;
        PostMapping postMapping = targetMethod.getAnnotation(PostMapping.class);
        GetMapping getMapping = targetMethod.getAnnotation(GetMapping.class);
        RequestMapping requestMapping = targetMethod.getAnnotation(RequestMapping.class);
        if (null != postMapping) {
            path = postMapping.value()[0];
        } else if (null != getMapping) {
            path = getMapping.value()[0];
        } else if (null != requestMapping) {
            path = requestMapping.value()[0];
        }
        return path;
    }

}


```

上面的代码中我们定义了一个切面指定需要拦截的包名和注解，因为涉及到很多业务相关的代码，所以不能完整的提供出来，但是整个思路就是这样的，在每种操作类型前后将需要记录的数据查询出来进行记录。代码很长主要是用来获取相应的参数值的，大家使用的时候可以根据自己的需要进行取舍。比如在新增操作的时候，我们将新增的数据进行记录下来；编辑的时候将编辑前的数据查询出来和编辑后的数据一起保存起来，删除也是一样的，在删除前将数据查询出来保存到日志表中。

同样导出和下载都会记录相应信息，整个操作类型的代码如下：

```
package com.api.annotation;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author 子悠<br>
 * <b>Date：</b>2020-11-17 22:11<br>
 * <b>Desc：</b>无<br>
 */
public interface OperationType {
    /**
     * 新增
     **/
    String ADD = "add";
    /**
     * 删除
     **/
    String DELETE = "delete";
    /**
     * 使用实体参数修改
     **/
    String EDIT = "edit";
    /**
     * 查询
     **/
    String SELECT = "select";

    /**
     * 新增和修改的保存方法，使用此类型时必须配置主键字段名称
     **/
    String SAVE = "save";

    /**
     * 导出
     **/
    String EXPORT = "export";

    /**
     * 下载
     **/
    String DOWNLOAD = "download";

}

```

后续在使用的时候只需要在需要的方法上加上注解，填上相应的参数即可`@OperationLog(desc = "查询单条记录", path = "/data")`

# 总结

注解一个我们天天再用的东西，虽然不难，但是我们却很少自己去写注解的代码，通过这篇文章能给大家展示一下注解的使用逻辑，希望对大家有帮助。Spring 中的各种注解本质上也是这种逻辑都需要定义使用和解析。很多时候我们可以通过自定义注解去解决很多场景，比如日志，缓存等。

# 写在最后

最后邀请你加入我们的知识星球，这里有 1800+ 优秀的人与你一起进步，如果你是小白那你是稳赚了，很多业内经验和干活分享给你；如果你是大佬，那可以进来我们一起交流分享你的经验，说不定日后我们还可以有合作，给你的人生多一个可能。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)

