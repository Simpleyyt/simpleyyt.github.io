---
layout: post  
title: Java Grammar：数组
tagline: by Vi的技术博客
categories: Java  
tag: 
    - Java

---

> 数组，一种应用非常广泛的数据结构，简单地来说就是一组**类型相同**且**无序**的元素的存储在**固定长度**且有序**的内存空间。



### 创建一个数组

在Java中，我们可以通过`[]`去声明一个指定类型的数组：

```java
int[] a; // 写法一
int a[]; // 写法二
```

当然，一般情况下我们更喜欢使用第一种方式来声明一个数组，因为它将类型与变量名分开，优化了代码的可读性。 
刚刚我们只是声明了一个数组a，但是并没有将a初始化为一个真正的数组。

在给数组赋值时，我们

可以通过三种方式：

```java
int[] a = new int[4];
int[] a = new int[]{1,2,3,4};
int[] a = {1,4,3,2}
```

其中第三种实际上是第二种的简写，我们可以通过使用new关键字去创建一个匿名的数组：

```java
new int[4];
```

但是记得一定要指定长度或者指定数组中的元素，这里如果想要创建一个匿名的数组，new关键字是必不可少的：

```java
{1,2,4,3} // 这样写是错误的！
```

无论我们怎么去定义一个数组，它的长度在创建之初都是被确定的，但是需要注意一点，它的长度也不是无穷无尽的，我们可以通过查看反射包中的`Array`类源码获得它的长度数据类型：

```java
public static Object newInstance(Class<?> componentType, int length)
        throws NegativeArraySizeException {
        return newArray(componentType, length);
}
```

这里可以看到数组的数据类型是int类型，而int类型在前面我们也提过，它的最大长度是$2^{31}$，也就是2GB。



### 访问数组中的元素

我们可以通过下标的方式来访问数组中的元素，数组的下标从0开始，最大长度是数组的长度，如果我们访问超出数组下标范围的数据，就会抛出索引越界异常（ArrayOutOfIndexError），因为我们可以通过下标直接访问数组中的元素，所以时间复杂度是O(1)。

```java
int[] a = {1,2,3};
System.out.println(a[0]); // 1
```



### 往数组中添加元素

刚刚我们说过，数组中的长度是固定的，所以我们无法去改变该数组的结构，但是我们可以通过另外一种方法来实现这样的效果：

```java
		int[] arr = {9,7,5};
		int[] temp = new int[arr.length+1];
		for(int i = 0;i < arr.length;i++) { 
			temp[i]=arr[i];
		}
		temp[arr.length] = 6;
		arr = temp;
```

![](http://www.justdojava.com/assets/images/2019/java/image_vi/08_31/2019-08-26-142707.png)



### 删除元素

和新增一样，删除数组中的元素同样是不允许的，我们可以通过和新增类似的方式来完成删除的操作：

```java
int[] arr = { 1, 2, 3, 4, 5};
int[] tmp = new int[arr.length - 1];
for (int i = 0; i < tmp.length; i++) {
  tmp[i] = arr[i];
}
arr = tmp;
```

原理上和新增是比较类似的，这里我就不再画图去详细的说明了~



### 二维数组（了解）

我们像创建一维数组一样可以创建一个二维数组

```java
int[][] doubleArr = new int[2][3];
int[][] doubleArr = {{1,2,3,4},{5,6,7,8}};
int[][] doubleArr = new int[5][];
```

这里需要注意一点，二维数组的创建时，可以指定一个维度的长度，而不指定第二维度的长度，使之动态的变化。比如我们可以画个星星

```java
String[][] arr = new String[5][];
for (int i = 0; i < arr.length; i++) {
  arr[i] = new String[i + 1];
  for (int j = 0; j < arr[i].length;j++) {
    arr[i][j] = "*";
  }
}

for (int i = 0; i < arr.length; i++) {
  for (int j = 0; j < arr[i].length;j++) {
    System.out.print(arr[i][j]);
  }
  System.out.println();
}
```


![](http://www.justdojava.com/assets/images/2019/java/image_vi/08_31/2019-08-26-151109.png)


