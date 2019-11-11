---
layout: post
title:  Shiro框架详解
tagline: by 懿
categories: 权限控制框架
tags: 
    - 懿
---


之间工作中曾经用到过shiro这个权限控制的框架，之前一直都是停留在用的方面，没有过多的
去理解这方面的知识，现在有时间，专门研究了一下这个Shiro权限的框架使用。
<!--more-->

## Shiro是什么？

Apache Shiro是一个强大而灵活的开源安全框架，它干净利落地处理身份认证，授权，企业会话管理和加密。

Apache Shiro的首要目标是易于使用和理解。安全有时候是很复杂的，甚至是痛苦的，但它没有必要这样。框架应该尽可能掩盖复杂的地方，露出一个干净而直观的API，来简化开发人员在使他们的应用程序安全上的努力。

因为很多时候很多人比较喜欢使用Spring提供的Spring Security和Shiro来进行对比，我们也对比一下这个内容。

Apache Shiro是Java的一个安全框架。

目前，使用Apache Shiro的人越来越多，因为它相当简单，对比Spring Security，可能没有Spring Security做的功能强大，但是在实际工作时可能并不需要那么复杂的东西，所以使用小而简单的Shiro就足够了。

所以，我们工作中很多时候都是直接使用的Shiro来进行安全控制。

我们介绍完了之后，在看看Shiro有什么用处？

## Shiro作用

1. 验证用户来核实他们的身份
2. 对用户执行访问控制

比如说我们可以用来判断一个用户是否被分配了一个安全的角色。然后来判断这个人到底在项目中能有什么权限来处理，
意思就是假如说有2个人，一个是管理员，他有增删改查的功能，而另外一个用户就是只又查的功能，没有增删改的功能，
通过Shiro来进行控制，就能达到这种效果。

我们可以看一下Apache官网上对Shiro，它都包含哪些功能。

![](/assets/images/2019/java/image_yi/04_13/shiro图.jpg)

我们看看它每一个模块代表的是什么意思

Authentication：认证，有时也简称为“登录”，这是一个证明用户是他们所说的他们是谁的行为。

Authorization：授权，访问控制的过程，也就是绝对“谁”去访问“什么”
授权用于回答安全问题，例如“用户是否允许编辑帐户”，“该用户是否允许查看此网页”，“该用户是否可以访问”到这个按钮？“这些都是决定用户有权访问的决定，因此都代表授权检查。

Cryptography：密码术是通过隐藏信息或将其转换为无意义来保护信息免受不良访问的做法，因此没有其他人可以阅读它。Shiro专注于密码学的两个核心要素：使用公钥或私钥加密数据的密码，以及对密码等数据进行不可逆转加密的哈希（也称为消息摘要）。

Shiro Cryptography的主要目标是采用传统上非常复杂的领域，并在提供强大的密码学功能的同时使其他人轻松实现。

Session Management：Session会话，会话是您的用户在使用您的应用程序时携带一段时间的数据桶。传统上，会话专用于Web或EJB环境。不再！Shiro支持任何应用程序环境的会话。此外，Shiro还提供许多其他强大功能来帮助您管理会话。

Web Support：Shiro的web支持的API能够轻松地帮助保护 Web 应用程序。主要就是用来对Web程序进行一个好的支持的。

Caching：缓存，他是Apache Shiro中的第一层公民，来确保安全操作快速而又高效。

Concurrency：shiro利用它的并发特性来支持多线程应用程序

Testing：测试支持的存在来帮助你编写单元测试和集成测试，并确保你的能够如预期的一样安全。

"Run As"：其实这个就是有是有允许一个用户假设为另外一个用户身份的功能，有时候在管理脚本的时候很有效果

Remember Me：在会话中记住用户的身份，所以他们只需要在强制时候登录。

## Shiro核心

Shiro其实是有三大核心组件的，Subject、SecurityManager和Realms。

Subject：Subject实质上是一个当前执行用户的特定的安全“视图”。鉴于"User"一词通常意味着一个人，而一个Subject可以是一个人，
但它还可以代表第三方服务，daemon account，cron job，或其他类似的任何东西——基本上是当前正与软件进行交互的任何东西。
所有Subject实例都被绑定到（且这是必须的）一个SecurityManager上。当你与一个Subject交互时，那些交互作用转化为与SecurityManager交互的特定subject的交互作用。
我们可以把Subject认为是一个门面，SecurityManager才是真正的执行者。

SecurityManager：安全管理器，也就是说所有与安全有关的操作都会与SecurityManager进行交互，而且他管理这Subject，它其实是Shiro的核心
是Shiro架构的心脏。并作为一种“保护伞”对象来协调内部的安全组件共同构成一个对象图。

Realms：
域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。

当配置Shiro时，你必须指定至少一个Realm用来进行身份验证和/或授权。SecurityManager可能配置多个Realms，但至少有一个是必须的。

![](/assets/images/2019/java/image_yi/04_13/Shiro整体架构.jpg)

我们可以通过一个简单的登录来看shiro对权限的控制。

我们就通过图解来理解一下，然后写个简单的代码

![](/assets/images/2019/java/image_yi/04_13/用户权限图.jpg)

上图是对shiro角色权限的设计，

而接下来我们就可以看一下它具体的登录图解了

![](/assets/images/2019/java/image_yi/04_13/shiro登录图.jpg)

我解释一下这个图。

1、登陆操作  携带用户名密码给subject，subject调用自己的登陆方法传递用户名和密码给权限管理器，权限管理器将用户名密码传递给开发人员编写的realm的认证方法，realm根据用户名到数据库中查找是否存在该用户，若存在将认证信息存入到session中
 
 ```
 
 /*
 * 参数1：登陆标识
 * 参数2：正确的密码
 * 参数3：认证|授权器的名字
 */
 SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, user.getPassword(), this.getClass().getName());
 
 ```  

2、权限管理器会自动判断传递的密码与正确密码是否一致

3、访问3类资源（页面） 过滤器寻找权限管理器判断该用户是否拥有xxx权限，权限管理器从session中取出认证信息对象，返回给realm，realm判断该用户拥有什么权限，封装到授权信息中返回给权限管理器，权限管理器将判断的结果返回给过滤器

4、访问3类资源（xxx添加需要访问service）（对于过滤器来说属于2类资源），在执行方法时，会到达前置通知（esrvice方法上添加注解@RequiresPermissions("courier:list")），权限通知寻找权限管理器判断该用户是否拥有xxx权限，权限管理器从session中取出认证信息对象，返回给realm，realm判断该用户拥有什么权限，封装到授权信息中返回给权限管理器，权限管理器将判断的结果返回给权限通知


其实简单来说
/userAction_login ---------->请求先到达权限过滤器shiroFilter，先判断是几类资源 

登录属于一类资源直接放行到——------>userActon中（userAction中调用执行subject对象（使用入口是一个操作入口对象，里面有登陆方法，登出方法，获取当前对象方法）的登陆方法subject.login方法（携带着用户名，密码）

————>subject对象调用 securityManager的login方法  权限管理器不能判断用户和密码是对的需要

————>ream认证|授权器（开发人员编写，判断用户名是否存在，拥有什么权限）————>处理完后把认证信息对象返回给securityManager（）如果认证信息没有问题，权限管理器会把认证信息存入session（证明认证登陆过了）


可以自定义一个Realm；

```  

@Service("MyRealm")
public class MyRealm extends AuthorizingRealm{ //父类接口Realm
	
	@Autowired
	private UserDao ud;
	
	@Autowired
	private RoleService rs;
	
	@Autowired
	private PermissionService ps;

	@Override
	//授权
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		User user = (User) principals.getPrimaryPrincipal();
		SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
		List<Role> roles = rs.findByUser(user);
		if(roles != null && roles.size() > 0){
			for (Role role : roles) {
				info.addRole(role.getKeyword());
			}
		}
		List<Permission> permissions = ps.findByUser(user);
		if(permissions != null && permissions.size() > 0) {
			for (Permission permission : permissions) {
				info.addStringPermission(permission.getKeyword());
			}
		}
		return info;
	}

	@Override
	//认证
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken t) throws AuthenticationException {
		UsernamePasswordToken token = (UsernamePasswordToken) t;
		String username = token.getUsername();
		User user = ud.findByUsername(username);
		if(user != null){
			/*
			 * 参数1：登陆标识
			 * 参数2：正确的密码
			 * 参数3：认证|授权器的名字
			 */
			SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, user.getPassword(), this.getClass().getName());
			//ActionContext.getContext().getSession().put("loginUser", user);
			return info;
		} else {
			return null;
		}
		
	}

}


```  

登陆完以后访问页面资源（页面资源属于三类资源需要权限），

shiroFilter（已经配置了哪些资源是一类哪些资源是三类）

————>访问权限管理器，找权限管理器判断是否有xxx权限（权限管理器本身不能做出判断），权限管理器把之前登陆时保存在session中的认证信息取出

交给————>realm判断(realm中认证方法是登陆时候调用的)，realm查询数据库获得权限，把权限信息返还给————>权限管理器。

权限管理器根据realm的授权信息判断是否拥有xxx权限， 判断后把结果通知给————>权限管理器，权限管理器ShiraFilter 如果没有权限跳转到响应页面。

这其实就是一个简单的shiro框架的设计，可能个人设计的有点小毛病，只是一个测试，大家自行体会

## 总结 

Shiro是一个功能很齐全的框架，使用起来也很容易，总结一下
三大核心内容：
1. Subject 
2. SecurityManager 
3. Realms

Shiro 功能强大、且 简单、灵活。是Apache 下的项目比较可靠，且不跟任何的框架或者容器绑定，可以独立运行

所以这个权限控制框架，大家理解了么？有想法的咱们可以共同交流一下。
