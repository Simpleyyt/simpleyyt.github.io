# Java中内部类的骚操作

[TOC]

#### 10.1 如何定义内部类

如代码10.1-1 所示

```java
public class Parcel1 {
    public class Contents{
        private int value = 0;
    
        public int getValue(){
            return value;
        }
    }
}
```

**这是一个很简单的内部类定义方式,你可以直接把一个类至于另一个类的内部，这种定义Contents类的方式被称为内部类**

那么，就像代码10.1-1所展示的，程序员该如何访问Contents中的内容呢？

如代码10.1-2 所示

```java
public class Parcel1 {

    public class Contents{
        private int value = 0;

        public int getValue(){
            return value;
        }
    }

    public Contents contents(){
        return new Contents();
    }

    public static void main(String[] args) {
        Parcel1 p1 = new Parcel1();
        Parcel1.Contents pc1 = p1.contents();
        System.out.println(pc1.getValue());
    }
}
```

输出结果： 0

就像上面代码看到的那样，你可以写一个方法来访问Contents，相当于指向了一个对Contents的引用，可以用外部类.内部类这种定义方式来创建一个对于内部类的引用，就像Parcel1.Contents pc1 = p1.contents();所展示的，而pc1 相当于持有了对于内部类Contents的访问权限。

现在，我就有一个疑问，如果10.1-2 中的contents方法变为静态方法，pc1还能访问到吗？

> 编译就过不去，那么为什么会访问不到呢？请看接下来的分析。

#### 10.2 链接到外部类的方式

看到这里，你还不明白为什么要采用这种方式来编写代码，好像只是为了装逼？或者你觉得重新定义一个类很麻烦，干脆直接定义一个内部类得了，好像到现在并没有看到这种定义内部类的方式为我们带来的好处。请看下面这个例子10.2-1

```java
public class Parcel2 {

    private static int i = 11;

    public class Parcel2Inner {

        public Parcel2Inner(){
            i++;
        }

        public int getValue(){
            return i;
        }

    }

    public Parcel2Inner parcel2Inner(){
        return new Parcel2Inner();
    }

    public static void main(String[] args) {
        Parcel2 p2 = new Parcel2();
        for(int i = 0;i < 5;i++){
            p2.parcel2Inner();
        }
        System.out.println("p2.i = " + p2.i);
    }
}
```

输出结果： 16

当你创建了一个内部类对象的时候，此对象就与它的外围对象产生了某种联系，如上面代码所示，内部类Parcel2Inner 是可以访问到Parcel2中的i的值的，也可以对这个值进行修改。

那么，问题来了，如何创建一个内部类的对象呢？程序员不能每次都写一个方法返回外部类的对象吧？见代码10.2-2

```java
public class Parcel3 {

    public class Contents {

        public Parcel3 dotThis(){
            return Parcel3.this;
        }

        public String toString(){
            return "Contents";
        }
    }

    public Parcel3 contents(){
        return new Contents().dotThis();
    }

    public String toString(){
        return "Parcel3";
    }

    public static void main(String[] args) {
        Parcel3 pc3 = new Parcel3();
        Contents c = pc3.new Contents();
        Parcel3 parcel3 = pc3.contents();
        System.out.println(pc3);
        System.out.println(c);
        System.out.println(parcel3);
    }
}
```

输出: 
Parcel3
Contents
Parcel3

如上面代码所示，Parcel3内定义了一个内部类Contents,内部类中定义了一个方法dotThis(),这个方法的返回值为外部类的对象,在外部类中有一个contents()方法，这个方法返回的还是外部类的引用。

#### 10.3 内部类与向上转型

本文到现在所展示的都是本类持有内部类的访问权限，那么，与此类无关的类是如何持有此类内部类的访问权限呢？而且内部类与向上转型到底有什么关系呢？
如图10.3-1

```java
public interface Animal {

    void eat();
}

public class Parcel4 {

    private class Dog implements Animal {

        @Override
        public void eat() {
            System.out.println("啃骨头");
        }
    }

    public Animal getDog(){
        return new Dog();
    }

    public static void main(String[] args) {
        Parcel4 p4 = new Parcel4();
        //Animal dog = p4.new Dog();
        Animal dog = p4.getDog();
        dog.eat();
    }
}
```

输出： 啃骨头

这个输出大家肯定都知道了，Dog是由private修饰的，按说非本类的任何一个类都是访问不到，那么为什么能够访问到呢？ 仔细想一下便知，因为Parcel4 是public的，而Parcel4是可以访问自己的内部类的，那么Animal也可以访问到Parcel4的内部类也就是Dog类，并且Dog类是实现了Animal接口，所以getDog()方法返回的也是Animal类的子类，从而达到了向上转型的目的，让代码更美妙。

#### 10.4 定义在方法中和任意作用域内部的类

上面所展示的一些内部类的定义都是普通内部类的定义，如果我想在一个方法中或者某个作用域内定义一个内部类该如何编写呢？
你可能会考虑这几种定义的思路：

1. 我想定义一个内部类，它实现了某个接口，我定义内部类是为了返回接口的引用
2. 我想解决某个问题，并且这个类又不希望它是公共可用的，顾名思义就是封装起来，不让别人用
3. 因为懒...

以下是几种定义内部类的方式：

- 一个在方法中定义的类(局部内部类)
- 一个定义在作用域内的类，这个作用域在方法的内部(成员内部类)
- 一个实现了接口的匿名类(匿名内部类)
- 一个匿名类，它扩展了非默认构造器的类
- 一个匿名类，执行字段初始化操作
- 一个匿名类，它通过实例初始化实现构造
- __定义在方法内部的类又被称为局部内部类__

```java
public class Parcel5 {
        private Destination destination(String s){
    
            class PDestination implements Destination{
    
                String label;
    
                public PDestination(String whereTo){
                    label = whereTo;
                }
    
                @Override
                public String readLabel() {
                    return label;
                }
            }
            return new PDestination(s);
        }
    
        public static void main(String[] args) {
            Parcel5 p5 = new Parcel5();
            Destination destination = p5.destination("China");
            System.out.println(destination.readLabel());
        }
}
```

输出 ： China

如上面代码所示，你可以在编写一个方法的时候，在方法中插入一个类的定义，而内部类中的属性是归类所有的，我在写这段代码的时候很好奇,内部类的执行过程是怎样的，Debugger走了一下发现当执行到p5.destination("China")的时候，先会执行return new PDestination(s)，然后才会走PDestination的初始化操作，这与我们对其外部类的初始化方式是一样的，只不过这个方法提供了一个访问内部类的入口而已。
__注: 局部内部类的定义不能有访问修饰符__

- __一个定义在作用域内的类，这个作用域在方法的内部__

```java
public class Parcel6 {
        // 吃椰子的方法
        private void eatCoconut(boolean flag){
            // 如果可以吃椰子的话
            if(flag){
                class Coconut {
                    private String pipe;
    
                    public Coconut(String pipe){
                        this.pipe = pipe;
                    }
    
                    // 喝椰子汁的方法
                    String drinkCoconutJuice(){
                        System.out.println("喝椰子汁");
                        return pipe;
                    }
                }
                // 提供一个吸管，可以喝椰子汁
                Coconut coconut = new Coconut("用吸管喝");
                coconut.drinkCoconutJuice();
            }
    
            /**
             * 如果可以吃椰子的话，你才可以用吸管喝椰子汁
             * 如果不能接到喝椰子汁的指令的话，那么你就不能喝椰子汁
             */
            // Coconut coconut = new Coconut("用吸管喝");
            // coconut.drinkCoconutJuice();
        }
    
        public static void main(String[] args) {
            Parcel6 p6 = new Parcel6();
            p6.eatCoconut(true);
        }
}
```

输出： 喝椰子汁

如上面代码所示，只有程序员告诉程序，现在我想吃一个椰子，当程序接收到这条命令的时候，它回答好的，马上为您准备一个椰子，并提供一个吸管让您可以喝到新鲜的椰子汁。程序员如果不想吃椰子的话，那么程序就不会为你准备椰子，更别说让你喝椰子汁了。

- __一个实现了匿名接口的类__

我们都知道接口是不能被实例化的，也就是说你不能return 一个接口的对象，你只能是返回这个接口子类的对象，但是如果像下面这样定义，你会不会表示怀疑呢？

```java
public interface Contents {

    int getValue();
}

public class Parcel7 {

    private Contents contents(){
        return new Contents() {

            private int value = 11;

            @Override
            public int getValue() {
                return value;
            }
        };
    }

    public static void main(String[] args) {
        Parcel7 p7 = new Parcel7();
        System.out.println(p7.contents().getValue());
    }
}
```

输出 : 11

为什么能够返回一个接口的定义？而且还有 {}，这到底是什么鬼？ 这其实是一种匿名内部类的写法，其实和上面所讲的内部类和向上转型是相似的。也就是说匿名内部类返回的new Contents()其实也是属于Contents的一个实现类，只不过这个实现类的名字被隐藏掉了，能用如下的代码示例来进行转换：

```java
public class Parcel7b {

    private class MyContents implements Contents {

        private int value = 11;

        @Override
        public int getValue() {
            return 11;
        }
    }

    public Contents contents(){
        return new MyContents();
    }

    public static void main(String[] args) {
        Parcel7b parcel7b = new Parcel7b();
        System.out.println(parcel7b.contents().getValue());
    }
}
```

输出的结果你应该知道了吧～！ 你是不是觉得这段代码和 10.3 章节所表示的代码很一致呢？

- __一个匿名类，它扩展了非默认构造器的类__

如果你想返回一个带有参数的构造器(非默认的构造器)，该怎么表示呢？

```java
public class WithArgsConstructor {

    private int sum;

    public WithArgsConstructor(int sum){
        this.sum = sum;
    }

    public int sumAll(){
        return sum;
    }
}

public class Parcel8 {

    private WithArgsConstructor withArgsConstructor(int x){

        // 返回WithArgsConstructor带参数的构造器，执行字段初始化
        return new WithArgsConstructor(x){

            // 重写sumAll方法，实现子类的执行逻辑
            @Override
            public int sumAll(){
                return super.sumAll() * 2;
            }
        };
    }

    public static void main(String[] args) {
        Parcel8 p8 = new Parcel8();
        System.out.println(p8.withArgsConstructor(10).sumAll());
    }
}
```

以上WithArgsConstructor 中的代码很简单，定义一个sum的字段，构造器进行初始化，sumAll方法返回sum的值，Parcel8中的withArgsConstructor方法直接返回x的值，但是在这个时候，你想在返回值上做一些特殊的处理，比如你想定义一个类，重写sumAll方法，来实现子类的业务逻辑。 Java编程思想198页中说 代码中的“;”并不是表示内部类结束，而是表达式的结束，只不过这个表达式正巧包含了匿名内部类而已。

- **一个匿名类，它能够执行字段初始化**

上面代码确实可以进行初始化操作，不过是通过构造器执行字段的初始化，如果没有带参数的构造器，还能执行初始化操作吗？ 这样也是可以的。

```java 
public class Parcel9 {

    private Destination destination(String dest){
        return new Destination() {

            // 初始化赋值操作
            private String label = dest;

            @Override
            public String readLabel() {
                return label;
            }
        };
    }

    public static void main(String[] args) {
        Parcel9 p9 = new Parcel9();
        System.out.println(p9.destination("pen").readLabel());
    }
}
```

如果给字段进行初始化操作，那么形参必须是final的，如果不是final，编译器会报错，这部分提出来质疑，因为我不定义为final，编译器也没有报错。
我考虑过是不是private的问题，当我把private 改为public,也没有任何问题。

我不清楚是中文版作者翻译有问题，还是经过这么多Java版本的升级排除了这个问题，我没有考证原版是怎样写的，这里还希望有知道的大牛帮忙解释一下这个问题。

- **一个匿名类，它通过实例初始化实现构造**

```java
public abstract class Base {

    public Base(int i){
        System.out.println("Base Constructor = " + i);
    }

    abstract void f();
}

public class AnonymousConstructor {

    private static Base getBase(int i){

        return new Base(i){
            {
                System.out.println("Base Initialization" + i);
            }

            @Override
            public void f(){
                System.out.println("AnonymousConstructor.f()方法被调用了");
            }
        };
    }

    public static void main(String[] args) {
        Base base = getBase(57);
        base.f();
    }
}
```

输出：
Base Constructor = 57
Base Initialization 57
AnonymousConstructor.f()方法被调用了

这段代码和 "一个匿名类，它扩展了非默认构造器的类" 中属于相同的范畴，都是通过构造器实现初始化的过程。

#### 10.5 嵌套类

10.4 我们介绍了6种内部类定义的方式，现在我们来解决一下10.1 提出的疑问，为什么contents()方法变成静态的，会编译出错的原因：

如果不需要内部类与其外围类之前产生关系的话，就把内部类声明为static。这通常称为嵌套类，也就是说嵌套类的内部类与其外围类之前不会产生某种联系，也就是说内部类虽然定义在外围类中，但是确实可以独立存在的。嵌套类也被称为静态内部类。
静态内部类意味着：
__（1）要创建嵌套类的对象，并不需要其外围类的对象__
__（2）不能从嵌套类的对象中访问非静态的外围类对象__

代码示例 10.5-1

```java
public class Parcel10 {

    private int value = 11;

    static int bValue = 12;

    // 静态内部类
    private static class PContents implements Contents {

        // 编译报错，静态内部类PContents中没有叫value的字段
        @Override
        public int getValue() {
            return value;
        }

        // 编译不报错，静态内部类PContents可以访问静态属性bValue
        public int f(){
            return bValue;
        }
    }

    // 普通内部类
    private class PDestination implements Destination {

        @Override
        public String readLabel() {
            return "label";
        }
    }

    // 编译不报错，因为静态方法可以访问静态内部类
    public static Contents contents(){
        return new PContents();
    }

    // 编译报错，因为非静态方法不能访问静态内部类
    public Contents contents2(){
        Parcel10 p10 = new Parcel10();
        return p10.new PContents();
    }

    // 编译不报错，静态方法可以访问非静态内部类
    public static Destination destination(){
        Parcel10 p10 = new Parcel10();
        return p10.new PDestination();
    }

    // 编译不报错，非静态方法可以访问非静态内部类
    public Destination destination2(){
        return new PDestination();
    }
}
```

由上面代码可以解释，10.1编译出错的原因是 静态方法不能直接访问非静态内部类，而需要通过创建外围类的对象来访问普通内部类。

##### 接口内部的类

纳尼？接口内部只能定义方法，难道接口内部还能放一个类吗？可以！
正常情况下，不能在接口内部放置任何代码，但是嵌套类作为接口的一部分，你放在接口中的任何类默认都是public和static的。因为类是static的，只是将嵌套类置于接口的命名空间内，这并不违反接口的规则，你甚至可以在内部类实现外部类的接口，不过一般我们不提倡这么写

```java
public interface InnerInterface {

    void f();

    class InnerClass implements InnerInterface {

        @Override
        public void f() {
            System.out.println("实现了接口的方法");
        }

        public static void main(String[] args) {
            new InnerClass().f();
        }
    }

    // 不能在接口中使用main方法，你必须把它定义在接口的内部类中
//    public static void main(String[] args) {}
}
```

输出： 实现了接口的方法

##### 内部类实现多重继承

在Java中，类与类之间的关系通常是一对一的，也就是单项继承原则，那么在接口中，类与接口之间的关系是一对多的，也就是说一个类可以实现多个接口，而接口和内部类结合可以实现"多重继承"，并不是说用extends关键字来实现，而是接口和内部类的对多重继承的模拟实现。

参考chenssy的文章 http://www.cnblogs.com/chenssy/p/3389027.html 已经写的很不错了。

```java
public class Food {

    private class InnerFruit implements Fruit{
        void meakFruit(){
            System.out.println("种一个水果");
        }
    }

    private class InnerMeat implements Meat{
        void makeMeat(){
            System.out.println("煮一块肉");
        }
    }

    public Fruit fruit(){
        return new InnerFruit();
    }

    public Meat meat(){
        return new InnerMeat();
    }

    public static void main(String[] args) {
        Food food = new Food();
        InnerFruit innerFruit = (InnerFruit)food.fruit();
        innerFruit.meakFruit();
        InnerMeat innerMeat = (InnerMeat) food.meat();
        innerMeat.makeMeat();
    }
}
```

输出： 
种一个水果
煮一块肉

#### 10.6 内部类的继承

内部类之间也可以实现继承，与普通类之间的继承相似，不过不完全一样。

```java
public class BaseClass {

    class BaseInnerClass {

        public void f(){
            System.out.println("BaseInnerClass.f()");
        }
    }

    private void g(){
        System.out.println("BaseClass.g()");
    }
}
/**
 *  可以看到，InheritInner只是继承自内部类BaseInnerClass，而不是外围类
 *  但是默认的构造方式会报编译错误，
 *  必须使用类似enclosingClassReference.super()才能编译通过
 *  用来来说明内部类与外部类对象引用之间的关联。
 *
 */
public class InheritInner extends BaseClass.BaseInnerClass{

    // 编译出错
//    public InheritInner(){}

    public InheritInner(BaseClass bc){
        bc.super();
    }

    @Override
    public void f() {
        System.out.println("InheritInner.f()");
    }

    /*
    * 加上@Override 会报错，因为BaseInnerClass 中没有g()方法
    * 这也是为什么覆写一定要加上Override注解的原因，否则默认是本类
    * 中持有的方法，会造成误解，程序员以为g()方法是重写过后的。
    @Override
    public void g(){
        System.out.println("InheritInner.g()");
    }*/

    public static void main(String[] args) {
        BaseClass baseClass = new BaseClass();
        InheritInner inheritInner = new InheritInner(baseClass);
        inheritInner.f();
    }
}
```

输出：InheritInner.f()

#### 10.7 内部类的覆盖

关于内部类的覆盖先来看一段代码：

```java
public class Man {

    private ManWithKnowledge man;

    protected class ManWithKnowledge {

        public void haveKnowledge(){
            System.out.println("当今社会是需要知识的");
        }
    }

    // 我们想让它输出子类的haveKnowledge()方法
    public Man(){
        System.out.println("当我们有了一个孩子，我们更希望他可以当一个科学家，而不是网红");
        new ManWithKnowledge().haveKnowledge();
    }
}

// 网红
public class InternetCelebrity extends Man {

    protected class ManWithKnowledge {

        public void haveKnowledge(){
            System.out.println("网红是当今社会的一种病态");
        }
    }

    public static void main(String[] args) {
        new InternetCelebrity();
    }
}
```

输出：当我们有了一个孩子，我们更希望他可以当一个科学家，而不是网红
     	 当今社会是需要知识的

我们默认内部类是可以覆盖的。所以我们想让他输出 InternetCelebrity.haveKnowledge() ,来实现我们的猜想，但是却输出了ManWithKnowledge.haveKnowledge()方法。
这个例子说明当继承了某个外围类的时候，内部类并没有发生特别神奇的变化，两个内部类各自独立，都在各自的命名空间内。

#### 10.8 关于源码中内部类的表示

由于每个类都会产生一个.class 文件，包含了创建该类型对象的全部信息
同样的，内部类也会生成一个.class 文件
表示方法为:
OneClass$OneInnerClass 

__后记: 
内部类属于Java语言的高级特性，内部类在实际的应用场景比较少，内部类可以让代码变的更加优雅，但是对于平常开发的话用处不是很大，毕竟不是所有同事都能了解全面内部类的特性和用法，所以在平常开发中尽量避免使用内部类，加大维护成本。内部类应该在设计阶段考虑使用，你应该区分内部类和接口的应用场景，就算用不到，你也get到了一项新技能，就算装逼也能落落大方的不是吗？__

#### 10.9 内部类的优点

1、封装部分代码，当你创建一个内部类的时候，该内部类默认持有外部类的引用；

2、内部类具有一定的灵活性，无论外围类是否继承某个接口的实现，对于内部类都没有影响；

3、内部类能够有效的解决多重继承的问题。