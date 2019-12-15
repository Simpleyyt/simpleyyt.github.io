---
layout: post
categories: Java
title: for循环用了那么多次，但你真的了解它么？
tagline: by 淼淼之森
tags: 
  - 淼淼之森
---

其实我们写代码的时候一直都在使用for循环，但是偶尔还是会纠结用哪一个循环。
<!--more-->

## 一、基础的for循环
0、使用while也是一种循环方式，此处探究for相关的循环，就不做拓展了。

1、遍历数组的时候，初学时是使用的如下样式的for循环：
```
for(int i=0;i<a.length;i++){
    System.out.println(n);
}
```
2、而遍历集合的时候使用的都是Iterator迭代器：

给定一组人名，两两组队（此处允许自己和自己组队），实现如下：
```
enum Option {Tom, Jerry, Jack, Mary}
```
想象中的写法是：
```
Collection<Option> options = Arrays.asList(Option.values());
for(Iterator<Option> i = options.iterator(); i.hasNext();){
    for (Iterator<Option> j = options.iterator(); j.hasNext();) {
        System.out.println(i.next()+" "+j.next());
    }
```
但是执行过后你会发现这段代码是有瑕疵的，出现的结果只有四组：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2019-12/01/1.jpg) 

那么剩下的组合去哪里了呢？

这里程序并不会抛出异常，只是单纯的因为`i.next()`每次都会取下一个值，所以就出现了上图的情况。

但是，如果外部集合的元素大于内部元素：

例如下面这段代码：
```
enum OptionFirst {Tom, Jerry, Jack, Mary}
enum OptionSecond {Tom, Jerry, Jack, Mary, Mali, Tomsun, Lijie, Oolyyou}

public static void main(String[] args) {
    Collection<OptionFirst> optionFirstCollection = Arrays.asList(OptionFirst.values());
    Collection<OptionSecond> optionSecondCollection = Arrays.asList(OptionSecond.values());
    for (Iterator<OptionFirst> i = optionFirstCollection.iterator(); i.hasNext(); ) {
        for (Iterator<OptionSecond> j = optionSecondCollection.iterator(); j.hasNext(); ) {
            System.out.println(i.next() + " " + j.next());
        }
    }
}
```
运行后，就会抛出`java.util.NoSuchElementException`异常，造成的原因就是因为外部循环调用了多次，而内不循环因为元素不足，导致循环抛出了这样的异常。

要想解决这种困扰只需要在二次循环前添加一个变量来保存外部元素；即可实现想要达到的效果。

```
Collection<Option> options = Arrays.asList(Option.values());
for(Iterator<Option> i = options.iterator(); i.hasNext();){
    Option option = i.next();
    for (Iterator<Option> j = options.iterator(); j.hasNext();) {
        System.out.println(option+" "+j.next());
    }
}
```

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2019-12/01/2.jpg) 


## 二、for-each循环
这种循环方式不论是数组还是集合都实用，而且效率更高；表达形式更加简洁明了。
```
for(Element e:elements){
    System.out.println(e);
}
```
当再次遇到上面的两两组队问题时，根本不需要考虑元素不足的问题，而且代码也简洁多了：
```
for (Option option : options) {
    for (Option rank : options) {
        System.out.println(option + " " + rank);
    }
}
```
《Effective Java》中是这样子写for-each循环的：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2019-12/01/3.jpg) 

## 三、for-each is not god
说了for-each那么多好处，但是它也不是神，并非万能的，有那么三种情况是它需要注意的。
### 3.1、解构过滤的时候不能使用
如果需要遍历集合，并删除选定的元素，就需要使用显式的迭代器，以便可以调用它的remove方法。不过在Java8中增加的Collection的removeIf方法常常可以避免显式的遍历。

例如下面这段代码：
```
List<String> list = new ArrayList<String>();

list.add("1");
list.add("2");

for (String item : list) {
    if ("1".equals(item)) {
        list.remove(item);
        System.out.println("执行if语句成功");
    }
}
```
直接运行这段代码是没什么问题的，数组list能成功删除元素1，也能打印对应语句。

但是，我们进行如下任意一种操作：
- 若把list.remove(item)换成list.add(“3”);操作如何？
- 若在第6行添加list.add("3");那么代码会出错吗？
- 若把if语句中的“1”换成“2”，结果你感到意外吗？

如果都能正确执行当然就不需要问了，所以3个都会报ConcurrentModificationException的异常；

![执行结果异常](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2019-12/01/4.jpg) 

而出现这些情况的原因稍稍解释下就是：

首先，这涉及多线程操作，Iterator是不支持多线程操作的，List类会在内部维护一个modCount的变量，用来记录修改次数。

举例：ArrayList源码
```
protected transient int modCount = 0;
```
每生成一个Iterator，Iterator就会记录该modCount，每次调用next()方法就会将该记录与外部类List的modCount进行对比，发现不相等就会抛出多线程编辑异常。

为什么这么做呢？我的理解是你创建了一个迭代器，该迭代器和要遍历的集合的内容是紧耦合的，意思就是这个迭代器对应的集合内容就是当前的内容，我肯定不会希望在我冒泡排序的时候，还有线程在向我的集合里插入数据对吧？所以Java用了这种简单的处理机制来禁止遍历时修改集合。

至于为什么删除“1”就可以呢，原因在于foreach和迭代器的hasNext()方法，foreach这个语法，实际上就是
```
while(itr.hasNext()){
    itr.next()
}
```
所以每次循环都会先执行hasNext()，那么看看ArrayList的hasNext()是怎么写的：
```
public boolean hasNext() {
    return cursor != size;
}
```
cursor是用于标记迭代器位置的变量，该变量由0开始，每次调用next执行+1操作，于是：

所以代码在执行删除“1”后，size=1，cursor=1，此时hasNext()返回false，结束循环，因此你的迭代器并没有调用next查找第二个元素，也就无从检测modCount了，因此也不会出现多线程修改异常；但当你删除“2”时，迭代器调用了两次next，此时size=1，cursor=2，hasNext()返回true，于是迭代器傻乎乎的就又去调用了一次next()，因此也引发了modCount不相等，抛出多线程修改的异常。

当你的集合有三个元素的时候，你就会神奇的发现，删除“1”是会抛出异常的，但删除“2”就没有问题了，究其原因，和上面的程序执行顺序是一致的。

因此，在《阿里巴巴Java开发手册中有这样一条规定》：

![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2019-12/01/5.jpg) 

### 3.2、转换
如果需要遍历列表或数组，并取代它的部分或者全部元素值，就需要使用列表迭代器或者数组索引，以便替换元素的值。

因为for-each是一循到底的，中间不做停留和位置信息的显示；所以要替换元素就不能使用它了。

### 3.3、平行迭代
如果需要并行的遍历多个集合，就需要显式的控制迭代器或者索引变量，以便所有迭代器或者索引变量都可以同步前进（就像上面讲述Iterator迭代器的时候提到的组合减少的情况，只想出现下标一一对应的元素组合）。

## 四、总结
for-each循环不仅适用于遍历集合和数组，而且能让你遍历任何实现Iterator接口的对象；最最关键的是它还没有性能损失。而对数组或集合进行修改（添加删除操作），就要用for循环。

所以循环遍历所有数据的时候，能用它的时候还是选择它吧，嘻嘻。