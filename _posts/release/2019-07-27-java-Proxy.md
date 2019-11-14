---
layout: post
title: 代理到底是什么？
tagline: by 懿
categories: 代理
tags:
    - 懿
---

之前星球的球友面试，问了我一些问题，说让我写一下这个代理，和代理到底是根据什么来进行区分，又该在什么地方使用。这篇文章我细致的讲解一下关于代理的一些问题。
<!--more-->

## 代理分类

1. 静态代理

2. 动态搭理

## 静态代理

我们先说静态代理的实现方式，为什么不推荐使用静态代理？

1.继承方式实现代理（静态代理中的继承代理）

```
//目标对象
public class UserImpl {
    public void system(){
        System.out.println("输出测试");
    }
}
//代理对象
public class Proxy extends UserImpl {
    @Override
    public void system() {
        super.system();
        System.out.println("增强之后的输出");
    }
}
//测试类
public class TestMain {
    public static void main(String[] args) {
        UserImpl user = new Proxy();
        user.system();
    }
}

```

静态代理可以看出来一点问题吧？

每次代理都要实现一个类，导致项目中代码很多；你每次想要代理，都要去实现一个类，代码就会成堆的增加，然后你就会发现项目的类就会越来越多，就会导致你们的项目显得很臃肿。而且代码的复用性太低了，并且耦合度非常高，这个我们所说的高内聚低耦合是相悖的。

## 动态代理

我们在看一下这个动态代理：

```
    //接口类
    public interface Italk {
        public void talk(String msg);
    }
    //实现类
    public class People implements Italk {
            public String username;
            public String age;
            public String getName() {
            return username;
        }
        public void setName(String name) {
            this.username= name;
        }
        public String getAge() {
            return age;
        }
        public void setAge(String age) {
            this.age = age;
        }
        public People(String name1, String age1) {
            this.username= name1;
            this.age = age1;
        }
        public void talk(String msg) {
            System.out.println(msg+"!你好,我是"+username+"，我年龄是"+age);
         }
    }
    
    //代理类
    public class TalkProxy implements Italk {
            Italk talker;
        public TalkProxy (Italk talker) {
            //super();
            this.talker=talker;
        }
        public void talk(String msg) {
            talker.talk(msg);
        }
        public void talk(String msg,String singname) {
            talker.talk(msg);
            sing(singname);
        }
        private void sing(String singname){
             System.out.println("唱歌："+singname);
        }
    }
    
    //测试
    
    public class MyProxyTest {
        //代理模式
        public static void main(String[] args) {
        
        //不需要执行额外方法的
        Italk people1=new People("湖海散人","18");
        people1.talk("No ProXY Test");
        System.out.println("-----------------------------");
        
        //需要执行额外方法的
        TalkProxy talker=new TalkProxy(people1);
        talker.talk("ProXY Test","七里香");
        }
    }

```

上面代码解析

一个 `Italk` 接口，有空的方法 `talk（）`（说话），所有的 `people` 对象都实现（implements）这个接口，实现 `talk()` 方法，前端有很多地方都将 `people ` 实例化，执行 `talk` 方法，后来发现这些前端里有一些除了要说话以外还要唱歌（sing），那么我们既不能在 `Italk` 接口里增加 `sing()` 方法，又不能在每个前端都增加 `sing` 方法，我们只有增加一个代理类 `talkProxy` ，这个代理类里实现 `talk` 和 `sing` 方法，然后在需要 `sing` 方法的客户端调用代理类即可，

这也是实现动态代理的方式，是通过实现（implements）的方式来实现的，这种方法的优点，在编码时，代理逻辑与业务逻辑互相独立，各不影响，没有侵入，没有耦合。

## cgLib代理

还有一种是cgLib的代理，这种代理则是适合那些没有接口抽象的类代理，而Java 动态代理适合于那些有接口抽象的类代理。

我们来通过代码了解一下他到底是怎么玩的。

```
//我们实现一个业务类，不实现任何的接口
    /**
     * 业务类，
     */
    public class TestService {
        public TestService() {
            System.out.println("TestService的构造");
        }
    
        /**
         * 该方法不能被子类覆盖,Cglib是无法代理final修饰的方法的
         */
        final public String sayOthers(String name) {
            System.out.println("TestService:sayOthers>>"+name);
            return null;
        }
    
        public void sayHello() {
            System.out.println("TestService:sayHello");
        }
    
    }


/**
 * 自定义MethodInterceptor
 */
public class MethodInterceptorTest implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("======插入前置通知======");
        Object object = methodProxy.invokeSuper(o, objects);
        System.out.println("======插入后者通知======");
        return object;
    }

}

    /**
     * 测试
     */
    public class Client {
        public static void main(String[] args) {
            // 代理类class文件存入本地磁盘方便我们反编译查看源码
            System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "D:\\code");
            // 通过CGLIB动态代理获取代理对象的过程
            Enhancer enhancer = new Enhancer();
            // 设置enhancer对象的父类
            enhancer.setSuperclass(TestService.class);
            // 设置enhancer的回调对象
            MethodInterceptorTest t =  new  MethodInterceptorTest();
            enhancer.setCallback(t);
            // 创建代理对象
            TestService proxy= (TestService)enhancer.create();
            // 通过代理对象调用目标方法
            proxy.sayHello();
        }
    }
上面代码想要运行，需要cglib的jar和asm的jar不要忘记找
```

运行结果
```
CGLIB debugging enabled, writing to 'D:\code'
TestService的构造
======插入前置通知======
TestService:sayHello
======插入后者通知======

```

实现CGLIB动态代理必须实现MethodInterceptor(方法拦截器)接口，

这个接口只有一个intercept()方法，这个方法有4个参数：

- obj表示增强的对象，即实现这个接口类的一个对象；

- method表示要被拦截的方法；

- args表示要被拦截方法的参数；

- proxy表示要触发父类的方法对象；

## 代理的使用
那么什么时候使用静态态代理，什么时候使用动态代理和cgLib代理呢？

一般情况静态代理是很少是用的，因为他对代码的复用性或者说是耦合度都非常不友好，不推荐使用。

如果目标对象至少实现了一个接口，那么就用JDK动态代理，所有由目标对象实现的接口将全部都被代理。

如果目标对象没有实现任何接口，就是个类，那么就用CGLIB代理。

我是懿，一个正在被打击还在努力前进的码农。欢迎大家关注我们的公众号，加入我们的知识星球，我们在知识星球中等着你的加入。

