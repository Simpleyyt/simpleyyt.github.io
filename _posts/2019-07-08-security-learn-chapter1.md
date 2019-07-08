---
layout: post
title:  跟我学spring security系列文章第一章 实现一个基本的登入
category: java
tags: [java,spring,spring security]
---

* content
{:toc}

源码地址:

https://github.com/pony-maggie/spring-security-learn

## 指定依赖

spring boot工程是需要引入spring-boot-starter-security即可

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
```

## 安全配置

我们需要自己实现一个类(类名无关)继承WebSecurityConfigurerAdapter ,然后重写里面的方法

```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {

		http.authorizeRequests().antMatchers("/anon1", "/anon2").permitAll().anyRequest().authenticated().and()
				.formLogin().loginPage("/login").defaultSuccessUrl("/user").permitAll().and().logout().permitAll();
	}

	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
		auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()).withUser("user")
				.password(new BCryptPasswordEncoder().encode("111111")).roles("USER");
	}
}
```

对于上面的配置说明如下：

@EnableWebSecurity是必须的，表示开启spring security功能。不加这个注解的话你的应用启动也没问题，但是在configure配置的规则不会生效。

configure这里的配置意思是/anon1,/anon2两个路径访问不受权限保护，可以任意访问。其它路径都要进行身份认证，认证的方式formlogin表单登录。然后接着又指定表单登录的时候用到的登录页面地址
是"/login"。defaultSuccessUrl指明了登录成功后跳转的页面("/user")，当然可以不用指定defaultSuccessUrl，这种情况下spring security会默认跳转到"/"页面。

下面会通过例子演示上面的规则是否生效。

configureGlobal不是必须的，但是它可以在内存中创建了一个用户，方便进行演示说明（真实的项目中账户信息是存入数据库的）。因为是入门教程，我们先不把问题复杂化，第一章我并不打算引入UserDetailsService的概念（后面章节会详细讲）。

该用户的名称为user，密码为password，用户角色为USER


在spring security 5.0之前你也可以这样写:

```
 spring security 5之后就不能用这种方式了，一定要指定密码的加密方法
	 @Autowired
	 public void configureGlobal(AuthenticationManagerBuilder auth) throws
	 Exception {
	 auth.inMemoryAuthentication().withUser("user").password("111111").roles("USER");
	 }
```

但是spring security 5.0之后一定要指明密码的加密方法(BCryptPasswordEncoder)，所以需要写成下面这种方式：

```
@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
		auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()).withUser("user")
				.password(new BCryptPasswordEncoder().encode("111111")).roles("USER");
	}
```

Spring security 5.0中新增了多种加密方式，也改变了密码的格式。以下是官方文档原话：


>-------------------------------------------------------------------------------------------------------------------
The general format for a password is:
{id}encodedPassword
Such that id is an identifier used to look up which PasswordEncoder should be used and encodedPassword is the original encoded password for the selected PasswordEncoder. The id must be at the beginning of the password, start with { and end with }. If the id cannot be found, the id will be null. For example, the following might be a list of passwords encoded using different id. All of the original passwords are "password".
{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG 
{noop}password 
{pbkdf2}5d923b44a6d129f3ddf3e3c8d29412723dcbde72445e8ef6bf3b508fbf17fa4ed4d6b99ca763d8dc 
{scrypt}$e0801$8bWJaSu2IKSn9Z9kM+TPXfOc/9bdYSrN1oD9qfVThWEwdRTnO7re7Ei+fUZRJ68k9lTyuTeUp4of4g24hHnazw==$OAOec05+bXxvuu/1qZ6NUR+xQYvYv7BeL1QxwRpY5Pc=  
{sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0
-------------------------------------------------------------------------------------------------------------------



另外需要注意的是，必须至少指定一个角色，否则会报错

```
java.lang.IllegalArgumentException: Cannot pass a null GrantedAuthority collection
```

## 添加controller测试代码

首先我们再controller里定义几个接口，方便我们一会再浏览器进行测试，添加几个网页，只显示基本的提示信息，能让你明白基本的跳转流程即可。

```
@Controller
public class TestController {
	
	@RequestMapping({"/anon1","/anon2"})
	public String anon() {
		return "anon";
	}
	
	@RequestMapping("/login")
	public String login() {
		return "login";
	}
	
	@RequestMapping("/")
	public String index() {
		return "index";
	}
	
	@RequestMapping("/user")
	public String user() {
		return "user";
	}

}
```

## 测试

启动工程，进行如下测试：

1. 访问首页，http://localhost:9090，需要授权，会自动跳入登录页：http://localhost:9090/login
2. 访问，http://localhost:9090/anon1和http://localhost:9090/anon2，可以不用登录直接访问
3. 访问用户页：http://localhost:9090/user，也需要授权，会自动跳入登录页：http://localhost:9090/login
4. 从登录页输入用户名和密码(user/111111)，进入用户主页







