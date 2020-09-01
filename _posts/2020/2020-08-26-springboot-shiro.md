---
layout: post
categories: SpringBoot
title: SpringBoot2.x 整合 shiro 权限框架
tagline: by 淼淼之森
tags: 
    - 淼淼之森
---
在实际项目中，经常需要用到角色权限区分，以此来为不同的角色赋予不同的权利，分配不同的任务。比如，普通用户只能浏览；会员可以浏览和评论；超级会员可以浏览、评论和看视频课等；实际应用场景很多。毫不夸张的说，几乎每个完整的项目都会设计到权限管理。

因此，这篇文章，阿粉就带大家将 shiro 权限框架整合到 SpringBoot 中，以达到快速的实现权限管理的功能。

<!--more-->

## 序
在 Spring Boot 中做权限管理，一般来说，主流的方案是 Spring Security ，但是由于 Spring Security 过于庞大和复杂，只要能满足业务需要，大多数公司还是会选择 Apache Shiro 来使用。

一般来说，Spring Security 和 Shiro 的区别如下：

Spring Security | Apache Shiro
---|---
重量级的安全管理框架| 轻量级的安全管理框架
概念复杂，配置繁琐 | 概念简单、配置简单
功能强大 | 功能简单

这篇文章会首先带大家了解 Apache Shiro ，然后再给出使用案例 Demo。

## 走进 Apache Shiro
### 官网认知
照例又去官网扒了扒介绍：

> **Apache Shiro™** is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management. With Shiro’s easy-to-understand API, you can quickly and easily secure any application – from the smallest mobile applications to the largest web and enterprise applications.
> **Apache Shiro™**是一个强大且易用的Java安全框架,能够用于身份验证、授权、加密和会话管理。Shiro拥有易于理解的API,您可以快速、轻松地获得任何应用程序——从最小的移动应用程序到最大的网络和企业应用程序。

简而言之，Apache Shiro 是一个强大灵活的开源安全框架，可以完全处理身份验证、授权、加密和会话管理。

### Shiro能到底能做些什么呢？

- 验证用户身份
- 用户访问权限控制，比如：1、判断用户是否分配了一定的安全角色。2、判断用户是否被授予完成某个操作的权限
- 在非 Web 或 EJB 容器的环境下可以任意使用Session API
- 可以响应认证、访问控制，或者 Session 生命周期中发生的事件
- 可将一个或以上用户安全数据源数据组合成一个复合的用户 “view”(视图)
- 支持单点登录(SSO)功能
- 支持提供“Remember Me”服务，获取用户关联信息而无需登录
  ···


### 为什么今天还要使用Apache Shiro？
对此，官方给出了详细的解释：[http://shiro.apache.org/](http://shiro.apache.org/)

自2003年以来，框架环境发生了很大变化，因此今天仍然有充分的理由使用Shiro。实际上有很多原因。Apache Shiro是：

- 易于使用 -易于使用是该项目的最终目标。应用程序安全性可能非常令人困惑和沮丧，并被视为“必要的邪恶”。如果您使它易于使用，以使新手程序员可以开始使用它，那么就不必再痛苦了。
- 全面 -Apache Shiro声称没有其他具有范围广度的安全框架，因此它可能是满足安全需求的“一站式服务”。
- 灵活 -Apache Shiro可以在任何应用程序环境中工作。尽管它可以在Web，EJB和IoC环境中运行，但并不需要它们。Shiro也不要求任何规范，甚至没有很多依赖性。
- 具有Web功能 -Apache Shiro具有出色的Web应用程序支持，使您可以基于应用程序URL和Web协议（例如REST）创建灵活的安全策略，同时还提供一组JSP库来控制页面输出。
- 可插拔 -Shiro干净的API和设计模式使它易于与许多其他框架和应用程序集成。您会看到Shiro与Spring，Grails，Wicket，Tapestry，Mule，Apache Camel，Vaadin等框架无缝集成。
- 受支持 -Apache Shiro是Apache Software Foundation（Apache软件基金会）的一部分，该组织被证明以其社区的最大利益行事。项目开发和用户群体友好的公民随时可以提供帮助。如果需要，像Katasoft这样的商业公司也可以提供专业的支持和服务。

### Shiro 核心概念
Apache Shiro  是一个全面的、蕴含丰富功能的安全框架。

下图为描述 Shiro 功能的框架图：

![图片来源于网络](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/08-01/01.png)

如图所示，功能包括：

- Authentication（认证）：用户身份识别，通常被称为用户“登录”
- Authorization（授权）：访问控制。比如某个用户是否具有某个操作的使用权限。
- Session Management（会话管理）：特定于用户的会话管理,甚至在非web 或 EJB 应用程序。
- Cryptography（加密）：在对数据源使用加密算法加密的同时，保证易于使用。

并且 Shiro 还有通过增加其他的功能来支持和加强这些不同应用环境下安全领域的关注点。

特别是对以下的功能支持：

- Web支持：Shiro 提供的 Web 支持 api ，可以很轻松的保护 Web 应用程序的安全。
- 缓存：缓存是 Apache Shiro 保证安全操作快速、高效的重要手段。
- 并发：Apache Shiro 支持多线程应用程序的并发特性。
- 测试：支持单元测试和集成测试，确保代码和预想的一样安全。
- "Run As"：这个功能允许用户假设另一个用户的身份(在许可的前提下)。
- "Remember Me"：跨 session 记录用户的身份，只有在强制需要时才需要登录。

**注意**： Shiro 不会去维护用户、维护权限，这些需要我们自己去设计/提供，然后通过相应的接口注入给 Shiro


## 使用案例 Demo
### 1.新建 maven 项目
为方便我们初始化项目，Spring Boot给我们提供一个项目模板生成网站。
- 1、打开浏览器，访问：[https://start.spring.io/](https://start.spring.io/)
- 2、根据页面提示，选择构建工具，开发语言，项目信息等。

### 2.导入 springboot 父依赖
```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.2.RELEASE</version>
</parent>
```	

### 3.相关 jar 包
web 包
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
shiro-spring 包就是此篇文章的核心
```xml
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-spring</artifactId>
	<version>1.4.0</version>
</dependency>
shiro 注解会用到 aop
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
数据库相关包使用的是mybatisplus
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid</artifactId>
	<version>1.1.12</version>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.1.0</version>
</dependency>
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-generator</artifactId>
	<version>3.1.0</version>
</dependency>
	<dependency>
	<groupId>org.apache.velocity</groupId>
	<artifactId>velocity-engine-core</artifactId>
	<version>2.0</version>
</dependency>
```		
### 4.数据库
建表语句在项目中有，项目地址: [https://github.com/justdojava/java-samples](https://github.com/justdojava/java-samples)

### 5.自定义 realm
```java
public class MyShiroRealm extends AuthorizingRealm {
	@Autowired
	private UserService userService;
	@Autowired
	private RoleService roleService;
	@Autowired
	private PermissionService permissionService;

	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {

		SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
		// HttpServletRequest request = (HttpServletRequest) ((WebSubject) SecurityUtils
		// .getSubject()).getServletRequest();//这个可以用来获取在登录的时候提交的其他额外的参数信息
		String username = (String) principals.getPrimaryPrincipal();
		// 受理权限
		// 角色
		Set<String> roles = new HashSet<String>();
		Role role = roleService.getRoleByUserName(username);
		System.out.println(role.getRoleName());
		roles.add(role.getRoleName());
		authorizationInfo.setRoles(roles);
		// 权限
		Set<String> permissions = new HashSet<String>();
		List<Permission> querypermissions = permissionService.getPermissionsByRoleId(role.getId());
		for (Permission permission : querypermissions) {
			permissions.add(permission.getPermissionName());
		}
		authorizationInfo.setStringPermissions(permissions);
		return authorizationInfo;
	}

	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken)
			throws AuthenticationException {
		String loginName = (String) authcToken.getPrincipal();
		// 获取用户密码
		User user = userService.getOne(new QueryWrapper<User>().eq("username", loginName));
		if (user == null) {
			// 没找到帐号
			throw new UnknownAccountException();
		}
		String password = new String((char[]) authcToken.getCredentials());
		String inpass = (new Md5Hash(password, user.getUsername())).toString();
		if (!user.getPassword().equals(inpass)) {
			throw new IncorrectCredentialsException();
		}
		// 交给AuthenticatingRealm使用CredentialsMatcher进行密码匹配
		SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(loginName, user.getPassword(),
				ByteSource.Util.bytes(loginName), getName());

		return authenticationInfo;
	}

}
```
### 6.shiro 配置类
```java
@Configuration
public class ShiroConfiguration {
	private static final Logger logger = LoggerFactory.getLogger(ShiroConfiguration.class);

	/**
	 * Shiro的Web过滤器Factory 命名:shiroFilter
	 */
	@Bean(name = "shiroFilter")
	public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
		ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
		// Shiro的核心安全接口,这个属性是必须的
		shiroFilterFactoryBean.setSecurityManager(securityManager);
		//需要权限的请求，如果没有登录则会跳转到这里设置的url
		shiroFilterFactoryBean.setLoginUrl("/login.html");
		//设置登录成功跳转url，一般在登录成功后自己代码设置跳转url，此处基本没用
		shiroFilterFactoryBean.setSuccessUrl("/main.html");
		//设置无权限跳转界面,此处一般不生效,一般自定义异常
		shiroFilterFactoryBean.setUnauthorizedUrl("/error.html");
		Map<String, Filter> filterMap = new LinkedHashMap<>();
		// filterMap.put("authc", new AjaxPermissionsAuthorizationFilter());
		shiroFilterFactoryBean.setFilters(filterMap);
		/*
		 * 定义shiro过滤链 Map结构
		 * Map中key(xml中是指value值)的第一个'/'代表的路径是相对于HttpServletRequest.getContextPath()的值来的
		 * anon：它对应的过滤器里面是空的,什么都没做,这里.do和.jsp后面的*表示参数,比方说login.jsp?main这种
		 * authc：该过滤器下的页面必须验证后才能访问,它是Shiro内置的一个拦截器org.apache.shiro.web.filter.authc.
		 * FormAuthenticationFilter
		 */
		Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
		/*
		 * 过滤链定义，从上向下顺序执行，一般将 /**放在最为下边; authc:所有url都必须认证通过才可以访问;
		 * anon:所有url都都可以匿名访问
		 */
		filterChainDefinitionMap.put("/login.html", "authc");
		filterChainDefinitionMap.put("/login", "anon");
		filterChainDefinitionMap.put("/js/**", "anon");
		filterChainDefinitionMap.put("/css/**", "anon");
		filterChainDefinitionMap.put("/logout", "logout");
		filterChainDefinitionMap.put("/**", "authc");
		shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
		return shiroFilterFactoryBean;
	}

	/**
	 * 权限管理
	 */
	@Bean
	public SecurityManager securityManager() {
		logger.info("=======================shiro=======================");
		DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
		securityManager.setRealm(MyShiroRealm());
		// securityManager.setRememberMeManager(rememberMeManager);
		return securityManager;
	}

	/**
	 * Shiro Realm 继承自AuthorizingRealm的自定义Realm,即指定Shiro验证用户登录的类为自定义的
	 */
	@Bean
	public MyShiroRealm MyShiroRealm() {
		MyShiroRealm userRealm = new MyShiroRealm();
		userRealm.setCredentialsMatcher(hashedCredentialsMatcher());
		return userRealm;
	}

	/**
	 * 凭证匹配器 密码验证
	 */
	@Bean(name = "credentialsMatcher")
	public HashedCredentialsMatcher hashedCredentialsMatcher() {
		HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
		// 散列算法:这里使用MD5算法;
		hashedCredentialsMatcher.setHashAlgorithmName("md5");
		// 散列的次数，比如散列两次，相当于 md5(md5(""));
		hashedCredentialsMatcher.setHashIterations(1);
		// storedCredentialsHexEncoded默认是true，此时用的是密码加密用的是Hex编码；false时用Base64编码
		hashedCredentialsMatcher.setStoredCredentialsHexEncoded(true);
		return hashedCredentialsMatcher;
	}

	/**
	 * 开启Shiro的注解(如@RequiresRoles,@RequiresPermissions),需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
	 */
	@Bean
	public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor() {
		AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
		authorizationAttributeSourceAdvisor.setSecurityManager(securityManager());
		return authorizationAttributeSourceAdvisor;
	}

}
```
### 7.测试类
```java
@RestController
public class UserController {
	@PostMapping("login")
	public String name(String username, String password) {
		String result = "已登录";
		Subject currentUser = SecurityUtils.getSubject();
		UsernamePasswordToken token = new UsernamePasswordToken(username, password);
		if (!currentUser.isAuthenticated()) {
			try {
				currentUser.login(token);// 会触发com.shiro.config.MyShiroRealm的doGetAuthenticationInfo方法
				result = "登录成功";
			} catch (UnknownAccountException e) {
				result = "用户名错误";
			} catch (IncorrectCredentialsException e) {
				result = "密码错误";
			}
		}
		return result;
	}

	@GetMapping("logout")
	public void logout() {
		Subject currentUser = SecurityUtils.getSubject();
		currentUser.logout();
	}

	@RequiresPermissions("role:update")
	@GetMapping("/role")
	public String name() {
		return "hello";
	}

	@RequiresPermissions("user:select")
	@GetMapping("/role2")
	public String permission() {
		return "hello sel";
	}

}
```
#### 7.1 登录测试
数据库账号（密码经过md5加盐加密）

![数据库账号](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/08-01/02.png)

![账号错误测试](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/08-01/03.png)

![密码错误测试](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/08-01/04.png)

![账号正确测试](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/08-01/05.png)

![登录成功界面](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/08-01/06.png)

#### 7.2 权限测试
![权限测试1](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/08-01/07.png)

![权限测试2](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/08-01/08.png)


### 8.说明
#### 8.1 无权限时的处理
无权限时自定义了一个异常。所以，权限测试的时候没有权限就会提示配置的提示语 “没有权限”。
```java
@ControllerAdvice
public class ShiroException {
	@ExceptionHandler(value = UnauthorizedException.class)
	@ResponseBody
	public String name() {
		return "没有权限";
	}
}
```
#### 8.2 角色权限测试与权限测试相同
权限设置可在shiro配置类中shiro过滤链设置，也可用注解方式设置，本文使用注解方式。

#### 8.3 shiro 的 session 和 cache
shiro 的 session 和 cache 管理可以自定义，本文用的是默认的，推荐自定义，方便管理。

## 小结
- Apache Shiro是Java的一个安全框架
- Shiro是一个强大的简单易用的Java安全框架，主要用来更便捷的认证、授权、加密、会话管理、与Web集成、缓存等
- Shiro使用起来小而简单
- spring中有spring security ,是一个权限框架，它和spring依赖过于紧密，没有shiro使用简单。
- shiro不依赖于spring,shiro不仅可以实现web应用的权限管理，还可以实现c/s系统，分布式系统权限管理，
- shiro属于轻量框架，越来越多企业项目开始使用shiro.

参考：[https://www.cnblogs.com/joker-dj/archive/2020/04/13/12690648.html](https://www.cnblogs.com/joker-dj/archive/2020/04/13/12690648.html)