---
layout: post
title: 深入解析java反射
tagline: by 炸鸡可乐
categories: java反射
tags: 
  - 炸鸡可乐
---

本博文主要记录Java 反射（reflect）的使用，在了解反射之前，你应该先了解 Java 中的 Class 类，如果你不是很了解，可以先简单了解下。

<!--more-->

### 一、什么是反射？
反射 (Reflection) 是 Java 的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。

Oracle 官方对反射的解释是：
> Reflection enables Java code to discover information about the fields, methods and constructors of loaded classes, and to use reflected fields, methods, and constructors to operate on their underlying counterparts, within security restrictions.
The API accommodates applications that need access to either the public members of a target object (based on its runtime class) or the members declared by a given class. It also allows programs to suppress default reflective access control.

简而言之，通过反射，我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。程序中一般的对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。
### 二、反射的主要用途
很多人都认为反射在实际的 Java 开发应用中并不广泛，其实不然。当我们在使用 IDE(如 Eclipse，IDEA)时，当我们输入一个对象或类并想调用它的属性或方法时，一按点号，编译器就会自动列出它的属性或方法，这里就会用到反射。

反射最重要的用途就是开发各种通用框架。很多框架（比如 `Spring`）都是配置化的（比如通过 `XML` 文件配置 `Bean`），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射，运行时动态加载需要加载的对象。

举一个例子，在运用 Struts 2 框架的开发中我们一般会在 struts.xml 里去配置 Action，比如：
```
<action name="login"
       class="org.xxx.SimpleLoginAction"
       method="execute">
   <result>/shop/shop-index.jsp</result>
   <result name="error">login.jsp</result>
</action>
```
配置文件与`Action`建立了一种映射关系，当 `View `层发出请求时，请求会被 `StrutsPrepareAndExecuteFilter` 拦截，然后 `StrutsPrepareAndExecuteFilter` 会去动态地创建 `Action` 实例。比如我们请求 `login.action`，那么 `StrutsPrepareAndExecuteFilter`就会去解析`struts.xml`文件，检索`action`中`name`为`login`的`Action`，并根据`class`属性创建`SimpleLoginAction`实例，并用`invoke`方法来调用`execute`方法，这个过程离不开反射。

对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。而对于一般的开发者来说，不深入框架开发则用反射用的就会少一点，不过了解一下框架的底层机制有助于丰富自己的编程思想，也是很有益的。
### 三、反射的基本运用
#### 3.1、通过反射获取class对象
> 通过反射获取对象有三种方式

##### 3.1.1、Class.forName()获取
```
public static Class<?> forName(String className)
            throws ClassNotFoundException {
    Class<?> caller = Reflection.getCallerClass();
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
}
```
比如，在 JDBC 开发中常用此方法加载数据库驱动
```
Class.forName("包名.类名");
```
##### 3.1.2、类名.class获取
例如：
```
Class<?> intClass = int.class;
Class<?> integerClass = Integer.TYPE;

#RelfectEntity类为本文的例子
Class relfectEntity2 = RelfectEntity.class;
```
##### 3.1.3、对象getClass()获取
```
StringBuilder str = new StringBuilder("hello world");
Class<?> strClass = str.getClass();
```
三种方法都可以实现获取class对象，框架开发中，一般第一种用的比较多。
#### 3.2、获取类的构造器信息
获取类构造器的用法，主要是通过Class类的getConstructor方法得到Constructor类的一个实例，而Constructor类有一个newInstance方法可以创建一个对象实例:
```
public T newInstance(Object ... initargs)
```
#### 3.3、获取类的实例
> 通过反射来生成对象主要有两种方式。

* 使用Class对象的newInstance()方法来创建Class对象对应类的实例。

```
Class<?> c = String.class;
Object str = c.newInstance();
```
* 先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建实例。

```
//获取String所对应的Class对象
Class<?> c = String.class;
//获取String类带一个String参数的构造器
Constructor constructor = c.getConstructor(String.class);
//根据构造器创建实例
Object obj = constructor.newInstance("23333");
System.out.println(obj);
```
这种方法可以用指定的构造器来创建实例。
#### 3.4、获取类的变量
实体类：
 ```
/**
 * 基类
 */
public class BaseClass {

	public String publicBaseVar1;

	public String publicBaseVar2;
}

/**
 * 子类
 */
public class ChildClass extends BaseClass{

	public String publicOneVar1;

	public String publicOneVar2;

	private String privateOneVar1;

	private String privateOneVar2;
}
```
测试：
```
public class VarTest {
	
	public static void main(String[] args) {
		//1.获取并输出类的名称
		Class mClass = ChildClass.class;
		System.out.println("类的名称：" + mClass.getName());
		System.out.println("----获取所有 public 访问权限的变量(包括本类声明的和从父类继承的)----");
		
		//2.获取所有 public 访问权限的变量(包括本类声明的和从父类继承的)
		Field[] fields = mClass.getFields();
		
		//遍历变量并输出变量信息
		for (Field field : fields) {
			//获取访问权限并输出
			int modifiers = field.getModifiers();
			System.out.print(Modifier.toString(modifiers) + " ");
			
			//输出变量的类型及变量名
	        System.out.println(field.getType().getName() + " " + field.getName());
		}
		System.out.println("----获取所有本类声明的变量----");
		
		//3.获取所有本类声明的变量
		Field[] allFields = mClass.getDeclaredFields();
		for (Field field : allFields) {
			//获取访问权限并输出
			int modifiers = field.getModifiers();
			System.out.print(Modifier.toString(modifiers) + " ");
			
			//输出变量的类型及变量名
	        System.out.println(field.getType().getName() + " " + field.getName());
		}
	}
}
```
输出结果：
```
类的名称：com.example.java.reflect.ChildClass
----获取所有 public 访问权限的变量(包括本类声明的和从父类继承的)----
public java.lang.String publicOneVar1
public java.lang.String publicOneVar2
public java.lang.String publicBaseVar1
public java.lang.String publicBaseVar2
----获取所有本类声明的变量----
public java.lang.String publicOneVar1
public java.lang.String publicOneVar2
private java.lang.String privateOneVar1
private java.lang.String privateOneVar2
```
#### 3.5、修改类的变量
修改子类
```
/**
 * 子类
 */
public class ChildClass extends BaseClass{

	public String publicOneVar1;

	public String publicOneVar2;

	private String privateOneVar1;

	private String privateOneVar2;

	public String printOneMsg() {
		return privateOneVar1;
	}
}
```
测试：
```
public class VarModfiyTest {
	
	public static void main(String[] args) throws Exception {
		//1.获取并输出类的名称
		Class mClass = ChildClass.class;
		System.out.println("类的名称：" + mClass.getName());
		System.out.println("----获取ChildClass类中的privateOneVar1私有变量----");
		
		//2.获取ChildClass类中的privateOneVar1私有变量
		Field privateField = mClass.getDeclaredField("privateOneVar1");
		
		//3. 操作私有变量
	    if (privateField != null) {
	        //获取私有变量的访问权
	        privateField.setAccessible(true);
	        
	        //实例化对象
	        ChildClass obj = (ChildClass) mClass.newInstance();
	        
	        //修改私有变量，并输出以测试
	        System.out.println("privateOneVar1变量，修改前值： " + obj.printOneMsg());

	        //调用 set(object , value) 修改变量的值
	        //privateField 是获取到的私有变量
	        //obj 要操作的对象
	        //"hello world" 为要修改成的值
	        privateField.set(obj, "hello world");
	        System.out.println("privateOneVar1变量，修改后值： " + obj.printOneMsg());
	    }
	}
}
```
输出结果：
```
类的名称：com.example.java.reflect.ChildClass
----获取ChildClass类中的privateOneVar1私有变量----
privateOneVar1变量，修改前值： null
privateOneVar1变量，修改后值： hello world
```
#### 3.6、获取类的所有方法
修改实体类
```
/**
 * 基类
 */
public class BaseClass {

	public String publicBaseVar1;

	public String publicBaseVar2;

	private void privatePrintBaseMsg(String var) {
		System.out.println("基类-私有方法，变量：" + var);
	}

	public void publicPrintBaseMsg(String var) {
		System.out.println("基类-公共方法，变量：" + var);
	}
}

/**
 * 子类
 */
public class ChildClass extends BaseClass{

	public String publicOneVar1;

	public String publicOneVar2;

	private String privateOneVar1;

	private String privateOneVar2;

	public String printOneMsg() {
		return privateOneVar1;
	}

	private void privatePrintOneMsg(String var) {
		System.out.println("子类-私有方法，变量：" + var);
	}

	public void publicPrintOneMsg(String var) {
		System.out.println("子类-公共方法，变量：" + var);
	}
}
```
测试：
```
public class MethodTest {

	public static void main(String[] args) {
		//1.获取并输出类的名称
		Class mClass = ChildClass.class;
		System.out.println("类的名称：" + mClass.getName());
		System.out.println("----获取所有 public 访问权限的方法,包括自己声明和从父类继承的---");

		//2 获取所有 public 访问权限的方法,包括自己声明和从父类继承的
		Method[] mMethods = mClass.getMethods();
		for (Method method : mMethods) {
			//获取并输出方法的访问权限（Modifiers：修饰符）
			int modifiers = method.getModifiers();
	        System.out.print(Modifier.toString(modifiers) + " ");

	        //获取并输出方法的返回值类型
	        Class returnType = method.getReturnType();
	        System.out.print(returnType.getName() + " " + method.getName() + "( ");

	        //获取并输出方法的所有参数
	        Parameter[] parameters = method.getParameters();
	        for (Parameter parameter : parameters) {
	            System.out.print(parameter.getType().getName() + " " + parameter.getName() + ",");
	        }

	        //获取并输出方法抛出的异常
	        Class[] exceptionTypes = method.getExceptionTypes();
	        if (exceptionTypes.length == 0){
	            System.out.println(" )");
	        } else {
	            for (Class c : exceptionTypes) {
	                System.out.println(" ) throws " + c.getName());
	            }
	        }
		}
		System.out.println("----获取所有本类的的方法---");
		//3. 获取所有本类的的方法
	    Method[] allMethods = mClass.getDeclaredMethods();
	    for (Method method : allMethods) {
			//获取并输出方法的访问权限（Modifiers：修饰符）
			int modifiers = method.getModifiers();
			System.out.print(Modifier.toString(modifiers) + " ");

	        //获取并输出方法的返回值类型
	        Class returnType = method.getReturnType();
	        System.out.print(returnType.getName() + " " + method.getName() + "( ");

	        //获取并输出方法的所有参数
	        Parameter[] parameters = method.getParameters();
	        for (Parameter parameter : parameters) {
	            System.out.print(parameter.getType().getName() + " " + parameter.getName() + ",");
	        }

	        //获取并输出方法抛出的异常
	        Class[] exceptionTypes = method.getExceptionTypes();
	        if (exceptionTypes.length == 0){
	            System.out.println(" )");
	        } else {
	            for (Class c : exceptionTypes) {
	                System.out.println(" ) throws " + c.getName());
	            }
	        }
		}
	}
}
```
输出：
```
类的名称：com.example.java.reflect.ChildClass
----获取所有 public 访问权限的方法,包括自己声明和从父类继承的---
public java.lang.String printOneMsg(  )
public void publicPrintOneMsg( java.lang.String arg0, )
public void publicPrintBaseMsg( java.lang.String arg0, )
public final void wait( long arg0,int arg1, ) throws java.lang.InterruptedException
public final native void wait( long arg0, ) throws java.lang.InterruptedException
public final void wait(  ) throws java.lang.InterruptedException
public boolean equals( java.lang.Object arg0, )
public java.lang.String toString(  )
public native int hashCode(  )
public final native java.lang.Class getClass(  )
public final native void notify(  )
public final native void notifyAll(  )
----获取所有本类的的方法---
public java.lang.String printOneMsg(  )
private void privatePrintOneMsg( java.lang.String arg0, )
public void publicPrintOneMsg( java.lang.String arg0, )
```

为啥会输出这么多呢？

因为所有的类默认继承object类，打开object类会发现有些公共的方法，所以一并打印出来了！
#### 3.7、调用方法
```
public class MethodInvokeTest {

	public static void main(String[] args) throws Exception {
		// 1.获取并输出类的名称
		Class mClass = ChildClass.class;
		System.out.println("类的名称：" + mClass.getName());
		System.out.println("----获取ChildClass类的私有方法privatePrintOneMsg---");

		// 2. 获取对应的私有方法
		// 第一个参数为要获取的私有方法的名称
		// 第二个为要获取方法的参数的类型，参数为 Class...，没有参数就是null
		// 方法参数也可这么写 ：new Class[]{String.class}
		Method privateMethod = mClass.getDeclaredMethod("privatePrintOneMsg", String.class);

		// 3. 开始操作方法
		if (privateMethod != null) {
			// 获取私有方法的访问权
			// 只是获取访问权，并不是修改实际权限
			privateMethod.setAccessible(true);

			// 实例化对象
			ChildClass obj = (ChildClass) mClass.newInstance();

			// 使用 invoke 反射调用私有方法
			// obj 要操作的对象
			// 后面参数传实参
			privateMethod.invoke(obj, "hello world");
		}
	}
}
```
输出结果：
```
类的名称：com.example.java.reflect.ChildClass
----获取ChildClass类的私有方法privatePrintOneMsg---
子类-私有方法，变量：hello world
```
### 四、总结
由于反射会额外消耗一定的系统资源，因此如果不需要动态地创建一个对象，那么就不需要用反射。另外，反射调用方法时可以忽略权限检查，因此可能会破坏封装性而导致安全问题。
### 五、参考文章
sczyh30：[深入解析Java反射](https://www.sczyh30.com/posts/Java/java-reflection-1/#%E5%9B%9B%E3%80%81%E5%8F%8D%E5%B0%84%E7%9A%84%E4%B8%80%E4%BA%9B%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)

伯特：[Java 反射由浅入深](https://juejin.im/post/598ea9116fb9a03c335a99a4#heading-5)

