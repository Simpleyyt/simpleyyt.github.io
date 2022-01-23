---

layout: post
categories: java
title: 模版模式
tagline: by 炸鸡可乐
tags: 
  - 炸鸡可乐
---

模版模式又叫模板方法模式，在一个方法中定义算法的骨架，而将一些步骤延迟到子类中去实现。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

<!--more-->
### 一、介绍
> 模板方法模式是类的行为模式

准备一个抽象类，将部分逻辑以具体方法以及具体构造函数的形式实现，然后声明一些抽象方法来迫使子类实现剩余的逻辑。

不同的子类可以以不同的方式实现这些抽象方法，从而对剩余的逻辑有不同的实现，这就是模板方法模式的用意。

模板模式涉及到三个角色：

* 抽象类（AbstractClass）：实现了模板方法，定义了算法的骨架；
* 具体类（ConcreteClass）：实现抽象类中的抽象方法，已完成完整的算法；
* 客户角色：客户类提出使用具体类的请求；

### 二、示例
举个例子，以早上起床到上班所需要的操作为例，大致流程可以分为以下几步：穿衣服、刷牙、洗脸、吃早餐等。男生和女生的操作可能有些区别。

我们创建一个抽象的类，定义好大致的操作流程，如下：
```java
/**
 * 抽象类
 */
public abstract class AbstractPerson {

    /**
     * 定义操作流程
     */
    public void prepareGoWorking(){
        dressing();//穿衣服
        brushTeeth();//刷牙
        washFace();//洗脸
        eatBreakFast();//吃早餐
    }

    /**穿衣服*/
    protected abstract void dressing();

    /**刷牙*/
    protected void brushTeeth(){
        System.out.println("刷牙");
    }

    /**洗脸*/
    protected void washFace(){
        System.out.println("洗脸");
    }

    /**吃早餐*/
    protected abstract void eatBreakFast();

}
```
因为男生和女生的行为不一样，我们分别创建两个具体类，如下：
```java
/**
 * 男生
 * 具体实现类
 */
public class ManPerson extends AbstractPerson{

    @Override
    protected void dressing() {
        System.out.println("穿西装");
    }

    @Override
    protected void eatBreakFast() {
        System.out.println("直接在公司吃早餐");
    }
}
```
```java
/**
 * 女生
 * 具体实现类
 */
public class WomanPerson extends AbstractPerson{

    @Override
    protected void dressing() {
        System.out.println("穿休闲衣服");
    }

    @Override
    protected void eatBreakFast() {
        System.out.println("在家弄点吃的，或者在外面买一点小吃");
    }
}
```
创建一个客户端，实现如下：
```java
public class TemplateClient {

    public static void main(String[] args) {
        //男生起床步骤
        ManPerson manPerson = new ManPerson();
        System.out.println("-----男生起床步骤----");
        manPerson.prepareGoWorking();
        System.out.println("-----女生起床步骤----");
        //女生起床步骤
        WomanPerson womanPerson = new WomanPerson();
        womanPerson.prepareGoWorking();
    }
}
```
输出结果：
```java
-----男生起床步骤----
穿西装
刷牙
洗脸
直接在公司吃早餐
-----女生起床步骤----
穿休闲衣服
刷牙
洗脸
在家弄点吃的，或者在外面买一点小吃
```
当然，模版模式的玩法，还不仅仅只有这些，还可以**在模版模式中使用挂钩(hook)**。

什么是`hook`呢？**存在一个空实现的方法，我们称这种方法为`hook`**。子类可以视情况来决定是否要覆盖它。

还是以上面为例子，比如吃完早餐就要出门上班，选择什么交通工具呢？

抽象类新增方法`hook()`，内容如下：
```java
/**
 * 抽象类
 */
public abstract class AbstractPerson {

    /**
     * 定义操作流程
     */
    public void prepareGoWorking(){
        dressing();//穿衣服
        brushTeeth();//刷牙
        washFace();//洗脸
        eatBreakFast();//吃早餐
        hook();//挂钩
    }

    /**穿衣服*/
    protected abstract void dressing();

    /**刷牙*/
    protected void brushTeeth(){
        System.out.println("刷牙");
    }

    /**洗脸*/
    protected void washFace(){
        System.out.println("洗脸");
    }

    /**吃早餐*/
    protected abstract void eatBreakFast();

    /**挂钩*/
    protected void hook(){};

}
```
男生具体实现类，重写`hook()`方法，内容如下：
```java
/**
 * 男生
 * 具体实现类
 */
public class ManPerson extends AbstractPerson{

    @Override
    protected void dressing() {
        System.out.println("穿西装");
    }

    @Override
    protected void eatBreakFast() {
        System.out.println("直接在公司吃早餐");
    }

    @Override
    protected void hook() {
        System.out.println("乘地铁上班");
    }
}
```
运行测试类，男生具体实现类，输出结果：
```java
-----男生起床步骤----
穿西装
刷牙
洗脸
直接在公司吃早餐
乘地铁上班
```
当然，还有其他的玩法，比如女生洗完脸之后，可能需要化妆，我们再次将抽象类进行处理，内容如下：
```java
/**
 * 抽象类
 */
public abstract class AbstractPerson {

    /**
     * 定义操作流程
     */
    public void prepareGoWorking(){
        dressing();//穿衣服
        brushTeeth();//刷牙
        washFace();//洗脸
        //是否需要化妆，默认不化妆
        if(isMakeUp()){
            System.out.println("进行化妆");
        }
        eatBreakFast();//吃早餐
        hook();//挂钩
    }

    /**是否需要化妆方法*/
    protected boolean isMakeUp(){
        return false;
    }

    /**穿衣服*/
    protected abstract void dressing();

    /**刷牙*/
    protected void brushTeeth(){
        System.out.println("刷牙");
    }

    /**洗脸*/
    protected void washFace(){
        System.out.println("洗脸");
    }

    /**吃早餐*/
    protected abstract void eatBreakFast();

    /**挂钩*/
    protected void hook(){};

}
```
女生具体实现类，重写`isMakeUp()`方法，内容如下：
```java
/**
 * 女生
 * 具体实现类
 */
public class WomanPerson extends AbstractPerson{

    @Override
    protected void dressing() {
        System.out.println("穿休闲衣服");
    }

    @Override
    protected void eatBreakFast() {
        System.out.println("在家弄点吃的，或者在外面买一点小吃");
    }

    @Override
    protected boolean isMakeUp() {
        return true;
    }
}
```
运行测试类，女生具体实现类，输出结果：
```java
-----女生起床步骤----
穿休闲衣服
刷牙
洗脸
进行化妆
在家弄点吃的，或者在外面买一点小吃
```

### 三、应用
模版设计模式，应用非常广泛，比如javaEE中的servlet，当我们每创建一个servlet的时候，都会继承`HttpServlet`，其实`HttpServlet`已经为我们提供一套操作流程，我们只需要重写里面的方法即可！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-single/6a954b0597ef4d2ca83c866b5ab57a48.jpg)

HttpServlet 的部分源码如下：
```java
public abstract class HttpServlet extends GenericServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // ...
    }
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // ...
    }
    protected void doHead(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // ...
    }
    protected void doPut(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // ...
    }
    protected void doDelete(HttpServletRequest req,  HttpServletResponse resp) throws ServletException, IOException {
        // ...
    }
    protected void doOptions(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // ...
    }
    protected void doTrace(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // ...
    }
    
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                if (ifModifiedSince < lastModified) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);
            
        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);
            
        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);
            
        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);
            
        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);
            
        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);
            
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
    
    // ...省略...
}
```
自定义一个 HelloWorld 的 Servlet 类，如下：
```java
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;

public class HelloWorld extends HttpServlet {

  public void init() throws ServletException {
    // ...
  }

  public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
      response.setContentType("text/html");
      PrintWriter out = response.getWriter();
      out.println("<h1>Hello World!</h1>");
  }
  
  public void destroy() {
      // ...
  }
}
```
### 四、总结
**模版模式有着许多的优点：**
1、模板方法模式通过把不变的行为搬移到超类，去除了子类中的重复代码；
2、子类实现算法的某些细节，有助于算法的扩展；
3、通过一个父类调用子类实现的操作，通过子类扩展增加新的行为，符合`开放-封闭原则`；

**也有些缺点：**
1、每个不同的实现都需要定义一个子类，这会导致类的个数的增加，设计更加抽象

如果某些类有一些共同的行为，可以使用模版设计模式，创建一个抽象类，将共同的行为定义在抽象类中，可以有效的减少子类重复的代码。
### 五、参考
[CSDN - 炸斯特 - 模板方法模式](https://blog.csdn.net/jason0539/article/details/45037535)