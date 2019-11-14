---
layout: post  
title: 工厂模式详解
tagline: by xiaojiu
categories: 设计模式
tags: 
    - xiaojiu

---

工厂模式详解
<!--more-->

### 简单工厂

简单工厂模式（Simple Factory Pattern）是指由一个工厂对象决定创建出哪一种产品类的实例。简单工厂适用于工厂类负责创建的对象较少的场景，且客户端只需要传入工厂类的参数，对于如何创建对象的逻辑不需要关心。

接下来我们来看代码，还是以课程为例。我们定义一个课程标准 ICourse 接口：

```java
public interface ICourse {
    //录制视频
    void record();
}
```

创建一个 Java 课程的实现 JavaCourse 类：

```java
public class JavaCourse implements ICourse {
    @Override
    public void record() {
        System.out.println("录制java视频");
    }
}
```

看客户端调用代码，一般我们会这样写：

```java
public class SimpleFactoryTest {
    public static void main(String[] args) {
        ICourse course = new JavaCourse();
        course.record();
    }
}
```

看上面的代码，父类 ICourse 指向子类 JavaCourse 的引用，应用层代码需要依赖JavaCourse，如果业务扩展，我继续增加 PythonCourse 甚至更多，那么我们客户端的依赖会变得越来越臃肿。因此，我们要想办法把这种依赖减弱，把创建细节隐藏。虽然目前的代码中，我们创建对象的过程并不复杂，但从代码设计角度来讲不易于扩展。现在，我们用简单工厂模式对代码进行优化。先增加课程 PythonCourse 类：

```java
public class PythonCourse implements ICourse {
    @Override
    public void record() {
        System.out.println("录制Python视频");
    }
}
```

创建 CourseFactory 工厂类：

```java
public class CourseFactory {
    public static ICourse create(String name){
        if("java".equals(name)){
            return new JavaCourse();
        }else if("python".equals(name)){
            return new PythonCourse();
        }else {
            return null;
        }
    }
}
```

修改测试代码：

```java
public class SimpleFactoryTest {
    public static void main(String[] args) {
        //ICourse course = new JavaCourse();
        ICourse course = CourseFactory.create("java");
        course.record();
    }
}
```

现在客户端调用是不是就简单了呢，但如果我们业务继续扩展，要增加前端课程，那么工厂中的 create()就要根据产品的增加每次都要修改代码逻辑。不符合开闭原则。因此，我们对简单工厂还可以继续优化，可以采用反射技术：

```java
public class CourseFactory {
    public static ICourse create(String className){
        //这里没有引入springUtil相关的jar包,也没有自己封装,将就吧
        if(className != null && !"".equals(className)){
            try {
                return (ICourse) Class.forName(className).newInstance();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }

}
```

修改测试代码：

```java
public class SimpleFactoryTest {
    public static void main(String[] args) {
        //ICourse course = new JavaCourse();
        //ICourse course = CourseFactory.create("java");
        ICourse course = CourseFactory.create("com.cl.factory.simplefactory.JavaCourse");
        course.record();
    }
}
```

优化之后，产品不断丰富不需要修改 CourseFactory 中的代码。但是，有个问题是，方法参数是字符串，可控性有待提升，而且还需要强制转型。我们再修改一下代码：

```java
public class CourseFactory {
    public static ICourse create(Class<? extends ICourse> clazz){
        if(clazz != null){
            try {
                return clazz.newInstance();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}

```

优化测试代码：

```java
public class SimpleFactoryTest {
    public static void main(String[] args) {
        //ICourse course = new JavaCourse();
        //ICourse course = CourseFactory.create("java");
        //ICourse course = CourseFactory.create("com.cl.factory.simplefactory.JavaCourse");
        ICourse course = CourseFactory.create(JavaCourse.class);
        course.record();
    }
}
```

简单工厂到这里就结束了。这里再简单举例说下简单工厂的应用场景

```java
	//这个代码大家应该很熟悉
	Logger logger = LoggerFactory.getLogger(CacheManager.class);

	//看下它的实现
	public static Logger getLogger(String name) {
        ILoggerFactory iLoggerFactory = getILoggerFactory();
        return iLoggerFactory.getLogger(name);
    }

    public static Logger getLogger(Class<?> clazz) {
        Logger logger = getLogger(clazz.getName());
        if (DETECT_LOGGER_NAME_MISMATCH) {
            Class<?> autoComputedCallingClass = Util.getCallingClass();
            if (autoComputedCallingClass != null && nonMatchingClasses(clazz, autoComputedCallingClass)) {
                Util.report(String.format("Detected logger name mismatch. Given name: \"%s\"; computed name: \"%s\".", logger.getName(), autoComputedCallingClass.getName()));
                Util.report("See http://www.slf4j.org/codes.html#loggerNameMismatch for an explanation");
            }
        }

        return logger;
    }
```



### 工厂方法模式

工厂方法模式（Fatory Method Pattern）是指定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类，工厂方法让类的实例化推迟到子类中进行。在工厂方法模式中用户只需要关心所需产品对应的工厂，无须关心创建细节，而且加入新的产品符合开闭原则。
工厂方法模式主要解决产品扩展的问题，在简单工厂中，随着产品链的丰富，如果每个课程的创建逻辑有区别的话，工厂的职责会变得越来越多，有点像万能工厂，并不便于维护。根据单一职责原则我们将职能继续拆分，专人干专事。Java 课程由 Java 工厂创建，Python 课程由 Python 工厂创建，对工厂本身也做一个抽象。来看代码，先创建ICourseFactory 接口：

```java
public interface ICourseFactory {
    ICourse crete();
}
```

在分别创建子工厂，JavaCourseFactory 类：

```java
public class JavaCourseFactory implements ICourseFactory {
    @Override
    public ICourse crete() {
        return new JavaCourse();
    }
}
```

PythonCourseFactory 类：

```java
public class PythonCourseFactory implements ICourseFactory {
    @Override
    public ICourse crete() {
        return new PythonCourse();
    }
}
```

看测试代码：

```java
public class SimpleFactoryTest {
    public static void main(String[] args) {
        ICourseFactory courseFactory = new JavaCourseFactory();
        ICourse course = courseFactory.crete();
        course.record();

        courseFactory = new PythonCourseFactory();
        course = courseFactory.crete();
        course.record();
    }
}
```

#### 工厂方法适用于以下场景：

1. 创建对象需要大量重复的代码。
2. 客户端（应用层）不依赖于产品类实例如何被创建、实现等细节。
3. 一个类通过其子类来指定创建哪个对象。

#### 工厂方法的缺点

1. 类的个数容易过多，增加复杂度。
2. 增加了系统的抽象性和理解难度。

### 抽象工厂模式

抽象工厂模式（Abastract Factory Pattern）是指提供一个创建一系列相关或相互依赖对象的接口，无须指定他们具体的类。客户端（应用层）不依赖于产品类实例如何被创建、实现等细节，强调的是一系列相关的产品对象（属于同一产品族）一起使用创建对象需要大量重复的代码。需要提供一个产品类的库，所有的产品以同样的接口出现，从而使客户端不依赖于具体实现。

接下来我们来看一个具体的业务场景而且用代码来实现。还是以课程为例，每个课程不仅要提供课程的录播视频，而且还要提供课堂笔记。相当于现在的业务变更为同一个课程不单纯是一个课程信息，要同时包含录播视频、课堂笔记甚至还要提供源码才能构成一个完整的课程。在产品等级中增加两个产品 IVideo 录播视频和 INote 课堂记。

IVideo 接口：

```java
public interface IVideo {
    void  record();
}
```

INote 接口：

```java
public interface INote {
    void edit();
}
```

接下来，创建 Java 产品族，Java 视频 JavaVideo 类:

```java
public class JavaVideo implements IVideo {
    @Override
    public void record() {
        System.out.println("java视频");
    }
}
```

 Java 笔记 JavaNote 类：

```java
public class JavaNote implements INote {
    @Override
    public void edit() {
        System.out.println("java笔记");
    }
}
```

然后创建一个抽象工厂 CourseFactory 类(最好新建一个包,为了和工厂方法区别开)：

```java
public interface CourseFactory {
    INote createNote();
    IVideo createVideo();
}
```

创建 Java 产品族的具体工厂 JavaCourseFactory:

```java
public class JavaCourseFactory implements CourseFactory {

    @Override
    public INote createNote() {
        return new JavaNote();
    }
    
    @Override
    public IVideo createVideo() {
        return new JavaVideo();
    }
}
```

然后创建 Python 产品，Python 视频 PythonVideo 类：

```java
public class PythonVideo implements IVideo {

    @Override
    public void record() {
        System.out.println("python视频");
    }
}
```

扩展产品等级 Python 课堂笔记 PythonNote 类：

```java
public class PythonNote implements INote {

    @Override
    public void edit() {
        System.out.println("python笔记");
    }
}
```

创建 Python 产品族的具体工厂 PythonCourseFactory:

```java
public class PythonCourseFactory implements CourseFactory {

    @Override
    public INote createNote() {
        return new PythonNote();
    }

    @Override
    public IVideo createVideo() {
        return new PythonVideo();
    }
}
```

下面是测试代码:

```java
public class AbstractFactoryTest {

    public static void main(String[] args) {
        JavaCourseFactory javaCourseFactory = new JavaCourseFactory();
        javaCourseFactory.createNote().edit();
        javaCourseFactory.createVideo().record();
        
        PythonCourseFactory pythonCourseFactory = new PythonCourseFactory();
        pythonCourseFactory.createNote().edit();
        pythonCourseFactory.createVideo().record();
    }
}
```

上面的代码完整地描述了两个产品族 Java 课程和 Python 课程，也描述了两个产品等级视频和笔记。抽象工厂非常完美清晰地描述这样一层复杂的关系。但是，不知道大家有没有发现，如果我们再继续扩展产品等级，将源码 Source 也加入到课程中，那么我们的代码从抽象工厂，到具体工厂要全部调整，很显然不符合开闭原则。因此抽象工厂也是有缺点的：

1. 规定了所有可能被创建的产品集合，产品族中扩展新的产品困难，需要修改抽象工厂的接口。
2. 增加了系统的抽象性和理解难度。
