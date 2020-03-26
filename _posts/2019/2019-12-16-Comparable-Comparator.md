---
layout: post
categories: Java
title: 浅谈Comparable和Comparator
tagline: by 炭烧生蚝
tags: 
  - 炭烧生蚝
---

# 场景引入

首先我们考虑一个场景：有一个整形数组, 我们希望通过调用一个工具类的排序方法就能对该数组进行排序. 请看下面的代码: 

```java
public class Strategy {
    public static void main(String[] args) {
        int[] arr = {5, 3, 1, 7, 2};
        new DataSorter().sort(arr);//调用工具类进行排序
        for(int i = 0; i < arr.length; i++){
            System.out.println(arr[i]);
        }
    }
}

class DataSorter{//用于排序的工具类
    public void sort(int[] arr){//调用sort方法进行排序, 此处使用冒泡排序
        for(int i = arr.length - 1; i > 0; i--){
            for(int j = 0; j < i; j++){
                if(arr[j] > arr[j + 1])
                    swap(arr, j, j  + 1);
            }
        }
    }

    private void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

# Comparable接口
通过上面的代码, 我们能够轻易地对整形数组进行排序, 那么如果现在有了新需求, 需要对浮点类型数据进行排序, 排序工具类应该如何做呢? 

或许你会想, 不如就新添加一个排序方法, 方法的参数类型为 float 类型, 把 int 类型数组的排序算法复制一遍不就可以了吗? 

那如果我继续追问, 如果现在要对一只猫进行排序, 那应该怎么做呢? 猫的类如下：

```java
class Cat{
    private int age;//猫的年龄
    private int weight;//猫的体重

    //get / set 方法...
}
```

你也许会顺着原来的思路回答, 照样 copy 一份排序的算法, 修改方法参数, 然后在比较的地方指定比较猫的年龄或体重不就可以了吗? 

```java
public void sort(Cat[] arr){//以猫数组作为参数
    for(int i = arr.length - 1; i > 0; i--){
        for(int j = 0; j < i; j++){
            if(arr[j].getAge() > arr[j + 1].getAge())//根据猫的年龄作比较
                swap(arr, j, j  + 1);
        }
    }
}
```

但仔细想想, 如果还要继续比较小狗, 小鸡, 小鸭等各种对象, 那么这个排序工具类的代码量岂不是变得很大? 为了能让排序算法的可重用性高一点, 我们希望排序工具中的 sort() 方法可以对任何调用它的对象进行排序. 

你可能会想: 到对任何对象都能排序, 把 sort() 方法的参数改为 Object 类型不久可以了嘛. 这个方向是对的, 但是问题是, 当拿到两个 Object 类型对象, 应该根据什么规则进行比较呢? 

这个时候我们自然而然地就希望调用工具类进行排序的对象本身就具备自己的**比较法则**, 这样在排序的时候就能直接调用对象的排序法则进行排序了.

我们把比较法则抽象为 Comparable 接口, 凡是要进行比较的类都要实现 Comparable 接口, 并且定义自己的比较法则, 也就是 CompareTo() 方法. 

这样当我们在封装工具时, 就可以直接对实现了 Comparable 接口的对象进行比较, 不用担心比较的细节了. 

```java
public class Strategy {
public class Strategy {
    public static void main(String[] args) {
//      Integer[] arr = {5, 3, 1, 7, 2};//注意这里把int改为Integer, Integer是Object的子类
        Cat[] arr = {new Cat(3, 3), new Cat(1, 1), new Cat(5, 5)};
        DataSorter ds = new DataSorter();
        ds.sort(arr);
        for(int i = 0; i < arr.length; i++){
            System.out.println(arr[i]);
        }
    }
}
}

class DataSorter{//用于排序的工具类
    public void sort(Object[] arr){//参数类型为Object
        for(int i = arr.length - 1; i > 0; i--){
            for(int j = 0; j < i; j++){
                Comparable c1 = (Comparable) arr[j];//先转为Comparable类型
                Comparable c2 = (Comparable) arr[j + 1];
                if(c1.CompareTo(c2) == 1)//调用CompareTo()进行比较, 不关心具体的实现
                    swap(arr, j, j  + 1);
            }
        }
    }

    private void swap(Object[] arr, int i, int j){
        Object temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}

class Cat implements Comparable{
    private int age;
    private int weight;

    @Override
    public int CompareTo(Object o) {
        if(o instanceof Cat){//先判断传入的是否是Cat类对象, 不是则抛异常
            Cat c = (Cat) o;
            if(this.age > c.age) return 1;
            else if (this.age < c.age) return -1;
            else return 0;
        }
        throw null == o ? new NullPointerException() : new ClassCastException();
    }

    // get / set ...
    //toString() ...
}

interface Comparable{
    public int CompareTo(Object o);
}
```

# Comparator接口

相信看了上面的 Comparable 接口来由, 大家会感觉整个设计又美好了一些, 但是其中还有漏洞. 我们在 Cat 类的 CompareTo() 方法中, 对猫的比较策略是写死的, 现在我们按猫的年龄比较大小, 如果哪天我们想按照猫的体重比较大小, 又要去修改源码了. 有没有扩展性更好的设计?

我们可以让用户自己定义一个比较器类, 对象可以根据用户指定的比较器比较大小. 

整个逻辑是: 如果这个对象需要进行比较, 那么它必须实现 Comparable 接口, 但是它具体是怎么比较的, 则通过具体的 Comparator 比较器进行比较. 

当然这里少不了多态, 我们首先要定义一个比较器接口 Comparator, 用户的比较器需要实现 Comparator 接口, 下面上代码:

```java
public class Strategy {
    public static void main(String[] args) {
        Cat[] arr = {new Cat(3, 3), new Cat(1, 1), new Cat(5, 5)};
        DataSorter ds = new DataSorter();
        ds.sort(arr);
        for(int i = 0; i < arr.length; i++){
            System.out.println(arr[i]);
        }
    }
}

class DataSorter{//用于排序的工具类
    public void sort(Object[] arr){//参数类型为Object
        for(int i = arr.length - 1; i > 0; i--){
            for(int j = 0; j < i; j++){
                Comparable c1 = (Comparable) arr[j];//先转为Comparable类型
                Comparable c2 = (Comparable) arr[j + 1];
                if(c1.CompareTo(c2) == 1)//背后已经转换为使用Comparator的定义的规则进行比较
                    swap(arr, j, j  + 1);
            }
        }
    }

    private void swap(Object[] arr, int i, int j){
        Object temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}

class Cat implements Comparable{
    private int age;
    private int weight;
    private Comparator comparator = new CatAgeComparator();//默认持有年龄比较器

    @Override
    public int CompareTo(Object o) {
        return comparator.Compare(this, o);//调用比较器比较而不是直接在此写比较法则
    }

    // get / set / toString ...
}

interface Comparable{
    public int CompareTo(Object o);
}

interface Comparator{
    public int Compare(Object o1, Object o2);
}

//用户自己定义的, 按照猫的年龄比较大小的比较器
class CatAgeComparator implements Comparator{
    @Override
    public int Compare(Object o1, Object o2) {
        Cat c1 = (Cat) o1;
        Cat c2 = (Cat) o2;
        if(c1.getAge() > c2.getAge()) return 1;
        else if(c1.getAge() < c2.getAge()) return -1;
        else return 0;
    }
}

//按照猫的体重比较大小的比较器
class CatWeightComparator implements Comparator{
    @Override
    public int Compare(Object o1, Object o2) {
        Cat c1 = (Cat) o1;
        Cat c2 = (Cat) o2;
        if(c1.getWeight() > c2.getWeight()) return 1;
        else if(c1.getWeight() < c2.getWeight()) return -1;
        else return 0;
    }
}
```

# 关于策略模式的思考

在上面的例子中, 我们自己定义了 Comparable 接口和 Comparator 接口, 其实这两个接口都是 Java 自带的, 通过上面的代码示例, 想必大家也应该知道了为什么会有这两个接口。

其实 Comparable 定义的就是一种比较的策略, 这里的策略你可以理解为一个功能, 然而策略有了, 我们还需要有具体的策略实现, 于是便有了 Comparator 接口。

在比较对象的 Comparable 和 Comparator 中运用策略模式我认为有以下的好处：

- 不同的类在比较时的行为不同，策略模式可以让它们在运行时动态选择具体的比较逻辑。
- 方便对一个对象在不同的时候使用不同的比较策略，方便未来添加不同的比较策略。

但也有一个主要的缺点：

- 因为每个具体的比较策略类都会产生一个新类，随着时间增长，系统需要维护的类数量可能会增加。