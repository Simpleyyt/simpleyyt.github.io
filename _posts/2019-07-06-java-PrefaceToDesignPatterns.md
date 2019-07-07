---
layout: post
title: 我来先为我们的设计模式铺铺路(面向对象设计的原则一二)
tagline: by 懿
categories: java
tag: 
    - java
---

我们的知识星球马上就要开始更新设计模式了，在更新设计模式之前，我们是不是需要做一些准备呢？否则设计模式中一些遵循的原则大家会一头雾水，所以我今天来给大家说一些面向对象的七种原则，有人说是6种有人说是7种，我个人认为是7种，我就按照7种来说，今天我就介绍2种，下一篇文章将会继续介绍剩下的五种原则，这些原则也会在设计模式中出现，各位技术人，欢迎大家的踊跃参加呦。
<!--more-->

## 前言

在面向对象的软件设计中，只有尽量降低各个模块之间的耦合度，才能提高代码的复用率，系统的可维护性、可扩展性才能提高。面向对象的软件设计中，有23种经典的设计模式，是一套前人代码设计经验的总结，如果把设计模式比作武功招式，那么设计原则就好比是内功心法。常用的设计原则有七个，下文将具体介绍。

## 设计原则简介

- 单一职责原则：专注降低类的复杂度，实现类要职责单一；

- 开放关闭原则：所有面向对象原则的核心，设计要对扩展开发，对修改关闭；

- 里式替换原则：实现开放关闭原则的重要方式之一，设计不要破坏继承关系；

- 依赖倒置原则：系统抽象化的具体实现，要求面向接口编程，是面向对象设计的主要实现机制之一；

- 接口隔离原则：要求接口的方法尽量少，接口尽量细化；

- 迪米特法则：降低系统的耦合度，使一个模块的修改尽量少的影响其他模块，扩展会相对容易；

- 组合复用原则：在软件设计中，尽量使用组合/聚合而不是继承达到代码复用的目的。

这些设计原则并不说我们一定要遵循他们来进行设计，而是根据我们的实际情况去怎么去选择使用他们，来让我们的程序做的更加的完善。

## 单一职责原则

`解释`：

就一个类而言，应该仅有一个引起它变化的原因，通俗的说，就是一个类只负责一项职责。

此原则的核心就是解耦和增强内聚性

那么为什么要使用单一职责原则：

如果一个类承担的职责过多，就等于把这些职责耦合在一起，一个职责的变化可能会削弱或者抑制这个类完成其他职责的能力。这种耦合会导致脆弱的设计。

这也是他的优点，我们总结一下

`优点`：

（1）降低类的复杂度；

（2）提高类的可读性，提高系统的可维护性；

（3）降低变更引起的风险（降低对其他功能的影响）。

我们来举一些简单的例子来说明一下这个单一职责原则

`实例`：

```
    //我们用动物生活来做测试
    class Animal{
        public void breathe(String animal){
            System.out.println(animal+"生活在陆地上");
        }
    }
    public class Client{
        public static void main(String[] args){
            Animal animal = new Animal();
            animal.breathe("羊");
            animal.breathe("牛");
            animal.breathe("猪");
        }
    }
   
 运行结果
 羊生活在陆地上
 牛生活在陆地上
 猪生活在陆地上

```

但是问题来了，动物并不是都生活在陆地上的，鱼就是生活在水中的，修改时如果遵循单一职责原则，需要将Animal类细分为陆生动物类Terrestrial，水生动物Aquatic，代码如下：

```
class Terrestrial{
    public void breathe(String animal){
        System.out.println(animal+"生活在陆地上");
    }
}
class Aquatic{
    public void breathe(String animal){
        System.out.println(animal+"生活在水里");
    }
}

    public class Client{
        public static void main(String[] args){
            Terrestrial terrestrial = new Terrestrial();
            terrestrial.breathe("羊");
            terrestrial.breathe("牛");
            terrestrial.breathe("猪");
    
            Aquatic aquatic = new Aquatic();
            aquatic.breathe("鱼");
        }

运行结果：

羊生活在陆地上
牛生活在陆地上
猪生活在陆地上
鱼生活在水里

```

但是问题来了如果这样修改花销是很大的，除了将原来的类分解之外，还需要修改客户端。而直接修改类Animal来达成目的虽然违背了单一职责原则，但花销却小的多，代码如下：

```

class Animal{
    public void breathe(String animal){
        if("鱼".equals(animal)){
            System.out.println(animal+"生活在水中");
        }else{
            System.out.println(animal+"生活在陆地上");
        }
    }
}

public class Client{
    public static void main(String[] args){
        Animal animal = new Animal();
        animal.breathe("羊");
        animal.breathe("牛");
        animal.breathe("猪");
        animal.breathe("鱼");
    }

```
可以看到，这种修改方式要简单的多。但是却存在着隐患：有一天需要将鱼分为生活在淡水中的鱼和生活在海水的鱼，则又需要修改Animal类的breathe方法，而对原有代码的修改会对调用“猪”“牛”“羊”等相关功能带来风险，也许某一天你会发现程序运行的结果变为“牛生活在水中”了。

这种修改方式直接在代码级别上违背了单一职责原则，虽然修改起来最简单，但隐患却是最大的。还有一种修改方式：

```

class Animal{
    public void breathe(String animal){
        System.out.println(animal+"生活在陆地上");
    }

    public void breathe2(String animal){
        System.out.println(animal+"生活在水中");
    }
}

public class Client{
    public static void main(String[] args){
        Animal animal = new Animal();
        animal.breathe("牛");
        animal.breathe("羊");
        animal.breathe("猪");
        animal.breathe2("鱼");
    }
}
```

可以看到，这种修改方式没有改动原来的方法，而是在类中新加了一个方法，这样虽然也违背了单一职责原则，但在方法级别上却是符合单一职责原则的，因为它并没有动原来方法的代码。

这三种方式各有优缺点，那么在实际编程中，采用哪一中呢？其实这真的比较难说，需要根据实际情况来确定。我的原则是：只有逻辑足够简单，才可以在代码级别上违反单一职责原则；只有类中方法数量足够少，才可以在方法级别上违反单一职责原则；

例如本文所举的这个例子，它太简单了，它只有一个方法，所以，无论是在代码级别上违反单一职责原则，还是在方法级别上违反，都不会造成太大的影响。
实际应用中的类都要复杂的多，一旦发生职责扩散而需要修改类时，除非这个类本身非常简单，否则还是遵循单一职责原则的好。

以上就是我所说的单一职责原则了，很多书中介绍的说它并不属于面向对象设计原则中的一种，但是我认为它是，所以我就把他解释出来了。

## 开放关闭原则

开放关闭原则又称为开放封闭原则。

`定义`：

一个软件实体应该对扩展开放，对修改关闭，这个原则也是说，在设计一个模块的时候，应当使这个模块可以在不被修改的前提下被扩展，换句话说，应当可以在不必修改源码的情况下改变这个模块的行为。

这句话其实刚开始看上去是有些矛盾的，接下来在我后边文章解释里面，我会把他解释清楚一点。

我们先用个比较好玩的例子来说一下。

西游记大家都看过，玉帝招安孙悟空的时候的桥段，大家还有没有印象？

当年大脑天宫的时候美猴王对玉皇大帝做了个挑战，美猴王说：皇帝轮流做，明年到我家，只叫他搬出去，将天宫让给我，对于这个挑战，太白金星给玉皇大帝提了个意见，我们把它招上来，给他个小官做，他不就不闹事了？

换一句话说，不用兴师动众的，不破坏天庭的规矩这就是闭，但是收他为官，便是开，招安的方法便是符合开闭原则的，给他个‘弼马温’的官，便可以让这个系统正常不受威胁，是吧，何乐而不为？
我给大家画个图。

![](http://www.justdojava.com//assets/images/2019/java/image_yi/07_06/1.jpg)

招安的方法的关键就是不允许更改现有的天庭秩序，但是允许将妖猴纳入现有的秩序中，从而扩展了这个秩序，用面向对象的语言来说：不允许更改的是系统的抽象层，而允许扩展的是系统的实现层。

我们写一些简单的代码来进行一个完整的展示来进行一下理解。

`实例`：

```
    //书店卖书
    interface Books{
        //书籍名称
        public Sting getName();
        //书籍价格
        public int getPrice();
        //书籍作者
        public String getAuthor();
    }
    
public class NovelBook implements Books {
    private String name;

    private int price;

    private String author;

    public NovelBook(String name, int price, String author) {
        this.name = name;
        this.price = price;
        this.author = author;
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public int getPrice() {
        return price;
    }

    @Override
    public String getAuthor() {
        return author;
    }
}


```

以上的代码是数据的实现类和书籍的类别；

下面我们将要开始对书籍进行一个售卖活动：
```
public class BookStore {
    private final static ArrayList<Books> sBookList = new ArrayList<Books>();

    static {
        sBookList.add(new NovelBook("天龙八部", 4400, "金庸"));
        sBookList.add(new NovelBook("射雕英雄传", 7600, "金庸"));
        sBookList.add(new NovelBook("钢铁是怎么炼成的", 7500, "保尔·柯查金"));
        sBookList.add(new NovelBook("红楼梦", 3300, "曹雪芹"));
    }

    public static void main(String[] args) throws IOException {
        NumberFormat format = NumberFormat.getCurrencyInstance();
        format.setMaximumFractionDigits(2);
       System.out.println("----书店卖出去的书籍记录如下---");
        for (Books book : sBookList) {
            System.out.println("书籍名称:" + book.getName()
                    + "\t书籍作者:" + book.getAuthor()
                    + "\t书籍价格:" + format.format(book.getPrice() / 100.00) + "元");
        }
    }
}
```
运行结果如下：

```
D:\develop\JDK8\jdk1.8.0_181\bin\java.exe "-javaagent:D:\develop\IDEA\IntelliJ IDEA 2018.2.4\lib\idea_rt.jar=62787:D:\develop\IDEA\IntelliJ IDEA 2018.2.4\bin" -Dfile.encoding=UTF-8 -classpath D:\develop\JDK8\jdk1.8.0_181\jre\lib\charsets.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\deploy.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\access-bridge-64.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\cldrdata.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\dnsns.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\jaccess.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\jfxrt.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\localedata.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\nashorn.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\sunec.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\sunjce_provider.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\sunmscapi.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\sunpkcs11.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\ext\zipfs.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\javaws.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\jce.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\jfr.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\jfxswt.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\jsse.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\management-agent.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\plugin.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\resources.jar;D:\develop\JDK8\jdk1.8.0_181\jre\lib\rt.jar;D:\develop\IDEA_Workspace\CloudCode\out\production\PattemMoudle com.yldyyn.test.BookStore
----书店卖出去的书籍记录如下---
书籍名称:天龙八部	书籍作者:金庸	书籍价格:￥44.00元
书籍名称:射雕英雄传	书籍作者:金庸	书籍价格:￥76.00元
书籍名称:钢铁是怎么炼成的	书籍作者:保尔·柯查金	书籍价格:￥75.00元
书籍名称:红楼梦	书籍作者:曹雪芹	书籍价格:￥33.00元

Process finished with exit code 0

```

但是如果说现在书店卖书的时候要求打折出售，40以上的我们要7折售卖，40一下的我们打8折。

方法有三种，第一个办法：修改接口。在Books上新增加一个方法getOnSalePrice()，专门进行打折，所有实现类实现这个方法。 但是这样修改的后果就是实现类NovelBook要修改,BookStore中的main方法也修改，同时Books作为接口应该是稳定且可靠的，不应该经常发生变化，否则接口做为契约的作用就失去了效能，其他不想打折的书籍也会因为实现了书籍的接口必须打折，因此该方案被否定。

第二个办法：修改实现类。修改NovelBook 类中的方法，直接在getPrice()中实现打折处理，这个应该是大家在项目中经常使用的就是这样办法，通过class文件替换的方式可以完成部分业务（或是缺陷修复）变化，但是该方法还是有缺陷的，例如采购书籍人员也是要看价格的，由于该方法已经实现了打折处理价格，因此采购人员看到的也是打折后的价格，这就产生了信息的蒙蔽效果，导致信息不对称而出现决策失误的情况。该方案也不是一个最优的方案。

第三个办法，通过扩展实现变化增加一个子类 OffNovelBook，覆写getPrice方法，高层次的模块（也就是static静态模块区）通过OffNovelBook类产生新的对象，完成对业务变化开发任务。好办法，风险也小。

```
public class OnSaleBook extends NovelBook {

    public OnSaleBook(String name, int price, String author) {
        super(name, price, author);
    }

    @Override
    public String getName() {
        return super.getName();
    }

    @Override
    public int getPrice() {
        int OnsalePrice = super.getPrice();
        int salePrce = 0;
        if (OnsalePrice >4000){
            salePrce = OnsalePrice * 70/100;
        }else{
            salePrce = OnsalePrice * 80/100;
        }
        return  salePrce;
    }

    @Override
    public String getAuthor() {
        return super.getAuthor();
    }
}

```
上面的代码是扩展出来的一个类，而不是在原来的类中进行的修改。

```
public class BookStore {
    private final static ArrayList<Books> sBookList = new ArrayList<Books>();

    static {
        sBookList.add(new OnSaleBook("天龙八部", 4400, "金庸"));
        sBookList.add(new OnSaleBook("射雕英雄传", 7600, "金庸"));
        sBookList.add(new OnSaleBook("钢铁是怎么炼成的", 7500, "保尔·柯查金"));
        sBookList.add(new OnSaleBook("红楼梦", 3300, "曹雪芹"));
    }

    public static void main(String[] args) throws IOException {
        NumberFormat format = NumberFormat.getCurrencyInstance();
        format.setMaximumFractionDigits(2);
       System.out.println("----书店卖出去的书籍记录如下---");
        for (Books book : sBookList) {
            System.out.println("书籍名称:" + book.getName()
                    + "\t书籍作者:" + book.getAuthor()
                    + "\t书籍价格:" + format.format(book.getPrice() / 100.00) + "元");
        }
    }
}

```

结果展示：

```
----书店卖出去的书籍记录如下---
书籍名称:天龙八部	书籍作者:金庸	书籍价格:￥30.80元
书籍名称:射雕英雄传	书籍作者:金庸	书籍价格:￥53.20元
书籍名称:钢铁是怎么炼成的	书籍作者:保尔·柯查金	书籍价格:￥52.50元
书籍名称:红楼梦	书籍作者:曹雪芹	书籍价格:￥26.40元

Process finished with exit code 0

```

在开闭原则中，抽象化是一个关键，解决问题的关键在于抽象化，在JAVA语言这种面向对象的语言中，可以给系统定义出一个一劳永逸的，不再更改的抽象化的设计，此设计允许拥有无穷无尽的实现层被实现。

在JAVA语言中，可以给出一个或者多个抽象的JAVA类或者是JAVA接口，规定所有的具体类必须提供方法特征作为系统设计的抽象层，这个抽象层会遇见所有的可能出现的扩展，因此，在任何扩展情况下都不回去改变，这就让系统的抽象层不需要修改，从而满足开闭原则的第二条，对修改进行闭合。

同时，从抽象层里面导出一个或者多个新的具体类可以改变系统的行为，这样就满足了开闭原则的第一条。

尽管很多时候我们无法百分百的做到开闭原则，但是如果向着这个方向去努力，就能够有部分的成功，这也是可以改善系统的结构的。

我是懿，一个正在被打击还在努力前进的码农。欢迎大家关注我们的公众号，加入我们的知识星球，我们在知识星球中等着你的加入。

