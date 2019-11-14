---
layout: post
category: 面试
title: 聊聊面试中的过滤器与拦截器
tagline: by 子悠
tags: 
  - 子悠

---

### 背景

做过 JavaWeb 开发的对过滤器和拦截器肯定不会陌生，而且也会熟练的使用，但是关于过滤器和拦截器具体的区别和差异可能不是特别的了解，这篇文章就跟大家介绍下过滤器和拦截器的区别。
<!--more-->

#### 过滤器 Filter

首先介绍下什么是过滤器，过滤器英文叫 Filter，是 JavaEE 的标准，依赖于 Servlet 容器，使用的时候是配置在 web.xml 文件中的，可以配置多个，执行的顺序是根据配置顺序从上到下。常用来配置请求编码以及过滤一些非法参数，垃圾信息或者是网站登录验证码。

```

    <!-- filter -->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!-- filter end -->
    
```

#### 参考实现

```

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Date;

public class CaptchaFilter implements Filter {
	

	public void init(FilterConfig config) throws ServletException {
	}

	public void destroy() {
	}

	public void doFilter(ServletRequest servletRequest,
			ServletResponse servletResponse, FilterChain filterChain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) servletRequest;
		HttpServletResponse response = (HttpServletResponse) servletResponse;
		String servletPath = request.getServletPath();
		//获取验证码
		if(servletPath.matches("/captcha.jpg")) {
			response.setContentType("image/jpeg");
			//禁止图像缓存。
			response.setHeader("Pragma", "no-cache");
			response.setHeader("Cache-Control", "no-cache");
			response.setDateHeader("Expires", 0);
			//参数：宽、高、字符数、干扰量
			CaptchaProductor vCode = new CaptchaProductor(70,30,4,75);

			//根据token保存验证码内容
			CaptchaBean bean = new CaptchaBean();
			bean.setCaptcha(vCode.getCode());
			bean.setCreateTime(new Date());
			HttpSessionUtils.setSessionValue(request, "sessionCaptcha", bean);
			vCode.write(response.getOutputStream());
			return;
		}
	}
}

```
> 过滤器的实现可以通过实现 Filter 接口或者继承 Spring 的`org.springframework.web.filter.OncePerRequestFilter` 来实现。

#### 拦截器 Interceptor

拦截器 Interceptor 不依赖 Servlet 容器，依赖 Spring 等 Web 框架，在 SpringMVC 框架中是配置在SpringMVC 的配置文件中，在 SpringBoot 项目中也可以采用注解的形式实现。拦截器是 AOP 的一种应用，底层采用 Java 的反射机制来实现的。与过滤器一个很大的区别是在拦截器中可以注入 Spring 的 Bean，能够获取到各种需要的 Service 来处理业务逻辑，而过滤器则不行。

```

    <!-- 拦截器 -->
    <mvc:interceptors>
        <!-- 多个拦截器,顺序执行 -->
        <bean class="com.test.admin.interceptor.AuthInterceptor"/>
    </mvc:interceptors>
    
```

#### 参考实现

```

public class AuthInterceptor extends HandlerInterceptorAdapter {

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        //todo
        super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //todo 
        return super.preHandle(request, response, handler);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        
    }
}


```
> 拦截器的实现可以通过继承`org.springframework.web.servlet.handler.HandlerInterceptorAdapter;` 来实现。

### 执行顺序
因为我们的过滤器和拦截器都可以配置多个，那么关于各自的执行顺序是什么样子的呢？

过滤器的执行顺序首先跟在 web.xml 中配置的顺序有关，先配置的先执行，但是并不是说是等上一个过滤器执行结束了再执行下一个，它们之间是通过链来执行的，具体的过滤器和拦截器的执行过程我画了个图，可以看下。
![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/filter01.png)
<!--![filter01](http://www.justdojava.com/assets/images/2019/java/image_ziyou/protobuf1.jpg)-->


### 小结
今天简单的给大家介绍了过滤器和拦截器的区别和使用，希望对大家有帮忙。平时的工作中可能这些东西都是组长或者架构师搭建好的，自己只关注业务逻辑，但是很多时候我们还是要知其然知其所以然，多了解一些对自己是很有帮助的。

