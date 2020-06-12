---
layout: post
title: 【Spring 二】面试官：兄弟你来阐述一下Spring框架中Bean的生命周期？
catgories: java
tags:
  - 懿
---

前几天在阿粉CSDN上看一个文章，因为一个Spring的问题，期望薪资三万却被生生的压榨成了两万五，高于两万五人家都不要，让我感觉到了Spring的强大，不学习Spring是会吃亏的，那么我们就从各种高频面试来一点点深入吧。
<!--more-->

今天阿粉给大家带来的是关于Spring的另外的一道高频面试题，而且是非常非常高频的面试题，那就是Spring中的Bean的生命周期。

### 1.Bean的生命周期

关于Bean的生命周期，如果我们不谈这个Spring的话，实际上很多人都会想到New，通过 New 对象的形式来实现对 Bean的实例化操作，而在我们不再使用 Bean 了之后，这时候我们的 Java 就会对这个指定的 Bean 来进行垃圾回收了。

但是对于Spring来说，Bean的生命周期可能就比较让人头疼了，毕竟 Spring 这么复杂，而且里面的对 Bean 管理的非常的有逻辑了，每一层都有每一层的步骤。

如果现在我们去百度上面去搜索所有的关于Spring的Bean的生命周期，很多人会把这个解释出来

- 在IoC容器启动之后，并不会马上就实例化相应的bean，此时容器仅仅拥有所有对象的BeanDefinition(BeanDefinition：是容器依赖某些工具加载的XML配置信息进行解析和分析，并将分析后的信息编组为相应的BeanDefinition)。只有当getBean()调用时才是有可能触发Bean实例化阶段的活动

而有一些内容就不会说解释的很透彻，比如说为什么说只有当 getBean() 调用的时候才有可能触发Bean的实例化。

## 2.生命周期流程图

### 2.1简化版图解

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/06-07/1.png)

而这图解中，把 Spring 中 Bean 的生命周期分成了好几个步骤，分别是：

1. 通过构造方法实例化 Bean 对象。

2. 通过 setter 方法设置对象的属性。

3. 通过Aware，也就是他的子类BeanNameAware，调用Bean的setBeanName()方法传递Bean的ID(XML里面注册的ID)，setBeanName方法是在bean初始化时调用的，通过这个方法可以得到BeanFactory和 Bean 在 XML 里面注册的ID。

4. 如果说 Bean 实现了 BeanFactoryAware,那么工厂调用setBeanFactory(BeanFactory var1) 传入的参数也是自身。

5. 把 Bean 实例传递给 BeanPostProcessor 中的 postProcessBeforeInitialization 前置方法。

6. 完成 Bean 的初始化

7. 把 Bean 实例传递给 BeanPostProcessor 中的 postProcessAfterInitialization 后置方法。

8. 此时 Bean 已经能够正常时候，在最后的时候调用 DisposableBean 中的 destroy 方法进行销毁处理。

而阿粉觉得如果面试官在面试的时候问到这个问题的时候，你从图解开始入手，然后把这些都说给他之后，那么相对应的，这现在这些答案，如果不继续的深挖内容，可能已经就足够了。

而接下来还要从根本上来论证阿粉所写的内容。

而我们对这详细的可能有时候难以记忆，可能还是理解不深，而我们可以从四到五个方面来记忆，

- 构造实例化
- 属性赋值
- 完成初始化
- (前后处理)
- 使用后销毁

而从这五个方面来记忆，或许就能把这个图扩展开，从而言简意赅的回答面试官的问题。

代码验证
```

package com.yld.bean;

import org.springframework.beans.factory.BeanNameAware;

public class Person implements BeanNameAware {

    private String name;

    /**
     * 实现类上的override方法
     * @param s
     */
    @Override
    public void setBeanName(String s) {
        System.out.println("调用BeanNameAware中的setName赋值");
    }

    public Person() {
    }

    /**
     * 属性赋值
     * @param name
     */
    public void setName(String name) {
        System.out.println("设置对象属性setName()..");
        this.name = name;
    }

    /**
     * Bean初始化
     */
    public void initBeanPerson() {
        System.out.println("初始化Bean");
    }

    /**
     * Bean方法使用：说话
     */
    public void speak() {
        System.out.println("使用Bean的Speak方法");
    }

    /**
     * 销毁Bean
     */
    public void destroyBeanPerson() {
        System.out.println("销毁Bean");
    }


}

```

**Main**方法

```
public static void main(String[] args) {
        ClassPathXmlApplicationContext pathXmlApplicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        Person person = (Person)pathXmlApplicationContext.getBean("person");
        person.speak();
        pathXmlApplicationContext.close();
    }
```

**运行结果展示**

```
D:\develop\JDK8\jdk1.8.0_181\bin\java.exe "-javaagent:D:\develop\IDEA\IntelliJ IDEA 2018.1.8\lib\idea_rt.jar=63906:D:\develop\IDEA\IntelliJ IDEA 2018.1.8\bin" -Dfile.encoding=UTF-8 -classpath D:\develop\JDK8\jdk1.8.0_181\jre\lib\charsets.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\deploy.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\access-bridge-64.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\cldrdata.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\dnsns.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\jaccess.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\jfxrt.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\localedata.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\nashorn.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\sunec.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\sunjce_provider.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\sunmscapi.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\sunpkcs11.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\zipfs.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\javaws.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\jce.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\jfr.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\jfxswt.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\jsse.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\management-agent.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\plugin.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\resources.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\rt.jar;D:\develop\IDEAProject\KaiYuan\target\classes;C:\Users\Administrator\.m2\repository\org\springframework\boot\spring-boot-starter\2.1.8.RELEASE\spring-boot-starter-2.1.8.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\boot\spring-boot\2.1.8.RELEASE\spring-boot-2.1.8.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-context\5.1.9.RELEASE\spring-context-5.1.9.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-aop\5.1.9.RELEASE\spring-aop-5.1.9.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-beans\5.1.9.RELEASE\spring-beans-5.1.9.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-expression\5.1.9.RELEASE\spring-expression-5.1.9.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\boot\spring-boot-autoconfigure\2.1.8.RELEASE\spring-boot-autoconfigure-2.1.8.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\boot\spring-boot-starter-logging\2.1.8.RELEASE\spring-boot-starter-logging-2.1.8.RELEASE.jar;C:\Users\Administrator\.m2\repository\ch\qos\logback\logback-classic\1.2.3\logback-classic-1.2.3.jar;C:\Users\Administrator\.m2\repository\ch\qos\logback\logback-core\1.2.3\logback-core-1.2.3.jar;C:\Users\Administrator\.m2\repository\org\apache\logging\log4j\log4j-to-slf4j\2.11.2\log4j-to-slf4j-2.11.2.jar;C:\Users\Administrator\.m2\repository\org\apache\logging\log4j\log4j-api\2.11.2\log4j-api-2.11.2.jar;C:\Users\Administrator\.m2\repository\org\slf4j\jul-to-slf4j\1.7.28\jul-to-slf4j-1.7.28.jar;C:\Users\Administrator\.m2\repository\javax\annotation\javax.annotation-api\1.3.2\javax.annotation-api-1.3.2.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-core\5.1.9.RELEASE\spring-core-5.1.9.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\springframework\spring-jcl\5.1.9.RELEASE\spring-jcl-5.1.9.RELEASE.jar;C:\Users\Administrator\.m2\repository\org\yaml\snakeyaml\1.23\snakeyaml-1.23.jar;C:\Users\Administrator\.m2\repository\org\slf4j\slf4j-api\1.7.28\slf4j-api-1.7.28.jar com.yld.bean.Test
16:54:58.817 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@123772c4
16:54:59.074 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 1 bean definitions from class path resource [applicationContext.xml]
16:54:59.121 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'person'

设置对象属性setName()..

调用BeanNameAware中的setName赋值

初始化Bean

使用Bean的Speak方法

16:54:59.232 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Closing org.springframework.context.support.ClassPathXmlApplicationContext@123772c4, started on Sun Jun 07 16:54:58 CST 2020

销毁Bean

Process finished with exit code 0

```

和大家预想的是不是一样的呢? 在用案例回答面试官之后，我们最好还是要研究一下源码的部分，毕竟研究清楚了，会理解的更深刻不是么？

**InstantiationAwareBeanPostProcessor**

这个类是继承的 BeanPostProcessor 而这个类的作用是什么呢？源码注释解释的是这样子的：

方法一：

```
@Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}
应用这个Bean处理器在目标Bean实例化之前。返回的bean对象可能是一个代理bean的使用而不是目标,
```

也就是说postProcessBeforeInstantiation在bean实例化之前调用的，这是不是也是我们在面试中另外的一个面试点 AOP 的使用呢？到时候面试官让你举例子的时候，你直接用这个 Spring 里面的源码给他解释，分分钟让面试官对你刮目想看呀有木有。

方法二：
可以看到该方法在属性赋值方法内，但是在真正执行赋值操作之前。其返回值为boolean。
```
default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
```

大家是不是还可以这么理解，如果返回值为false的话，那么就出现了赋值失败，也就是间接阻断赋值了。

而初始化的类同样的 BeanPostProcessor 

方法一：
```
任何Bean之前初始化回调如初始化Bean的属性设置后
@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

```

方法二：

```
应用这个Bean后置处理程序给定新的Bean实例，任何Bean初始化后回调(如初始化Bean的属性设置后{@code}或一个自定义的init方法)。bean已经填充属性值。返回的bean实例可能是原始的包装器。
@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}


```


同样注释翻译出来的意思也是很明确的，这也是阿粉为什么喜欢自己下载个插件去看注释，毕竟源码这个东西如果看别人理解的和自己理解的，有时候差距也是很大的。

关于这个SpringBean的高频面试题，你会回答了么？

## 文献参考
《Spring源码深度解析》

## 代码参考
Spring 了解Bean的一生(生命周期)

