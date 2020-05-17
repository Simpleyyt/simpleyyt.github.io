---
layout: post
title: 【Spring 一】研究一下Spring里面的源码，发现好恐怖，循环依赖你会么？
catgories: java
tags:
  - 懿
---

前几天在阿粉CSDN上看一个文章，因为一个Spring的问题，期望薪资三万却被生生的压榨成了两万五，高于两万五人家都不要，让我感觉到了Spring的强大，不学习Spring是会吃亏的，那么我们就从各种高频面试来一点点深入吧。
<!--more-->

本文先从一个高频面试题开始说起吧！阿粉在接下来会给大家带来各种关于Spring的知识，希望大家能够一起讨论一下呦

## Spring是怎么去解决循环依赖的

### 1.什么是循环依赖

这个词，阿粉听到的时候，肯定和大家的反应一样的，循环，依赖，那是不是 A 引用了 B ，而此时 B 引用了 C,而 C 呢又引用了A，于是一个三角恋的关系出现了。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/05-16/1.jpg)

那么用代码来表示的话，是怎么表示的呢？

```
public class ClassTestA {

    private ClassTestB classTestB;

    public void a(){
        classTestB.b();
    }

    public ClassTestB getClassTestB() {
        return classTestB;
    }
    private void setClassTestB(ClassTestB classTestB){
        this.classTestB = classTestB;
    }
}

public class ClassTestB {

    private ClassTestC classTestC;

    public void b(){
        classTestC.c();
    }

    public ClassTestC getClassTestC() {
        return classTestC;
    }
    private void setClassTestC(ClassTestC classTestC){
        this.classTestC = classTestC;
    }
}

public class ClassTestC {

    private ClassTestA classTestA;

    public void c(){
        classTestA.a();
    }

    public ClassTestA getClassTestA() {
        return classTestA;
    }
    private void setClassTestA(ClassTestA classTestA){
        this.classTestA = classTestA;
    }
}

```

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/05-16/2.jpg)

### 2.循环依赖会出现什么问题

在阿粉的印象中，循环依赖最直接的问题就是会出现在对象的实例化上面，创建对象的时候，如果在Spring的配置中加入这种 A 依赖 B ，B 依赖 C，C 依赖 A 的话，那么最终创建 A 的实例对象的时候，会出现错误。

而如果这种循环调用的依赖不去终结掉他的话，那么就相当于一个死循环，就像阿粉前几天的在维护那个 “十六年”之前的项目的时候，各种内存溢出，表示内心很压抑呀。

而 Spring 中也将循环依赖的处理分成了不同的几种情况，阿粉带大家来看一下吧。

### 3.Spring循环依赖处理一 (构造器循环依赖)

构造器循环依赖的意思就是说，通过构造及注入构成的循环依赖，而这种依赖的话，是没有办法解决的，如果你敢强行依赖，不要意思，出现了你久违的异常 BeanCurrentlyInCreationException 出现这个异常的时候，就是表示循环依赖的问题。

相信大家出现异常的时候，在看不懂为什么的时候，第一时间，复制异常信息，放在百度，或者Google上面查询一下，BeanCurrentlyInCreationException 放在百度上，一目了然。

而在 Spring 的配置文件中，如果这么配置 A ，B ，C 的循环依赖的时候，在创建 A 的时候，发现，构造器需要 B 类，然后去创建 B ,而创建 B 的时候，发现又需要 C ，然后去创建 C ，创建的时候发现，竟然需要 A ，于是又掉头回去了，于是就形成了一个闭环，没有办法创建。

```
<beans>

  <bean id="ClassTestA" class="com.yldlsy.ClassTestA" >
        <constructor-arg index="0" ref="ClassTestB" />
  </bean>

  <bean id="ClassTestB" class="com.yldlsy.ClassTestB" >
        <constructor-arg index="0" ref="ClassTestC" />
  </bean>

  <bean id="ClassTestC" class="com.yldlsy.ClassTestC" >
         <constructor-arg index="0" ref="ClassTestA" />
  </bean>
</beans>

```

而在这种情况下，Spring实例化bean是通过ApplicationContext.getBean()方法来进行的。如果要获取的对象依赖了另一个对象，那么其首先会创建当前对象，然后通过递归的调用ApplicationContext.getBean()方法来获取所依赖的对象，最后将获取到的对象注入到当前对象中。而和刚才阿粉说的一样，创建了闭环，所以就没有办法创建了。

### 4.Spring循环依赖处理二（setter循环依赖）

setter循环注入是指通过setter注入方式构成的循环依赖。而这种方式，是Spring可以进行解决的。

而对于这种使用setter注入造成的依赖是通过Spring容器来提前暴露刚完成的构造注入器的bean来完成的，但是这时候还没有完成其他的步骤的时候。

这个时候我们就需要提前暴露出来一个单例的工厂方法，让其他的bean来引用这个bean，

```
addSingletonFactory(beanName，new ObjectFactory(){
    public Object getObject() throws BeanException{
        return getEarlyBeanReference(beanName,mbd,bean)
    }
})

```

- Spring在创建 A 的时候，根据无参构造来创建 A，并且暴露出 ObjectFactory 用来返回一个提前暴露好的 bean 然后再进行setter来注入，

同理的B和C都是这个样子的，这个时候就能完成setter注入了。

### 5.Spring循环依赖处理三（作用域循环依赖）

阿粉带大家看一下这个配置方式

```

    <bean id="ClassTestA" class="com.yldlsy.ClassTestA" scope="singleton" >
        <property name="ClassTestB" ref="ClassTestB" />
    </bean>
    <bean id="ClassTestA" class="com.yldlsy.ClassTestA" scope="singleton" >
            <property name="ClassTestB" ref="ClassTestB" />
     </bean>
     <bean id="ClassTestA" class="com.yldlsy.ClassTestA" scope="singleton" >
             <property name="ClassTestB" ref="ClassTestB" />
      </bean>

```

而对于 “singleton”作用域的话，他是可以通过“setAllowCircularReference（false）”这种方式来进制循环依赖的。

而且也是有缺陷的，这种方式只能解决单例作用域的bean循环依赖。

关于面试中高频的Spring解决循环依赖你会了么？


## 文献参考
《Spring源码深度解析》



