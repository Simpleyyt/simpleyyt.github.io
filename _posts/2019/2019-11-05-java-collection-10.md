---
layout: post
title: 【集合系列】- 深入浅出的分析 Properties
tagline: by 炸鸡可乐
categories: Java
tags: 
  - 炸鸡可乐
---

最近在看 java 集合源码的时候，发现原来我们经常使用的 Properties 类既然继承自 Hashtable！又涨见识了！


<!--more-->
### 01、摘要
在集合系列的第一章，咱们了解到，Map 的实现类有 HashMap、LinkedHashMap、TreeMap、IdentityHashMap、WeakHashMap、Hashtable、Properties 等等。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/10bfb0b009e142c7a3aa11f5f92a25f5.jpg)

在上一章节中，咱们介绍到 Hashtable 的数据结构和算法实现，在 Java 中其实还有一个非常重要的类** Properties，它继承自 Hashtable，主要用于读取配置文件。**

本文通过看 JDK 和一些网友的博客总结，主要从 Properties 的用法实例来做介绍，如果有理解不当之处，欢迎指正。

### 02、简介
Properties 类是 java 工具包中非常重要的一个类，比如在实际开发中，有些变量，我们可以直接硬写入到自定义的 java 枚举类中。

但是有些变量，**在测试环境、预生产环境、生产环境，变量所需要取的值都不一样**，这个时候，我们可以通过使用 properties 文件来加载程序需要的配置信息，以达到一行代码，多处环境都可以运行的效果！

最常见的比如 JDBC 数据源配置文件，`properties`文件以`.properties `作为后缀，文件内容以`键=值`格式书写，左边是变量名称，右边是变量值，用`#`做注释，比如新建一个`jdbc.properties`文件，内容如下：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/7c58699f5e1047ef92ce000d7cc01757.jpg)

Properties 类是 properties 文件和程序的中间桥梁，不论是从 properties 文件读取信息，还是写入信息到 properties 文件，都要经由 Properties 类。

好了，唠叨了这么多，咱们回到本文要介绍的主角 **Properties**！

从集合 Map 架构图可以看出，Properties 继承自 Hashtable，表示一个持久的 map 集合，属性列表以 key-value 的形式存在，Properties 类定义如下：

```java
public class Properties extends Hashtable<Object,Object> {
    ......
}
```
Properties 除了继承 Hashtable 中所定义的方法，Properties 也定义了以下几个常用方法，如图所示：

![](http://www.justdojava.com/assets/images/2019/java/image-jay/3c281e8984af48aca86aff4faa682b93.jpg)

#### 2.1、常用方法介绍
##### 2.1.1、set 方法（添加修改元素）
set 方法是将指定的 key, value 对添加到 map 里，在添加元素的时候，调用了 Hashtable 的 put 方法，**与 Hashtable 不同的是， key 和 value 都是字符串。**

打开 Properties 的 setProperty 方法，源码如下：
```java
public synchronized Object setProperty(String key, String value) {
    //调用父类 Hashtable 的 put 方法
    return put(key, value);
}
```
方法测试如下：
```java
public static void main(String[] args) {
    Properties properties = new Properties();
    properties.setProperty("name1","张三");
    properties.setProperty("name2","张四");
    properties.setProperty("name3","张五");
    System.out.println(properties.toString());
}
```
输出结果：
```java
{name3=张五, name2=张四, name1=张三}
```
##### 2.1.2、get 方法（搜索指定元素）
get 方法根据指定的 key 值返回对应的 value，第一步是从调用 `Hashtable` 的 get 方法，如果有返回值，直接返回；如果没有返回值，但是初始化时传入了`defaults`变量，从 `defaults` 变量中，也就是 `Properties` 中，去搜索是否有对于的变量，如果有就返回元素值。

打开 Properties 的 getProperty 方法，源码如下：
```java
public String getProperty(String key) {
    //调用父类 Hashtable 的 get 方法
    Object oval = super.get(key);
    String sval = (oval instanceof String) ? (String)oval : null;
     //进行变量非空判断
    return ((sval == null) && (defaults != null)) ? defaults.getProperty(key) : sval;
}
```
查看 defaults 这个变量，源码如下：
```java
public class Properties extends Hashtable<Object,Object> {
    protected Properties defaults;
}
```
这个变量在什么时候赋值呢，打开源码如下：
```java
public Properties(Properties defaults) {
    this.defaults = defaults;
}
```
可以发现，在 Properties 构造方法初始化阶段，如果你给了一个自定义的 defaults ，当调用 `Hashtable` 的 get 方法没有搜索到元素值的时候，并且 defaults 也不等于空，那么就会进一步在 defaults 里面进行搜索元素值。

方法测试如下：
```java
public static void main(String[] args) {
    Properties properties = new Properties();
    properties.setProperty("name1","张三");
    properties.setProperty("name2","张四");
    properties.setProperty("name3","张五");
    //将 properties 作为参数初始化到 newProperties 中
    Properties newProperties = new Properties(properties);
    newProperties.setProperty("name4","李三");
    //查询key中 name1 的值
    System.out.println("查询结果：" + properties.getProperty("name1"));
}
```
输出结果：
```java
通过key查询结果：张三
```
##### 2.1.3、load方法（加载配置文件）
load 方法，表示将 properties 文件以输入流的形式加载文件，并且提取里面的键、值对，将键值对元素添加到 map 中去。

打开 Properties 的 load 方法，源码如下：
```java
public synchronized void load(InputStream inStream) throws IOException {
    //读取文件流
    load0(new LineReader(inStream));
}
```
load0 方法，源码如下：
```java
private void load0 (LineReader lr) throws IOException {
    char[] convtBuf = new char[1024];
    int limit;
    int keyLen;
    int valueStart;
    char c;
    boolean hasSep;
    boolean precedingBackslash;

    //一行一行的读取
    while ((limit = lr.readLine()) >= 0) {
        c = 0;
        keyLen = 0;
        valueStart = limit;
        hasSep = false;

        precedingBackslash = false;
        //判断key的长度
        while (keyLen < limit) {
            c = lr.lineBuf[keyLen];
            if ((c == '=' ||  c == ':') && !precedingBackslash) {
                valueStart = keyLen + 1;
                hasSep = true;
                break;
            } else if ((c == ' ' || c == '\t' ||  c == '\f') && !precedingBackslash) {
                valueStart = keyLen + 1;
                break;
            }
            if (c == '\\') {
                precedingBackslash = !precedingBackslash;
            } else {
                precedingBackslash = false;
            }
            keyLen++;
        }
        //获取值的起始位置
        while (valueStart < limit) {
            c = lr.lineBuf[valueStart];
            if (c != ' ' && c != '\t' &&  c != '\f') {
                if (!hasSep && (c == '=' ||  c == ':')) {
                    hasSep = true;
                } else {
                    break;
                }
            }
            valueStart++;
        }
        //获取文件中的键和值参数
        String key = loadConvert(lr.lineBuf, 0, keyLen, convtBuf);
        String value = loadConvert(lr.lineBuf, valueStart, limit - valueStart, convtBuf);
        //调用 Hashtable 的 put 方法，将键值加入 map 中
        put(key, value);
    }
}
```
好了，我们来在`src/recources`目录下，新建一个`custom.properties`配置文件，内容如下：
```java
#定义一个变量名称和值
userName=李三
userPwd=123456
userAge=18
userGender=男
userEmail=123@123.com
```
方法测试如下：
```java
public class TestProperties  {

    public static void main(String[] args) throws Exception {
        //初始化 Properties
        Properties prop = new Properties();
        //加载配置文件
        InputStream in = TestProperties .class.getClassLoader().getResourceAsStream("custom.properties");
        //读取配置文件，指定编码格式，避免读取中文乱码
        prop.load(new InputStreamReader(in, "UTF-8"));
        //将内容输出到控制台
        prop.list(System.out);
    }
}
```
输出结果：
```java
userPwd=123456
userEmail=123@123.com
userAge=18
userName=李三
userGender=男
```
##### 2.1.4、propertyNames方法（读取全部信息）
propertyNames 方法，表示读取 Properties 的全部信息，**本质是创建一个新的 Hashtable 对象，然后将原 Hashtable 中的数据复制到新的 Hashtable 中，并将 map 中的 key 全部返回。**

打开 Properties 的 propertyNames 方法，源码如下：
```java
public Enumeration<?> propertyNames() {
    Hashtable<String,Object> h = new Hashtable<>();
    //将原 map 添加到新的 Hashtable 中
    enumerate(h);
    //返回 Hashtable 中全部的 key 元素
    return h.keys();
}
```
enumerate 方法，源码如下：
```java
private synchronized void enumerate(Hashtable<String,Object> h) {
    //判断 Properties 中是否有初始化的配置文件
    if (defaults != null) {
        defaults.enumerate(h);
    }
    //将原 Hashtable 中的数据添加到新的 Hashtable 中
    for (Enumeration<?> e = keys() ; e.hasMoreElements() ;) {
        String key = (String)e.nextElement();
        h.put(key, get(key));
    }
}
```
方法测试如下：
```java
public static void main(String[] args) throws Exception {
    //初始化 Properties
    Properties prop = new Properties();
    //加载配置文件
    InputStream in = TestProperties.class.getClassLoader().getResourceAsStream("custom.properties");
    //读取配置文件，指定读取编码 UTF-8，防止内容乱码
    prop.load(new InputStreamReader(in, "UTF-8"));
    //获取 Properties 中全部的 key 元素
    Enumeration enProp = prop.propertyNames();
    while (enProp.hasMoreElements()){
        String key = (String) enProp.nextElement();
        String value = prop.getProperty(key);
        System.out.println(key + "=" + value);
    }
}
```
输出内容如下：
```java
userPwd=123456
userEmail=123@123.com
userAge=18
userName=李三
userGender=男
```
##### 2.1.5、总结
Properties 继承自 Hashtable，大部分方法都复用于 Hashtable，比如，get、put、remove、clear 方法，**与 Hashtable 不同的是， Properties中的 key 和 value 都是字符串，**如果需要获取 properties 中全部内容，可以先通过迭代器或者 propertyNames 方法获取 map 中所有的 key 元素，然后遍历获取 key 和 value。

需要注意的是，Properties 中的 setProperty 、load 方法，都加了`synchronized`同步锁，用来控制线程同步。

### 03、properties 文件的加载方式
在实际开发中，**经常会遇到读取配置文件路径找不到，或者读取文件内容乱码的问题**，下面简单介绍一下，properties 文件的几种常用的加载方式。

properties 加载文件的方式，大致可以分两类，第一类是使用 `java.util.Properties` 的 load 方法来加载文件流；第二类是使用 `java.util.ResourceBundle` 类来获取文件内容。

在`src/recources`目录下，新建一个`custom.properties`配置文件，文件编码格式为`UTF-8`，内容还是以刚刚那个测试为例，各个加载方式如下！
#### 3.1、通过文件路径来加载文件
这类方法加载文件，主要是调用 Properties 的 load 方法，获取文件路径，读取文件以流的形式加载文件。

方法如下：
```java
Properties prop = new Properties();
//获取文件绝对路径
String filePath = "/coding/java/src/resources/custom.properties";
//加载配置文件
InputStream in = new FileInputStream(new File(filePath));
//读取配置文件
prop.load(new InputStreamReader(in, "UTF-8"));
System.out.println("userName："+prop.getProperty("userName"));
```
输出结果：
```
userName：李三
```
#### 3.2、通过当前类加载器的getResourceAsStream方法获取
这类方法加载文件，也是调用 Properties 的 load 方法，不同的是，通过类加载器来获取文件路径，如果当前文件是在`src/resources`目录下，那么直接传入文件名就可以了。

方法如下：
```java
Properties prop = new Properties();
//加载配置文件
InputStream in = TestProperties.class.getClassLoader().getResourceAsStream("custom.properties");
//读取配置文件
prop.load(new InputStreamReader(in, "UTF-8"));
System.out.println("userName："+prop.getProperty("userName"));
```
输出结果：
```
userName：李三
```
#### 3.3、使用ClassLoader类的getSystemResourceAsStream方法获取
和上面类似，也是通过类加载器来获取文件流，方法如下：
```java
Properties prop = new Properties();
//加载配置文件
InputStream in = ClassLoader.getSystemResourceAsStream("custom.properties");
//读取配置文件
prop.load(new InputStreamReader(in, "UTF-8"));
System.out.println("userName："+prop.getProperty("userName"));
```
输出结果：
```
userName：李三
```
#### 3.4、使用 ResourceBundle  类加载文件
ResourceBundle 类加载文件，与 Properties 有所不同，ResourceBundle 获取 properties 文件不需要加`.properties`后缀名，只需要文件名即可。

ResourceBundle 是按照iso8859编码格式来读取原属性文件，如果是读取中文内容，需要进行转码处理。

方法如下：
```java
//加载custom配置文件，不需要加`.properties`后缀名
ResourceBundle resource = ResourceBundle.getBundle("custom");
//转码处理，解决读取中文内容乱码问题
String value = new String(resource.getString("userName").getBytes("ISO-8859-1"),"UTF-8");
System.out.println("userName："+value);
```
输出结果：
```java
userName：李三
```
### 04、总结
从源码上可以看出，Properties 继承自 Hashtable，大部分方法都复用于 Hashtable，**与 Hashtable 不同的是， Properties 中的 key 和 value 都是字符串。**

实际开发中，Properties 主要用于读取配置文件，尤其是在不同的环境下，变量值需要不一样的情况，可以通过读取配置文件来**避免**将变量值写死在 java 的枚举类中，以达到一行代码，多处运行的目的！

在读取 Properties 配置文件的时候，容易因文件路径找不到报错，可以参考 properties 文件加载的几种方式，如果网友还有新的加载方法，欢迎给我们留言！
### 05、参考
1、JDK1.7&JDK1.8 源码

2、[CSDN - java读取properties配置文件的几种方法 ](https://blog.csdn.net/mhmyqn/article/details/7683909)