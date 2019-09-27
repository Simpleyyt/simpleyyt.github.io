---
layout: post
title: 浅谈从纯Servlet到Spring的请求分发机制
tagline: by 炭烧生蚝
categories: Java
tags: 
  - Java
---

本文将分享从纯 Servlet 时代到 Spring 框架时代的过程中，关于请求分发的一些思考。

<!--more-->

在讲请求分发之前先梳理一下一个Web请求的交互逻辑：

1. 首先用户在客户端发送一个请求到服务器。
2. 这个请求首先会经过操作系统的 TCP/IP 协议栈解析后发送至某一个端口
3. 在该端口运行着一个 Web 应用服务器（假设是 Tomcat）
4. 接着 Tomcat 会把请求根据请求路径传送给对应的 Servlet 处理  
（要注意的是，Web 服务器本身是不处理请求的，比如说 Tomcat，它只负责分发请求）

# 1.Servlet时期的请求分发

在还没有 spring 框架的时候，只能单纯用 Servlet 处理请求。

具体做法是：把 Servlet 及其映射路径配置在一个叫 web.xml 的配置文件中，当服务器启动时，Tomcat 会自动读取这个文件，然后根据文件中的配置，把请求分配到对应的 Servlet。

这个时候请求分发的工作是在 Tomcat 中完成的，通常一个业务对应一个 Servlet。比如说关于用户的增删改查，对应的 Servlet 很有可能是这样子的：  
1. AddUserServlet  
2. DeleteUserServlet  
3. UpdateUserServlet  
...

对应的结构图如下：

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/requestdistribution/1.png)

这和我们如今开发中的架构很不一样，因为现在通常是一个类里面包含了对同个业务所有处理逻辑。比如说对用户的增删改查都放在同一个类里，用不同的方法区分。

但在以前，一个 Servlet 里面通常只有一个 service 是有用的，其他和生命周期相关的方法基本只是给一个空壳的重写，所以一个 service 方法就对应一个业务处理逻辑。这种分类方法不仅给 web 服务器带来很大负担的，而且会导致 web.xml 文件十分庞大。（User 的增删改查逻辑对应4个 Servlet，要是能把和 User 相关的处理逻辑分发给一个大的 UserServlet 就好了）

后来有人找到了把和 User 相关的处理逻辑分发给一个大的 UserServlet 的方法。具体方法就是使用 Java 中的反射。详细代码如下：

```java
/**
 * 该Servlet不需要进行配置，因为该Servlet从来不需要被直接访问，使用来被继承的
 * 可以定义为abstract class
 */
public abstract class BaseServlet extends HttpServlet{

	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		//解决post表单中文乱码问题
		request.setCharacterEncoding("utf-8");

		//获取method属性的值（方法名）
		String methodName = request.getParameter("method");
		if(methodName == null || methodName.trim().isEmpty()) {
			throw new RuntimeException("您没有传递method参数！无法确定您想要调用的方法！");
		}
		
		//使用反射调用方法	
		try {
			//获取当前Servlet的Class信息
			Class clazz = this.getClass();//实际访问的Servlet，不是BaseServlet，是BaseServlet的子类比如UserServlet
			//使用反射创建对象
			//Object obj = clazz.newInstance();
			//获取方法
			Method method = clazz.getMethod(methodName, HttpServletRequest.class,HttpServletResponse.class);
			//使用反射执行方法
			method.invoke(this, request,response);
		} catch (Exception e) {
			e.printStackTrace();
		} 
	}
}
```

这样一来，分发请求的结构就变成了这样：

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/requestdistribution/2.png)

也就是说同类业务的请求分配给同一个 Serlvet，当具体到细分的业务逻辑时，使用不同的方法进行区分；在运行的时候使用反射调用具体的方法。

这么做能明显减轻服务器分发请求的压力，也让 web.xml 文件变得更加简洁。而且同类型的业务逻辑能够编写在同一个类中，方便管理维护。但是有没有可能做得更好呢？

# 2.Spring框架的请求分发

先用一幅图来描述 Spring 框架的请求分发，图片通俗易懂：

![](http://www.justdojava.com/assets/images/2019/java/image-tssh/requestdistribution/3.png)

从图中可以看到 web 服务器此时已经不用考虑分发请求的问题了，它只需要把请求发送给 DispatcherServlet 即可。请求分发的任务将落到 DispatcherServlet 身上。

而 DispatchServlet 也不再是把请求分发给其他 Servlet，而是根据请求路径分发给对应的 Controller，然后再分发到 Controller 中对应的处理方法。

下面给出一个迷你简写版的 DispatchServlet 代码：

```java
public class DispatcherServlet implements Servlet {

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {}

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        for(MappingHandler mappingHandler : HandlerManager.mappingHandlerList){
            try{
                if (mappingHandler.handle(servletRequest, servletResponse)){
                    return;
                }
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {}
}
```

从上面的代码可以看到 DispatcherServlet 处理请求的逻辑也很简单，就是根据请求路径找到对应可以处理的处理器。

至于 MappingHandler 是怎么初始化的，这就涉及到 Spring 框架中控制反转和依赖注入的知识了。简单来说就是，在开发的过程中，我们不再把请求路径配置在 web.xml 里面，而是通过注解的方式配置在每一个个处理方法的上方。当 spring 框架启动后，会根据注解把每个处理方法初始化为一个个 MappingHandler（里面包括请求路径和处理逻辑），供 DispatcherServlet 调配使用。

这样的请求分发方式无疑比 Serlvet 时期的请求分发更加灵活，强大。它把请求分发的工作从 web 服务器上移植到框架中，可扩展性更强。而且配合 spring 框架的控制反转机制，把处理逻辑类的初始化及生命周期控制交给框架管理，更加安全高效。