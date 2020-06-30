---
layout: post
categories: Java
title:  这可能是阿粉见过最详细的一份 Spring 异步任务教程
tagline: by 小黑
published: false
tags: 
  - 小黑
---

阿粉最近碰到一个场景，用户注册之后需要发送邮件给其邮箱。原先设计中，这是一个同步过程，注册方法需要等待邮件发送成功才能返回。

由于邮件发送流程对于注册来说并不是一个关键节点，我们可以将邮件发送异步执行，减少注册方法执行时间。

我们可以自己创建线程池，然后执行异步任务，示例代码如下：

<!--more-->

```java
// 生产使用线程池的最佳实践，一定要自定义线程池，不要嫌麻烦，使用 Executors 创建线程池
private ThreadPoolExecutor threadPool =
        new ThreadPoolExecutor(5,
                10,
                60l,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(200),
                new ThreadFactoryBuilder().setNameFormat("register-%d").build());

/**
 * 使用线程池执行发送邮件的任务
 */
private void sendEmailByThreadPool() {
    threadPool.submit(() -> emailService.sendEmail());
}
```

> ps: 生产使用线程池的最佳实践，一定要自定义线程池，根据业务场景设置合理的线程池参数，另外给线程设置具有明确意义的前缀，这样排查问题就非常简单。
>
> 千万不要为了方便，使用 Executors 相关方法创建线程池。

上面代码中使用线程池完成了发送邮件的异步任务，可以看到这个示例还是有点麻烦，我们不仅要自定义线程池，还需要在创建相关任务执行类。

Spring 提供执行异步任务功能，我们使用一个注解就可以轻松完成上面的功能。

今天阿粉就来讲解一下如何使用 Spring 异步任务，以及 Spring 异步任务使用过程中一些注意点。

## 异步任务使用方式

Spring 异步任务需要在相关的方法上设置 `@Async` 注解，这里为了举例，我们创建一个 `EmailService` 类，专用完成邮件服务。

代码如下所示：

```java
@Slf4j
@Service
public class EmailService {

    /**
     * 异步发送任务
     *
     * @throws InterruptedException
     */
    @SneakyThrows
    @Async
    public void sendEmailAsync() {
        log.info("使用 Spring 异步任务发送邮件示例");
        // 模拟邮件发送耗时
        TimeUnit.SECONDS.sleep(2l);
    }
}
```

这里要注意了，Spring 异步任务**默认关闭**的，我们需要使用 `@EnableAsync`开启异步任务。

如果还在使用 Spring XML 配置，我们需要配置如下配置：

```xml
<task:annotation-driven/>
```

上述配置完成之后，我们只需要在调用方，比如上一层 `Controller` 注入这个 `EmailService` ,然后直接调用这个方法，该方法将会在异步线程中执行。

```java
@Slf4j
@RestController
public class RegisterController {

    @Autowired
    EmailService emailService;

    @RequestMapping("register")
    public String register() {
		log.info("注册流程开始");
	    emailService.sendEmailAsync();
        return "success";
    }
 }
```

输出日志如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200630/1.png)

从日志上可以看到，两个方法执行线程不一样，这就说明了` EmailService#sendEmailAsync` 被异步线程成功执行。

## 带有返回值的异步任务

上面的异步任务比较简单，但是有时我们有需要获取异步任务返回值。

如果使用线程池执行异步任务，我们可以使用 `threadPool#submit` 获取返回对象 `Future`,接着我们就可以调用其内 `get` 方法,获取返回结果。

在 Spring 异步任务中，我们也可以使用 `Future` 获取返回结果，示例代码如下：

```java
@Async
@SneakyThrows
public Future<String> sendEmailAsyncWithResult() {
    log.info("使用 Spring 异步任务发送邮件，并且获取任务返回结果示例");
    TimeUnit.SECONDS.sleep(2l);
    return AsyncResult.forValue("success");
}
```

> 由于公号不能直接点击外链，关注「Java极客技术」，回复「20200701」获取源码~

这里需要注意，这里返回对象我们需要使用 Spring 内部类 `AsyncResult`。

Controller 层调用代码如下所示：

```java
 private void sendEmailWithResult() {
        Future<String> future = emailService.sendEmailAsyncWithResult();
        try {
            String result = future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }
}

```

我们知道 `Future#get` 方法将会一直阻塞，直到异步任务执行成功。

有时候我们获取异步任务的返回值是为了做一下后续业务，但是主流程方法是无需返回异步任务的返回值。如果我们使用了 `Future#get`方法，主流程就会一直被阻塞。

对于这种场景，我们可以使用 ` org.springframework.util.concurrent.ListenableFuture`稍微改造一下上面的方法。

`ListenableFuture` 这个类允许我们注册回调函数，一旦异步任务执行成功,或者执行异常，将会立刻执行回调函数。通过这种方式就可以不用阻塞执行的主线程。

示例代码如下：

```java
@Async
@SneakyThrows
public ListenableFuture<String> sendEmailAsyncWithListenableFuture() {
    log.info("使用 Spring 异步任务发送邮件，并且获取任务返回结果示例");
    TimeUnit.SECONDS.sleep(2l);
    return AsyncResult.forValue("success");
}
```

`Controller` 层代码如下所示：

```java
ListenableFuture<String> listenableFuture = emailService.sendEmailAsyncWithListenableFuture();
// 异步回调处理
listenableFuture.addCallback(new SuccessCallback<String>() {
    @Override
    public void onSuccess(String result) {
        log.info("异步回调处理返回值");

    }
}, new FailureCallback() {
    @Override
    public void onFailure(Throwable ex) {
        log.error("异步回调处理异常",ex);
    }
});
```



看到这里，如果有同学有疑惑，我们返回对象是 `AsyncResult`,为什么方法返回类可以是 `Future`,又可以是 `ListenableFuture`?

看完这张类继承关系，大家应该就知道答案了。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200630/2.png )

## 异常处理方式

异步任务中异常处理方式，不是很难，我们只要在方法中将整个代码块 `try...catch` 即可。

```java
try {
	// 其他代码
} catch (Exception e) {
    e.printStackTrace();
}
```

一般来说，我们只需要捕获 `Exception` 异常，就可以应对大部分情况

但是极端情况下，比如方法内发生 `OOM`，将会抛出 ` OutOfMemoryError`。如果发生` Error` 错误，以上的捕获代码就会失效。

Spring 的异步任务，默认提供几种异常处理方式，可以统一处理异步任务中的发生的异常。

### 带有返回值的异常处理方式

如果我们使用带有返回值的异步任务，处理方式就比较简单了，我们只需要捕获 `Future#get` 抛出的异常就好了。

```java
Future<String> future = emailService.sendEmailAsyncWithResult();
try {
    String result = future.get();
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (ExecutionException e) {
    e.printStackTrace();
}
```

如果我们使用 `ListenableFuture` 注册回调函数处理，那我们在方法内增加一个 `FailureCallback`,在这个实现类处理相关异常即可。

```java
ListenableFuture<String> listenableFuture = emailService.sendEmailAsyncWithListenableFuture();
// 异步回调处理
listenableFuture.addCallback(new SuccessCallback<String>() {
    @Override
    public void onSuccess(String result) {
        log.info("异步回调处理返回值");

    }
    // 异常处理
}, new FailureCallback() {
    @Override
    public void onFailure(Throwable ex) {
        log.error("异步回调处理异常",ex);
    }
});
```

### 统一异常处理方式

没有返回值的异步任务处理方式就比较复杂了，我们需要继承 `AsyncConfigurerSupport`，实现 `getAsyncUncaughtExceptionHandler` 方法，示例代码如下：

```java
@Slf4j
@Configuration
public class AsyncErrorHandler extends AsyncConfigurerSupport {

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        AsyncUncaughtExceptionHandler handler = (throwable, method, objects) -> {
            log.error("全局异常捕获", throwable);
        };
        return handler;
    }

}
```

> ps:这个异常处理方式只能处理未带返回值的异步任务。

## 异步任务使用注意点

###  异步线程池设置

 Spring 异步任务默认使用 Spring 内部线程池  `SimpleAsyncTaskExecutor` 。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200630/3.jpg)

这个线程池比较坑爹，不会**复用线程**。也就是说来一个请求，将会新建一个线程。极端情况下，如果调用次数过多，将会创建大量线程。

Java 中的线程是会占用一定的内存空间 ，所以创建大量的线程将会导致 **OOM** 错误。

**所以如果需要使用异步任务，我们需要一定要使用自定义线程池替换默认线程池**。

 **XML 配置方式**

如果当前使用 Spring XML 配置方式，我们可以使用如下配置设置线程池：

```java
<task:annotation-driven/>
<task:executor id="executor" pool-size="10" queue-capacity="200"/>
```

**注解方式**

如果注解方式配置，配置方式如下：

```java
@Configuration
public class AsyncConfiguration {


    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setThreadNamePrefix("task-Executor-");
        executor.setMaxPoolSize(10);
        executor.setCorePoolSize(5);
        executor.setQueueCapacity(200);
        // 还有其他参数可以设置
        return executor;
    }
}
```

只要我们配置了这个线程池` Bean`，Spring 的异步任务都将会使用该线程池执行。

如果我们应用配置了多个线程池` Bean`，异步任务需要指定使用某个线程池执行，我们只需要在 `@Async`注解上设置相应 `Bean` 的名字即可。示例代码如下：

```java
@Async("taskExecutor")
public void sendEmailAsync() {
    log.info("使用 Spring 异步任务发送邮件示例");
    TimeUnit.SECONDS.sleep(2l);
}
```

**Spring Boot 方式**

如果是 SpringBoot 项目，从阿粉的测试情况来看，默认将会创建核心线程数为 8，最大线程数为 `Integer.MAX_VALUE`，队列数也为 `Integer.MAX_VALUE`线程池。

> ps:以下代码基于 Spring-Boot 2.1.6-RELEASE，暂不确定 Spring-Boot 1.x 版本是否也是这种策略，熟悉的同学的，也可以留言指出一下。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200630/4.jpg)

虽然上面的线程池不用担心创建过多线程的问题，不是还是有可能队列任务过多，导致 **OOM** 的问题。所以还是建议使用自定义线程池吗，或者在配置文件修改默认配置，例如：

```JAVA
spring.task.execution.pool.core-size=10
spring.task.execution.pool.max-size=20
spring.task.execution.pool.queue-capacity=200
```

> ps：如果我们使用注解方式自定义了一个线程池，那么 Spring 异步任务都将会使用这个线程池。通过 SpringBoot 配置文件创建的线程池将会失效。

### 异步方法失效

Spring 异步任务背后原理是使用 AOP ，而使用 Spring AOP 时我们需要注意，切勿在方法内部调用其他使用 AOP 的方法，可能有点拗口，我们来看下代码：

```java
@Async
@SneakyThrows
public ListenableFuture<String> sendEmailAsyncWithListenableFuture() {
    // 这样调用，sendEmailAsync 不会异步执行
    sendEmailAsync();
    log.info("使用 Spring 异步任务发送邮件，并且获取任务返回结果示例");
    TimeUnit.SECONDS.sleep(2l);
    return AsyncResult.forValue("success");
}

/**
     * 异步发送任务
     *
     * @throws InterruptedException
     */
@SneakyThrows
@Async("taskExecutor")
public void sendEmailAsync() {
    log.info("使用 Spring 异步任务发送邮件示例");
    TimeUnit.SECONDS.sleep(2l);
}
```

上面两个方法都处于同一个类中，这样调用将会导致 AOP 失效，无法起到 AOP 的效果。

其他类似的 ` @Transactional`，以及自定义的 AOP 注解都会有这个问题，大家使用过程，千万需要注意这一点。

## 总结

Spring 异步任务帮我们大大解决简化开发了流程，只要使用一个`@Async`就可与轻松解决异步任务。

不过，虽然使用方式比较简单，大家使用过程一定要注意设置合理的线程池。

> 由于公号不能直接点击外链，关注「Java极客技术」，回复「20200701」获取源码~

