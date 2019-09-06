---
layout: post  
title: Java Grammar：
tagline: by Vi的技术博客
categories: Java  
tag: 
    - Java

---

### 循环

老生常谈的一个控制流程了，我们在是使用数组和集合的时候，遍历元素的时候经常会用到循环的结构，Java具有非常灵活的三种循环机制：

![image-20190827222828453](http://www.justdojava.com/assets/images/2019/java/image_vi/09_06/2019-08-27-142828.png)

根据是否知道循环的次数可以为分为while循环，do...while循环和for循环，下面我们单独来了解一下：

### while循环

当我们不知道循环的具体次数时，可以使用while循环进行操作，下面是while循环的伪代码

```java
定义初始变量  
while (控制条件) {
	循环体
}
```

![image-20190827222337726](http://www.justdojava.com/assets/images/2019/java/image_vi/09_06/2019-08-27-142337.png)

代码示例：

```java
		//    定义控制循环变量
       int start = 0;
    //    循环条件
       while (start < 2) {
        //    循环体
        System.out.println("1");
        //  改变控制循环变量
        start++;
       }
   }
```

### do...while循环

和while循环类似，do...while循环同样适用于不知道循环具体的次数时，但是和while循环不太一样的是，如果控制循环的变量初始时就不符合循环条件，那么循环体一次也不会执行，而do...while循环至少会把循环体执行一次。

```java
定义初始变量
do {
   循环体
} while (循环条件);
```

![image-20190827223154870](http://www.justdojava.com/assets/images/2019/java/image_vi/09_06/2019-08-27-143155.png)



代码示例：

```java
		//    定义控制循环变量
       int start = 0;
    do {
        // 循环体
        System.out.println("1");
        // 控制循环变量
        start++;
        // 循环条件
    } while(start < 0);
```



### for循环

下面进入了我们的重头戏，日常中使用的最多的for循环，由于普通for循环可以准确的控制循环的次数，所以一般当我们在需要手动控制循环次数的时候，我们会使用普通for循环

```java
for(定义初始变量;判断条件;变量变化){
  循环体
}
```

这里的流程图和while是类似的，下面我们来看一下如何遍历一个数组：

```java
int[] a = {1,2,3,4};
for (int i = 0; i < a.length; i++) {
  System.out.println(a[i]);
}
```

同样的遍历一个集合也可以使用普通for循环：

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);
list.add(4);
for (int i = 0; i < list.size(); i++) {
  System.out.println(list.get(i));
}
```

这里细心的同学可能已经注意到了，我们这里描述的时候一直使用的是普通for循环，那么既然有普通的for循环，就一定有不普通的for循环，下面我们来看一下两种不太普通的for循环

#### 增强for循环

在JDK 5之后，出现了一种语法糖--forEach循环，也称之为增强for循环，循环语法如下

```java
for(数据类型 定义元素名：循环列表) {
  循环体
}
```

foreach语句是for语句的特殊简化版本，但是foreach语句并不能完全取代for语句，然而，任何的foreach语句都可以改写为for语句版本。 foreach并不是一个关键字，习惯上将这种特殊的for语句格式称之为“foreach”语句。



#### 关于增强for循环和普通for循环的效率问题

数组遍历：增强型for循环和普通循环遍历原理相同，效率相同。

集合遍历：增强型for循环的遍历其本质就是迭代器 iterator的遍历,和普通循环遍历相比，各自有自己适用的场景，比如说普通for循环比较适合List类（数组类）遍历通过下标查找数据的，而增强型for循环则比较适合**链表**结构的集合的遍历。

在**数据量较大**的情况下，如果是集合使用增强for循环的效率会低于使用普通for循环。

### 跳出循环的两个关键字

我们在使用的过程中，如果遇到需要中断一个流程的情况，通常会使用到以下两个关键字：`break`和`continue`。

`break` 主要用在循环语句或者 switch 语句中，用来跳出整个语句块。break 跳出最里层的循环，并且继续执行该循环下面的语句。当然我们也可以使用标签的方式来跳出某个指定的循环。

```java
read_data:
while(...) {
    for(...) {
        break read_data;    //这里就是直接跳出了while循环
    }
}
```

`continue` 适用于**任何循环控制结构**中。作用是让程序立刻**跳转到下一次循环**的迭代。在 for 循环中，`continue` 语句使程序立即跳转到更新语句。在 while 或者 do…while 循环中，程序立即跳转到布尔表达式的判断语句。当然，`continue`也有一种带标签的形式，将跳到与标签匹配的循环首部。用法和break一样，这里就不再举例说明。

