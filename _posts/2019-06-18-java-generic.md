---
layout: post
title: java泛型特性，你了解多少？
tagline: by 炸鸡可乐
categories: Java
tags: 
  - Java
---

对java泛型特性的了解，很多时候是从集合对象接触到的，今天小编带大家一起去深入的了解泛型的缘由和使用方式！

<!--more-->

### 01、泛型的由来
> 小编想告诉大家的是：泛型的产生本质是来源于软件设计！

在软件设计的过程中经常会用到容器类，容器类代码都一样只是数据类型不同，如果能够让一种类型容纳所有类型，就可以实现代码重用，但是没有一种类型可以容纳所有类型，为了解决容器的问题，由此就产生了泛型设计。

由此可见，泛型是一个不确定的参数类型，即“参数化类型”！

泛型在java中有很重要的地位，在面向对象编程及各种设计模式中有非常广泛的应用。

那么怎么使用泛型，进行软件设计呢？
### 02、使用方式
> 泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法

#### 2.1、泛型类
> 泛型类型用于类的定义中，典型的就是各种容器类，如：List、Set、Map。

最普通的自定义泛型类
```
/**
 * 此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
 * 在实例化泛型类时，必须指定T的具体类型
 */
public class Generic<T>{ 
    /**key这个成员变量的类型为T,T的类型由外部指定*/
    private T key;

	/**泛型构造方法形参key的类型也为T，T的类型由外部指定*/
    public Generic(T key) {
        this.key = key;
    }

	/**泛型方法getKey的返回值类型为T，T的类型由外部指定*/
    public T getKey(){
        return key;
    }
}
```
传参方式
```
//泛型的类型参数只能是类类型（包括自定义类），不能是简单类型
//传入的实参类型需与泛型的类型参数类型相同，即为Integer.
Generic<Integer> genericInteger = new Generic<Integer>(12345678);

//传入的实参类型需与泛型的类型参数类型相同，即为String.
Generic<String> genericString = new Generic<String>("hello");
System.out.println("泛型测试","key is " + genericInteger.getKey());
System.out.println("泛型测试","key is " + genericString.getKey());
```
输出结果
```
泛型测试: key is 12345678
泛型测试: key is hello
```
定义的泛型类，就一定要传入泛型类型实参么？

并不是这样，在使用泛型的时候如果传入泛型实参，则会根据传入的泛型实参做相应的限制，此时泛型才会起到本应起到的限制作用。如果不传入泛型类型实参的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。

举个例子
```
Generic generic = new Generic("111111");
Generic generic1 = new Generic(4444);
Generic generic2 = new Generic(55.55);
Generic generic3 = new Generic(false);

System.out.println("泛型测试","key is " + generic.getKey());
System.out.println("泛型测试","key is " + generic1.getKey());
System.out.println("泛型测试","key is " + generic2.getKey());
System.out.println("泛型测试","key is " + generic3.getKey());
```
输出结果
```
泛型测试: key is 111111
泛型测试: key is 4444
泛型测试: key is 55.55
泛型测试: key is false
```
总结：

1、泛型的类型参数只能是类类型，不能是简单类型。
2、不能对确切的泛型类型使用instanceof操作。如下面的操作是非法的，编译时会出错。如下：
```
if(ex_num instanceof Generic<Number>){   
}
```
#### 2.2、泛型接口
> 泛型接口与泛型类的定义及使用基本相同，泛型接口常被用在各种类的生产器中。

```
/**
 * 定义一个泛型接口
 */
public interface Generator<T> {
    public T next();
}
```
当实现泛型接口的类，未传入泛型实参时
```
/**
 * 未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中
 *
 * 如果不声明泛型，编译器会报错："Unknown class"
 */
public class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}
```
当实现泛型接口的类，传入泛型实参时
```
/**
 * 传入泛型实参时：
 * 定义一个生产器实现这个接口,虽然我们只创建了一个泛型接口Generator<T>
 * 但是我们可以为T传入无数个实参，形成无数种类型的Generator接口。
 * 在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型
 * 即：Generator<T>，public T next();中的的T都要替换成传入的String类型。
 */
public class FruitGenerator implements Generator<String> {

    @Override
    public String next() {
        return 'hello world';
    }
}
```
#### 2.3、泛型方法
> 泛型方法，是在调用方法的时候指明泛型的具体类型 。


```
/**
 * 泛型方法的基本介绍
 * @param tClass 传入的泛型实参
 * @return T 返回值为T类型
 */
public <T> T genericMethod(Class<T> tClass)throws InstantiationException ,
  IllegalAccessException{
        T instance = tClass.newInstance();
        return instance;
}
```
说明：

1、public 与 返回值中间`<T>`非常重要，可以理解为声明此方法为泛型方法。
2、只有声明了`<T>`的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法，如get、set。
3、`<T>`表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
4、与泛型类的定义一样，此处T可以随便写为任意标识，常见的如`T、E、K、V`等形式的参数常用于表示泛型。

方法调用方式
```
//通过泛型方法，实例化一个Test对象
Object obj = genericMethod(Class.forName("com.test.Test"));
```
### 03、其他使用介绍
#### 3.1、泛型通配符
还是举例子，我们知道`Ingeter`是`Number`的一个子类，那么问题来了，在使用`Generic<Number>`作为形参的方法中，能否使用`Generic<Ingeter>`的实例传入呢？

为了弄清楚这个问题，我们使用`Generic<T>`这个泛型类做例子：
```
public class Generic<T> {
	
	private T key;

	public T getKey() {
		return key;
	}

	public void setKey(T key) {
		this.key = key;
	}

	public Generic() {
		super();
		// TODO Auto-generated constructor stub
	}

	public Generic(T key) {
		super();
		this.key = key;
	}
}
```
测试
```
public class GenericTest {
	
	public static void main(String[] args) {
		//编译器会为我们报错：Generic<java.lang.Integer> cannot be applied to Generic<java.lang.Number>
		Generic<Integer> generic1 = new Generic<Integer>(1);
		new GenericTest().showKeyValue(generic1);
	}
	
	public void showKeyValue(Generic<Number> obj){
		System.out.println("泛型测试:key value is " + obj.getKey());
	}
}
```
通过提示信息我们可以看到`Generic<Integer>`不能被看作为`Generic<Number>`的子类。由此可以看出:同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

如何解决上面的问题？总不能为了定义一个新的方法来处理`Generic<Integer>`类型的类，这显然与java中的多态理念相违背。因此我们需要一个在逻辑上可以表示同时是`Generic<Integer>`和`Generic<Number>`父类的引用类型。由此类型通配符应运而生。

将上面的方法改一下：
```
public void showKeyValue(Generic<?> obj){
	   System.out.println("泛型测试:key value is " + obj.getKey());
}
```
类型通配符一般是使用`?`代替具体的类型实参。
#### 3.2、泛型上下边界
> 在使用泛型的时候，我们还可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。

* 上界通配符

为泛型添加上边界，即传入的类型实参必须是指定类型的子类型！例如：
```
public void showKeyValue(Generic<? extends Number> obj) {
		System.out.println("泛型测试:key value is " + obj.getKey());
}
```
测试
```
public static void main(String[] args) {
		Generic<Integer> generic1 = new Generic<Integer>(1);
		//编译器会提示错误，因为String类型并不是Number类型的子类
		Generic<String> generic2 = new Generic<String>("1");
		new GenericTest().showKeyValue(generic1);
		new GenericTest().showKeyValue(generic2);
}
```
* 下界通配符

下界通配符的意思是容器中只能存放T及其T的基类类型的数据，说白了，就是跟上界相反的过程
```
public void showKeyValue(Generic<? super Integer> obj) {
		System.out.println("泛型测试:key value is " + obj.getKey());
}
```
测试
```
public static void main(String[] args) {
		Generic<Integer> generic1 = new Generic<Integer>(1);
		Generic<Number> generic2 = new Generic<Number>(100);
		new GenericTest().showKeyValue(generic1);
		new GenericTest().showKeyValue(generic2);
}
```
输出结果
```
泛型测试:key value is 1
泛型测试:key value is 100
```
最后简单介绍下Effective Java这本书里面介绍的PECS原则。

* 上界<? extends T>不能往里存，只能往外取，适合频繁往外面读取内容的场景。
* 下界<? super T>不影响往里存，但往外取只能放在Object对象里，适合经常往里面插入数据的场景。

是不是听的很懵，没关系，看上面的例子就可以了，哈哈哈！
### 04、总结
本文中的例子主要是为了阐述泛型中的一些思想而简单举出的，并不一定有着实际的可用性。另外，一提到泛型，相信大家用到最多的就是在集合中，其实，在实际的编程过程中，自己可以使用泛型去简化开发，且能很好的保证代码质量。