---
layout: post
title: java中List元素移除元素的那些坑
tagline: by 炸鸡可乐
categories: java
tags: 
  - java
---

本文主要介绍，java中list集合移除元素的那些坑，今天小编就和大家一起来了解一下吧！

<!--more-->

### 一、问题案例
#### 1.1、for循环移除
```
public static void main(String[] args) {
	List<String> list = new ArrayList<String>();
	list.add("11");
	list.add("11");
	list.add("12");
	list.add("13");
	list.add("14");
	list.add("15");
	list.add("16");
	System.out.println("原始list元素："+ list.toString());
	CopyOnWriteArrayList<String> copyList = new CopyOnWriteArrayList<>(list);
	
	//通过下表移除等于11的元素
	for (int i = 0; i < list.size(); i++) {
		String item = list.get(i);
		if("11".equals(item)) {
			list.remove(i);
		}
	}
	System.out.println("通过下表移除后的list元素："+ list.toString());
	
	//通过对象移除等于11的元素
	for (int i = 0; i < copyList.size(); i++) {
		String item = copyList.get(i);
		if("11".equals(item)) {
			copyList.remove(item);
		}
	}
	System.out.println("通过对象移除后的list元素："+ list.toString());
	
}
```
输出结果：
```
原始list元素：[11, 11, 12, 13, 14, 15, 16]
通过下表移除后的list元素：[11, 12, 13, 14, 15, 16]
通过对象移除后的list元素：[11, 12, 13, 14, 15, 16]
```
有没有发现有蹊跷的地方？

从输出结果可以看的出，移除后的元素，并没有把内容为`11`的都移除掉！

发生了什么？

删除了第一个`11`后，集合里的元素个数减1，后面的元素往前移了1位，此时，第二个`11`已经移到了索引index=1的位置，而此时i马上i++了，list.get(i)获得的是数据`12`。同时`list.size()`的值也在减小。所以最后输出那个结果。
#### 1.2、fore循环移除
```
public static void main(String[] args) {
	List<String> list = new ArrayList<String>();
	list.add("11");
	list.add("11");
	list.add("12");
	list.add("13");
	list.add("14");
	list.add("15");
	list.add("16");
	System.out.println("原始list元素："+ list.toString());
	
	
	//通过对象移除等于11的元素
	for (String item : list) {
		if("11".equals(item)) {
			list.remove(item);
		}
	}
	System.out.println("通过对象移除后的list元素："+ list.toString());
	
}
```
输出结果：
![](http://www.justdojava.com/assets/images/2019/java/image-jay/e25a59e2606c4bbebeaecfd2a5de72d5.jpg)

抛ConcurrentModificationException异常！

`foreach` 写法实际上是对的 `Iterable`、`hasNext`、`next`方法的简写。因此我们从`List.iterator()`着手分析，跟踪`iterator()`方法，该方法返回了 `Itr` 迭代器对象。

找到`List`的迭代器类
```
public Iterator<E> iterator() {
	return new Itr();
}
```
`Itr`对象
```
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    Itr() {}

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

	/**报错的地方*/
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```
通过代码我们发现 `Itr` 是 `ArrayList` 中定义的一个私有内部类，在 `next`、`remove`方法中都会调用 `checkForComodification` 方法，该方法的作用是判断 `modCount != expectedModCount`是否相等，如果不相等则抛出`ConcurrentModificationException`异常。每次正常执行 `remove` 方法后，都会对执行`expectedModCount = modCount`赋值，保证两个值相等！

那么问题基本上已经清晰了，在` foreach` 循环中执行 `list.remove(item)`;，对 `list` 对象的 `modCount` 值进行了修改，而 `list` 对象的迭代器的 `expectedModCount` 值未进行修改，因此抛出了`ConcurrentModificationException`异常。
### 二、解决办法
#### 2.1、采用倒序移除
```
public static void main(String[] args) {
	List<String> list = new ArrayList<String>();
	list.add("11");
	list.add("11");
	list.add("12");
	list.add("13");
	list.add("14");
	list.add("15");
	list.add("16");
	System.out.println("原始list元素："+ list.toString());
	CopyOnWriteArrayList<String> copyList = new CopyOnWriteArrayList<>(list);
	
	//通过下表移除等于11的元素
	for (int i = list.size() - 1; i >= 0; i--) {
		String item = list.get(i);
		if("11".equals(item)) {
			list.remove(i);
		}
	}
	System.out.println("通过下表移除后的list元素："+ list.toString());
	
	//通过对象移除等于11的元素
	for (int i = copyList.size() - 1; i >= 0; i--) {
		String item = copyList.get(i);
		if("11".equals(item)) {
			copyList.remove(item);
		}
	}
	System.out.println("通过对象移除后的list元素："+ list.toString());
	
}
```
输出结果：
```
原始list元素：[11, 11, 12, 13, 14, 15, 16]
通过下表移除后的list元素：[12, 13, 14, 15, 16]
通过对象移除后的list元素：[12, 13, 14, 15, 16]
```
#### 2.2、fore的解决办法
```
public static void main(String[] args) {
	List<String> list = new ArrayList<String>();
	list.add("11");
	list.add("11");
	list.add("12");
	list.add("13");
	list.add("14");
	list.add("15");
	list.add("16");
	System.out.println("原始list元素："+ list.toString());
	CopyOnWriteArrayList<String> copyList = new CopyOnWriteArrayList<>(list);
	
	//通过对象移除等于11的元素
	for (String item : copyList) {
		if("11".equals(item)) {
			copyList.remove(item);
		}
	}
	System.out.println("通过对象移除后的list元素："+ copyList.toString());
	
}
```
输出结果：
```
原始list元素：[11, 11, 12, 13, 14, 15, 16]
通过对象移除后的list元素：[12, 13, 14, 15, 16]
```
#### 2.3、使用迭代器移除
```
public static void main(String[] args) {
	List<String> list = new ArrayList<String>();
	list.add("11");
	list.add("11");
	list.add("12");
	list.add("13");
	list.add("14");
	list.add("15");
	list.add("16");
	System.out.println("原始list元素："+ list.toString());
	
	//通过迭代器移除等于11的元素
	Iterator<String> iterator = list.iterator();
	while(iterator.hasNext()) {
		String item = iterator.next();
		if("11".equals(item)) {
			iterator.remove();
		}
	}
	System.out.println("通过迭代器移除后的list元素："+ list.toString());
	
}
```
输出结果：
```
原始list元素：[11, 11, 12, 13, 14, 15, 16]
通过迭代器移除后的list元素：[12, 13, 14, 15, 16]
```
#### 2.4、jdk1.8的写法
```
public static void main(String[] args) {
	List<String> list = new ArrayList<String>();
	list.add("11");
	list.add("11");
	list.add("12");
	list.add("13");
	list.add("14");
	list.add("15");
	list.add("16");
	System.out.println("原始list元素："+ list.toString());
	
	//jdk1.8移除等于11的元素
	list.removeIf(item -> "11".equals(item));
	System.out.println("移除后的list元素："+ list.toString());
	
}
```
输出结果：
```
原始list元素：[11, 11, 12, 13, 14, 15, 16]
通过迭代器移除后的list元素：[12, 13, 14, 15, 16]
```
是不是好简单，哈哈！
### 三、总结
如果开发中需要在集合中移除某个元素，如果jdk是1.8的，建议直接使用2.4方法，如果是低版本，那么建议采用迭代器方法，效率高，性能好！

### 四、参考
部分内容参考：[cdsn那是2008](https://blog.csdn.net/claram/article/details/53410175)
