---
layout: post
categories: 设计模式
title: 阿粉带你学习设计模式之原型（Prototype）模式
tags:
  - 子悠
---

大家好，我是鸭血粉丝，今天阿粉带大家学习一下设计模式之原型模式，话不多说，我们往下看。

<!--more-->

今天要跟大家分享的是创建型模式之原型模式，作为设计模式中创建型模式中的一员，很明显原型模式是用来创建对象的，那么到底什么是原型模式，以及原型模式到底是怎么使用的，有哪些使用场景呢？且听阿粉我慢慢道来。

### 原型模式
首先我们看下原型模式的定义，维基百科上这样解释的：**原型模式是创建型模式的一种，其特点在于通过“复制”一个已经存在的实例来返回新的实例，而不是新建实例。被复制的实例就是我们所称的“原型”，这个原型是可定制的。**

阿粉划重点了！

关键词：**原型实例，复制，新实例**

通俗点说就是我们在某个需要创建实例的地方不是通过 `new` 关键字来创建某个类，而是通过复制某个类已有的实例来创建一个新的实例。比如有个猴子 `Monkey` 类，对应有个孙悟空实例我们叫猴哥一号，在打不过某个妖怪 `Monster` 的时候拔一根猴毛吹一下，克隆一个新的 `Monkey`，我们叫做猴哥二号，这时有两个猴哥一起对付妖怪。这里我们原始的猴哥一号就是原型对象，猴哥二号就是克隆对象。

这里我们思考这几个点
1. 是不是每个类都可以克隆？为什么 `Monkey` 类可以克隆，妖怪 `Monster` 为什么不克隆呢？
2. 猴哥二号是不是具备所有猴哥一号的本领呢？要是猴哥二号被妖怪打伤了，猴哥一号会不会受伤呢？

这里阿粉先留下这两个问题，我们继续往下看。

### 原型模式使用场景
上面提到了原型模式使用场景是在需要创建新的实例的时候不通过 `new` 而是通过”复制“ 的方式来创建。那么我们会想为什么不通过 new 来创建呢？new 起来多方便啊，搞什么复制那么麻烦，在这里阿粉我摸了摸所剩不多的秀发。

![touding](/Users/zhuxiang/Desktop/touding.jpeg)

那是因为主要有下面两点的原因：

1. 无法根据类名来 new 对象。我们知道在 new 一个类的时候是需要知道类名的，如果不知道类名叫什么，我们自然无法通过 new 来创建实例；
2. 需要大量创建相同对象的时候。有时候我们可能在一个循环中创建对象，而且这个对象创建的复杂度很高，那这种时候如果我们每次都是用 new 来创建那么性能会比较低，当然现在 JVM 已经优化的很好了，但是对于有些对性能要求特别高的场景，还是要注意的。


### UML 图解
在整个原型模式中我们先了解下大概有几个角色的存在，了解各自的关系，才能更好的方便我们的理解。还是以上面猴哥的例子，首先有个 Monkey 类，而且这个 Monkey 类要支持能克隆。

![原型类](/Users/zhuxiang/Desktop/1.png)

![1](/Users/zhuxiang/Desktop/2.png)


### 代码演示
1. 第一步我们先创建Monkey 的原型类，原型类必须实现 Java 的 `Cloneable` 接口，然后重写父类 `Object` 的 `clone` 方法，如下；

```java

package com.silence.arts.pattern.creator.prototype;

import lombok.Data;

import java.util.ArrayList;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Desc：</b>Monkey 原型类<br>
 */
@Data
public class MonkeyPrototype implements Cloneable {

    private Integer age;
    private String name;
    private ArrayList<String> things;
    private ExtraSkill extraSkill;
    private MonkeyPrototype monkeyPrototype = null;

    @Override
    public MonkeyPrototype clone() {
        try {
            monkeyPrototype = (MonkeyPrototype) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return monkeyPrototype;
    }
}

```

2. 我们根据 `Monkey` 原型，来创建真正的 `Mokey` 对象，并设置一个输出方法，如下：
```java

package com.silence.arts.pattern.creator.prototype;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Desc：</b>无<br>
 */
public class Monkey extends MonkeyPrototype {

    /**
     * 输出内容
     *
     */
    public void message() {
        System.out.println(this.getName() + "-" + this.getAge() + "-" + this.getThings() + "- 金箍棒长度：" + this.getExtraSkill().getWidth());
    }
}


```

3. 由于猴哥还有一些其他本领，我们这里在创建一个其他技能的类
```java
package com.silence.arts.pattern.creator.prototype;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Desc：</b>无<br>
 */
public class ExtraSkill {

    /**
     * 放大金箍棒的宽度
     */
    private int width;
    /**
     * 放大金箍棒的高度
     */
    private int height;

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }
}


```

4. 创建一个客户端类来使用`Monkey`
```java
package com.silence.arts.pattern.creator.prototype;

import java.util.ArrayList;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Desc：</b>无<br>
 */
public class MonkeyClient {
    public static void main(String[] args) {
        //创建猴哥一号
        Monkey monkey = new Monkey();
        monkey.setName("猴哥一号");
        monkey.setAge(18);
        ArrayList<String> list = new ArrayList<>();
        list.add("七十二变化");
        list.add("火眼睛就");
        monkey.setThings(list);
        ExtraSkill m = new ExtraSkill();
        m.setWidth(1920);
        m.setHeight(1080);
        monkey.setExtraSkill(m);

        //克隆五个猴哥
        System.out.println("--------------------------克隆猴哥----------------------------");
        for (int i = 0; i < 5; i++) {
            Monkey cloneMonkey = (Monkey) monkey.clone();
            cloneMonkey.setAge(cloneMonkey.getAge() + i);
            cloneMonkey.setName(cloneMonkey.getName() + i);
            ArrayList<String> innerThing = cloneMonkey.getThings();
            innerThing.add("特殊技能 " + i);
            cloneMonkey.setThings(innerThing);
            cloneMonkey.getExtraSkill().setWidth(cloneMonkey.getExtraSkill().getWidth() + i * 100);
            cloneMonkey.getExtraSkill().setHeight(cloneMonkey.getExtraSkill().getHeight() + i * 100);
            System.out.println(cloneMonkey.getExtraSkill());
            cloneMonkey.message();
            System.out.println();
        }
        System.out.println("--------------------------猴哥一号----------------------------");
        System.out.println(monkey.getExtraSkill());
        monkey.message();
    }
}


```

5. 运行 `MonkeyClient`，得到如下输出：
```
--------------------------克隆猴哥----------------------------
com.silence.arts.pattern.creator.prototype.ExtraSkill@2503dbd3
猴哥一号0-18-[七十二变化, 火眼睛就, 特殊技能 0]- 金箍棒长度：1920

com.silence.arts.pattern.creator.prototype.ExtraSkill@2503dbd3
猴哥一号1-19-[七十二变化, 火眼睛就, 特殊技能 0, 特殊技能 1]- 金箍棒长度：2020

com.silence.arts.pattern.creator.prototype.ExtraSkill@2503dbd3
猴哥一号2-20-[七十二变化, 火眼睛就, 特殊技能 0, 特殊技能 1, 特殊技能 2]- 金箍棒长度：2220

com.silence.arts.pattern.creator.prototype.ExtraSkill@2503dbd3
猴哥一号3-21-[七十二变化, 火眼睛就, 特殊技能 0, 特殊技能 1, 特殊技能 2, 特殊技能 3]- 金箍棒长度：2520

com.silence.arts.pattern.creator.prototype.ExtraSkill@2503dbd3
猴哥一号4-22-[七十二变化, 火眼睛就, 特殊技能 0, 特殊技能 1, 特殊技能 2, 特殊技能 3, 特殊技能 4]- 金箍棒长度：2920

--------------------------猴哥一号----------------------------
com.silence.arts.pattern.creator.prototype.ExtraSkill@2503dbd3
猴哥一号-18-[七十二变化, 火眼睛就, 特殊技能 0, 特殊技能 1, 特殊技能 2, 特殊技能 3, 特殊技能 4]- 金箍棒长度：2920

Process finished with exit code 0

```
6. 运行结果分析
  上面程序中我们首先通过复杂的流程创建的猴哥一号（这里虽然很简单，但是我们假设很复杂。。。），并且设置了猴哥一号的名称和一些技能，然后通过克隆的方式克隆了五个猴哥，并且我们进行了重新命名，以及技能的调整，最后输出所有猴哥的名称和技能点。

  可以看到我们完美的克隆了五个猴哥，但是仔细观察我们会发现一个问题，在更改了克隆出来的猴哥的名字和年龄的时候对原始的猴哥一号没有影响，但是在更改克隆猴哥的技能的时候，原始猴哥的技能也变掉了！

  这里我们如果认为这些技能是原始猴哥的，那么就可以说克隆出来的猴哥并没有克隆出属于自己的技能。这里就回答了我们上面的问题，克隆猴哥不是具备所有猴哥一号的本领，至于为什么，那是因为在 Java 中有深拷贝和浅拷贝，关于这个，这里就不展开了，留给大家自己思考，自己去尝试写一些浅拷贝和深拷贝的代码。

  那么我们再看看上面第一个提问，为什么只有 `Monkey` 类能克隆呢？`Monster` 为什么不克隆呢？到这里我想大家都知道了，那是因为 `Monster` 类没有实现 Java 中的 `Cloneable` 接口，那自然是无法克隆的。

  解答了上面遗留的两个问题，阿粉的内心激动不已啊~


### 原型模式的应用
相信大家在使用 `Spring` 的时候都有用过 `@scope("prototype")`，由于 `Spring` 中默认 Bean 是单例模式的，如果我们需要某个 Bean 在每次请求的时候都创建一个新的，而不是只有一个，那就需要将这个 Bean 的模式改成原型模式，这样在每次使用的使用就会创建一个新的，使用结束后就会销毁。
另外原型模式很少会单独使用，很多时候都是结合工厂模式一起使用，所以很多时候我们看到的原型模式并不是本文的案例这么清晰，大家要擦亮眼睛。
在 Spring 源码 `AbstractBeanFactory` 类中的 `doGetBean()` 方法中会根据 Bean 的类型进行注册。
![1](/Users/zhuxiang/Desktop/3.png)


### 总结
今天阿粉先从原型模式的定义，再到给个具体的案例来带大家认识了解了原型模式，通过这篇文章希望大家对原型模式有了更深的理解。学习设计模式的学习是个漫长的过程，希望大家坚持初心，我们共同学习进步，欢迎大家一起讨论进步。

最后欢迎大家到我们《Java 极客技术》的知识星球中一起学习，更有其他设计模式相关内容分享和 SpringBoot 系列教学视频免费发放，名额有限，先到先得哦。