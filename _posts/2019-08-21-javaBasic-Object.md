---
layout: post  
title: Object 中有哪些常用方法？
tagline: by 比拉河畔
categories: Java  
tag: 
    - Java

---

「Object 中有哪些常用方法？」这是个基础的问题，面试了中问了很多人，都卡壳了？！今天一起看看。

<!--more-->


## 要说的话

- 为啥问这个问题？
- Object 类中的方法概览
- 依次认识下这些方法
- 结语


## 为啥问这个问题？

通常对于1-3年经验的同学。在面试中考察基础知识的时候，都习惯问一个基础问题--"在Java语言中的，Object是所有类的一个超类，可以讲讲您对它的了解吗？"。

大多数同学的第一反应是，”停顿3-5秒“，然后复述”Object是所有类的一个顶级父类，嗯...“，有点卡住了..

赶紧提示引导”可以讲讲它有哪些常用的方法，比如它有toString()方法，您可以讲讲它的其它常用方法。“

引导是很有必要的，面试是为了考察候选人对某一个知识点的理解情况，并不是为了为难面试同学，想想自己参加面试时也是这样，因为有点紧张，某个知识点可能一下子想不起来。遇到卡壳的，一定会引导提示下面试者。

多数同学，能说出两三个方法，能把所有方法都说出的真的很少。从这个问题可以引申考察很多基础的知识。



## Object 类中的方法概览

![](http://www.justdojava.com/assets//images/2019/java/image_bilahepan/20190821/Object-Overview.png)


总体来看有12个方法，9个public修饰的方法；2个protected修饰的方法，1个private修饰的方法。

其中notify() 和 notifyAll() 作用类似，可以看做一类方法。wait 相关的有3个重载方法。 如果说常用的方法，能说出如下7个方法，就很全面了。

- getClass
- toString
- equals
- hashCode
- notify相关的方法
- wait相关的方法
- clone
//以下两个是我们工作中几乎不会操作的
- finalize
- registerNatives




## 依次认识下这些方法

### （1）getClass

```java

public final native Class<?> getClass();

```

作用是：告诉大家我是谁，返回的是Class<?>对象。

用”人话“来简单介绍：getclass()用来获取这个类已经被实例化了的对象的该类的引用(真的被这个绕晕了)，这个引用指向的是Class类的对象。我们无法在程序中手动创建一个Class对象（可以看到Class的构造函数为private修饰)，
只能由 Java 虚拟机自动创建 Class 对象。

JVM为每个类管理唯一的Class对象，因此我们可以用双等号操作符来比较对象。工作中，在反射里经常用到它，来做各种骚操作。细节就不在此展开了。

可以从这里可以扩展到和面试同学聊”反射“的一些话题。

Class 构造方法的定义如下：

```java

/*
 * Private constructor. Only the Java Virtual Machine creates Class objects.
 * This constructor is not used and prevents the default constructor being
 * generated.
 */
private Class(ClassLoader loader) {
    // Initialize final field for classLoader.  The initialization value of non-null
    // prevents future JIT optimizations from assuming this final field is null.
    classLoader = loader;
}

```


实例：验证“JVM为每个类管理唯一的Class对象”：


```java

public class ClassDemo {
    public static UserDTO user1 = new UserDTO("1");
    private static UserDTO user2 = new UserDTO("2");

    public static void main(String[] args) {
        System.out.println(user1.getClass().equals(user2.getClass()));
        //结果是 true
    }

    public static class UserDTO implements Serializable {
        private String userId;
        public UserDTO(String userId) {this.userId = userId; }

        public String getUserId() {return userId; }
        public void setUserId(String userId) {this.userId = userId; }

        @Override
        public String toString() {
            return "UserDTO{" +
                    "userId='" + userId + '\'' +
                    '}';
        }
    }
}

```



### （2）toString

toString 方法的定义及描述如下：

![](http://www.justdojava.com/assets//images/2019/java/image_bilahepan/20190821/Object-toString.png)


作用是：返回对象的字符串表示形式，让人能简洁明了的知道当前对象长啥样。

这里有两点注意：
- 源码里建议我们在所有的子类中，对这个方法重写。 
- 默认返回的是”类名“+”@“+hashCode值的无符号数值形式表示。

可以从这里可以扩展到和面试同学聊”hashCode“、“序列化反序列化”的一些话题。




### （3）equals


equals 方法的定义及描述如下：

![](http://www.justdojava.com/assets//images/2019/java/image_bilahepan/20190821/Object-equals.png)


作用是：判断两个对象是否相等。

5个原则：
- (1)自反性:对于任何非空引用x,x.equals(x)应该返回true;
- (2)对称性:对于任何引用x,和y,当且仅当,y.equals(x)返回true,x.equals(y)也应该返回true;
- (3)传递性:对于任何引用x,y,z,如果x.equals(y)返回true,y.equals(z)返回true,那么x.equals(z)也应该返回true;
- (4)一致性:如果x,y引用的对象没有发生变化,反复调用x.equals(y)应该返回同样的结果;
- (5)对于任意非空引用x,x.equals(null)返回false。

1个约定：
重写equals方法的时候，也要重写hashCode方法，在后面的hashCode方法的介绍为什么要有这个约定。

1个注意：
equals的默认内部实现是 "=="来判等的。

可以从这里可以扩展到和面试同学聊”equals 和 ’==‘ 异同”的一些话题。



### （4）hashCode


hashCode 方法的定义及描述如下：

![](http://www.justdojava.com/assets//images/2019/java/image_bilahepan/20190821/Object-hashCode.png)

作用是：返回对象的哈希码值，哈希表的数据结构需要依赖哈希值，如{@link java.util.HashMap}哈希表结构。

3个约定：
- (1)同一个对象，多次获取其哈希码值不变，都相同。
- (2)如果A.equals(B)成立,那么对象A和对象B的哈希码值相同。
- (3)如果A.equals(B)不成立,那么对象A和对象B的哈希码值可能相同，也可能不相同。建议重写hashCode时，最好不同的对象A、B，它们的哈希码值也不相同，这有利于减少哈希冲突，在哈希表的结构中存储时可以提高性能。

1个常识：
默认情况下，Object类的实例对象的hashCode方法返回的值是该对象的内存存储地址转换得到的一个整数值。

可以从这里可以扩展到和面试同学聊”hash冲突、hash一致性实际应用”的一些话题。



### （5）notify/notifyAll

作用分别是：notify 唤醒此对象上等待的单个线程；notifyAll 唤醒此对象上等待的多个线程。

每个对象都拥有一个锁池(EntrySet)和一个等待池(WaitSet)。

锁池：
假如已经有线程A获取到了锁，这时候又有线程B需要获取这把锁(比如需要调用synchronized修饰的方法或者需要执行synchronized修饰的代码块)，由于该锁已经被占用，所以线程B只能等待这把锁，这时候线程B将会进入这把锁的锁池。

等待池：
假设线程A获取到锁之后，由于一些条件的不满足(例如生产者消费者模式中生产者获取到锁，然后判断队列为满)，此时需要调用对象锁的wait方法，那么线程A将放弃这把锁，并进入这把锁的等待池。

如果有其他线程调用了锁的notify方法，则会根据一定的算法从等待池中选取一个线程，将此线程放入锁池。
如果有其他线程调用了锁的notifyAll方法，则会将等待池中所有线程全部放入锁池，并争抢锁。

锁池与等待池的区别：
等待池中的线程不能获取锁，而是需要被唤醒进入锁池，才有获取到锁的机会。



### （6）wait

wait()的作用是让当前线程进入“等待(阻塞)状态”，同时wait()也会让当前线程释放它所持有的锁，“直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法”，当前线程才可能被唤醒(进入“就绪状态”)

wait(long timeout)的作用是让当前线程进入“等待(阻塞)状态”，同时wait(long timeout)也会让当前线程释放它所持有的锁。“直到其他线程调用此对象的notify()方法或 notifyAll() 方法，或者超过指定的时间量”，
当前线程才可能被唤醒(进入“就绪状态”)。

wait(long timeout, int nanos)方法同wait(long timeout),nanos 参数指定的是一个纳秒单位，取值范围为[0,999999]。

可以从这里可以扩展到和面试同学聊”线程相关”的一些话题。




### （7）clone

作用是：希望copy是一个新对象，它的初始状态与origin相同，但之后它们各自会有自己不同的状态，这种情况下使用clone方法。 这里涉及到”浅拷贝“，”深拷贝“。

浅拷贝：
clone方法是Object的一个protected方法，所以说你的代码不能直接调用这个方法（所有类都是Object的子类，其实例对象不会和Object在同一个包）。如果对象中的所有数据域都是数值或是其他基本数据类型，拷贝这些域没有问题；
如果包含对象引用，拷贝域就会得到对应对象的另一个引用。对象引用这一部分是共享的，其它引用对其修改会引起改变。这就是”浅拷贝“存在的问题。

深拷贝：如果能确保浅拷贝的情况下，共享的那一部分数据（引用的对象）始终是不可变的，如共享的是String对象（不可变对象），LocalDate对象（不可变对象），那么这种情况下就是深拷贝。

建立“深拷贝”步骤：
- 判定默认clone方法不能满足要求； 
- 类重写clone方法时要实现Cloneable接口（Cloneable只是一个标记作用，在类型查询时可用instanceof，如果这个对象的类没有实现Cloneable接口，就会抛出CloneNotSupportedException。所有的数组都有一个public的clone方法。）； 
- 重新定义clone方法，并指定修饰符为public。



### （8）finalize

这个方法在编程中一般是用不到的。

finalize方法是与Java编程中的垃圾回收器有关系。它最主要的用途是回收特殊渠道申请的内存。Java程序有垃圾回收器，所以一般情况下内存问题不用程序员来编程操作。但有一种JNI(Java Native Interface)调用non-Java程序（C或C++），
finalize()的工作就是回收这部分的内存。



### （9）registerNatives
本地方法是由其他语言（比如C，C++，或者汇编）编写的，编译成和处理器相关的机器代码。本地方法保存在动态连接库中，格式是各个平台专有的。Java方法是平台无关的，但本地方法却不是。运行中的Java程序调用本地方法时，
虚拟机需要装载包含这个本地方法的动态库。

本地方法的实现是由其他语言编写并保存在动态连接库中，因而在Java类中不需要方法实现。registerNatives本质上就是一个本地方法，但这又是一个有别于一般本地方法的本地方法，从方法名我们可以猜测该方法应该是用来注册本地方法的。
具体应用到时，需要先定义了registerNatives()方法，然后当该类被加载的时候，调用该方法完成对该类中本地方法的注册。


## 结语

Object 是顶级父类，所有的类都继承它。在这篇文章里初步了解了每个方法的情况，在后续的的文章中会继续探讨工作中是如何实践的。






