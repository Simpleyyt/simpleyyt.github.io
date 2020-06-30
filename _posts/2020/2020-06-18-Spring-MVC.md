---
layout: post
title: 口述完SpringMVC的执行流程后，面试官说兄弟，你是培训的吧！
catgories: java
tags:
  - 懿
---

前几天阿粉的一个朋友去面试，面试官问他，你知道SpringMVC的执行流程么，我这个朋友在回答完之后，面试官相继问了几个问题，之后面试官说，兄弟你是培训出来的吧？朋友懵了，我培训都是一年前的事情了，这都能知道，于是，找阿粉来吐槽这个事情，结果，阿粉听完之后，分分钟觉得，确实不冤枉呀。
<!--more-->

##1.培训出来的朋友的回答之SpringMVC的执行流程

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/06-19/1.jpg)

大家看这个图，确实是没有任何问题的对不对，

1. 用户的 HTTP 的请求提交到 DispatcherServlet。

2. 由 DispatcherServlet 控制器查询一个或多个 HandlerMapping，找到处理请求的Controller。

3. DispatcherServlet 将请求提交到 Controller，Controller 调用业务逻辑处理后，返回 ModelAndView

4. 业务逻辑处理完了，这时候DispatcherServlet 查询 ModelAndView

5.  DispatcherServlet 查询一个或多个 ViewResoler 视图解析器，找到 ModelAndView 指定的视图。

6. 这时候就把这个 ModelAndView解析之后反馈给浏览器。

7. Http 响应：视图负责将结果显示到客户端

这时候有的面试官就会问你了，说如果说我不想经过视图解析器用什么注解，那不经过视图解析器的话，那么返回的数据就是 Json 了，这个大家肯定熟悉，直接回答 @ResponseBody 就可以了。

这部分内容很多的培训机构都会教给学员们去背诵，而不是如何的去理解一下，如果不往继续深挖的话，这块内容直接就过了，但是很多稍微大一点的“厂子”肯定会继续往下说，比如说：

- 那你说说SpringMVC的工作机制吧，这时候再朋友们的心中会有个大大的懵，机制？原理？机制和原理有啥不一样的呢？

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/06-19/2.jpg)

## 2.SpringMVC的工作机制

对于大家来说，SpringMVC的执行流程大家肯定都熟悉了，这个肯定大家回答的也会很完美，那么接下来就看看机制的问题吧，

SpringMVC框架其实围绕的都是 DispatcherServlet 来工作的，这个类也尤其的重要，其实看到名字的时候，阿粉第一想法就是，它是不是一个另类的 Servlet，而学习过 Java 的我们当然也知道 Servlet 可以拦截到 HTTP 发送过来的请求。

而我们的 Servlet 在初始化的时候，也就是在调用 init 方法的时候，SpringMVC 会根据配置，来获取配置信息，从而来获得 URI 和处理器 Handler 之间的映射关系，而这个URI 是统一资源标识符。为了更加灵活的操作和增强某些我们所需要的功能，这时候，SpringMVC还会给处理器加入拦截器。

而SpringMVC的容器初始化的时候，会建立所有url和controller的对应关系，

**ApplicationObjectSupport** 里面内容比较多，源码部分我精简了一下

``` 
@Override
	public final void setApplicationContext(@Nullable ApplicationContext context) throws BeansException {
		else if (this.applicationContext == null) {
			// Initialize with passed-in context.
			if (!requiredContextClass().isInstance(context)) {
				throw new ApplicationContextException(
						"Invalid application context: needs to be of type [" + requiredContextClass().getName() + "]");
			}
			this.applicationContext = context;
			this.messageSourceAccessor = new MessageSourceAccessor(context);
			initApplicationContext(context);
		}
	}
	此处注意的是initApplicationContext(context);
    这个方法在子类中实现了 ：子类AbstractDetectingUrlHandlerMapping实现了该方法
   
```
子类 **AbstractDetectingUrlHandlerMapping**
```
 protected void detectHandlers() throws BeansException {
   		ApplicationContext applicationContext = obtainApplicationContext();
   		String[] beanNames = (this.detectHandlersInAncestorContexts ?
   				BeanFactoryUtils.beanNamesForTypeIncludingAncestors(applicationContext, Object.class) :
   				applicationContext.getBeanNamesForType(Object.class));
   
   		// 采取任何bean的名字,我们可以确定url。.
   		for (String beanName : beanNames) {
   			String[] urls = determineUrlsForHandler(beanName);
   			if (!ObjectUtils.isEmpty(urls)) {
   				// URL路径发现:我们认为这是一个处理程序 这时候就要保存urls和beanName的对应关系,
   				registerHandler(urls, beanName);
   			}
   		}
   
   		if ((logger.isDebugEnabled() && !getHandlerMap().isEmpty()) || logger.isTraceEnabled()) {
   			logger.debug("Detected " + getHandlerMap().size() + " mappings in " + formatMappingName());
   		}
   	}
   
   通过父类的registerHandler给put到HandlerMap里面了
```

而我们在使用SpringMVC的Controller里面的注解解析 Url 的时候，通过的是什么类？什么方法呢？就是接下来的这个方法，大家可以看注释

```
//确定给定的url处理器bean。(Determine the URLs for the given handler bean.)
protected abstract String[] determineUrlsForHandler(String beanName);
```

在我们日常写 CRUD 的时候，建立Controller的时候，在上面总是习惯的@RequestMapping注解，里面写我们从前端的ajax或者其他方式请求过来的路径的时候，通过这个方法来进行Controller和url之间的对应关系。这时候关系完成了，接下来肯定是根据url去找Controller，继续往下执行了呗。

这时候就会之行你写的Controller方法，在我们的 Servlet里面是不是就相当于我们的 doService 的方法了，这一步阿粉就不仔细的给大家讲述了，大家可以参照　Servlet 来进行分析呢。

最后一步来了，通过反射调用处理请求的方法，这时候给大家返回一个视图，也就是我们的 return。但是这个return也是有讲究的，JSP, JSON, Velocity, FreeMarker, XML, PDF, Excel, 还有Html字符流等等。那它们该如何的进行处理的呢？接下来阿粉就来带大家看一下

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/06-19/3.jpg)

大家看一下这个图里面的 UrlBaseViewResolver ,类名真的是起的很有水准 **Url基础视图解析器** 基础视图解析器，那么我们先说返回 JSP 的，配置如下：

```
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

相信项目中如果使用JSP的同志们去直接的配置文件去寻找这个，肯定不出意外的能找到，这时候我们return的一个字符串，通过配置，直接找寻指定的JSP页面，这也是最经常使用的一点了。如果我们返回的是我们的 test 页面，那么肯定是 `return "test"` ,然后结合上面的配置 <property name="prefix" value="/WEB-INF/jsp/"/> 和 <property name="suffix" value=".jsp"/>，最后得到最终的URL： "/WEB-INF/jsp/" + "test" + ".jsp" == "/WEB-INF/jsp/test.jsp".

那么HTML这种是怎么处理返回的呢？其实也很简单，之前阿粉就说过这个 SpringMVC 其实可以理解成 Servlet ，那么返回的方式就有了PrintWriter的事情了

```
StringBuffer sb = new StringBuffer();
sb.append("<!doctype html><html><head><meta http-equiv=\"Content-Type\" content=\"text/html; charset=UTF-8\">")
sb.append("<div>xxxxxx</div>")
writer.write(sb.toString());
```

还有一个最常见的，返回JSON数据，那么Json数据我们最长用的，什么ajax这种来返回数据，使用各种UI的时候，也会让你返回JSON数据啦，这些东西都是必不可少呢，那么就像阿粉之前说的一个注解完事，如果有什么指定格式的，那么可以新建一个DTO的类，里面有你自己的属性，还可能带着你为了数据完整性而带上的数据比如List<T>这种。

而你说了这些之后，面试官顺带来了一句，Spring MVC的主要组件都有那些，你知道么？随便列举出几个来就行。

SpringMVC的组件：

1、前端控制器 DispatcherServlet

- 作用：接收请求、响应结果 相当于转发器，有了DispatcherServlet 就减少了其它组件之间的耦合度。

2、处理器映射器HandlerMapping

- 作用：根据请求的URL来查找Handler

3、处理器适配器HandlerAdapter

- 注意：在编写Handler的时候要按照HandlerAdapter要求的规则去编写，这样适配器HandlerAdapter才可以正确的去执行Handler。

4、处理器Handler

5、视图解析器 ViewResolver

- 作用：进行视图的解析 根据视图逻辑名解析成真正的视图（view）

6、视图View

- View是一个接口， 它的实现类支持不同的视图类型（jsp，freemarker，pdf, json等等）

关于SpringMVC的高频面试，你会了么？

## 文章参考

springmvc的工作机制
SpringMVC源码解读
