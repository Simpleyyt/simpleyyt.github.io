---
layout: post
title: hashCode和equals的区别
tagline: by Jay pan
categories: java基础
tags: 
    - Jaypan
---

有面试官会问：你重写过 hashcode 和 equals 么，为什么重写equals时必须重写hashCode方法？equals和hashCode都是Object对象中的非final方法，它们设计的目的就是被用来覆盖(override)的，所以在程序设计中还是经常需要处理这两个方法。下面我们一起来看一下，它们到底有什么区别，总结一波！

<!--more-->

### 01、hashCode介绍
> hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数。

举个例子
```
public class DemoTest {

	public static void main(String[] args) {
		Object obj = new Object();
		System.out.println(obj.hashCode());
	}
}
```
通过调用hashCode()方法获取对象的hash值。
### 02、equals介绍
> equals它的作用也是判断两个对象是否相等，如果对象重写了equals()方法，比较两个对象的内容是否相等；如果没有重写，比较两个对象的地址是否相同，价于“==”。同样的，equals()定义在JDK的Object.java中，这就意味着Java中的任何类都包含有equals()函数。

举个例子
```
public class DemoTest {

	public static void main(String[] args) {
		Object obj = new Object();
		System.out.println(obj.equals(obj));
	}
}
```
### 03、hashCode() 和 equals() 有什么关系？
接下面，我们讨论另外一个话题。网上很多文章将 hashCode() 和 equals 关联起来，有的讲的不透彻，有误导读者的嫌疑。在这里，我们梳理了一下 “hashCode() 和 equals()的关系”。我们以“**类的用途**”来将“hashCode() 和 equals()的关系”分2种情况来说明。

#### 3.1、不会创建“类对应的散列表”
>这里所说的“不会创建类对应的散列表”是说：我们不会在HashSet, HashTable, HashMap等等这些本质是散列表的数据结构中，用到该类。例如，不会创建该类的HashSet集合。

**在这种情况下，该类的“hashCode() 和 equals() ”没有半毛钱关系的！**
    
equals() 用来比较该类的两个对象是否相等，而hashCode() 则根本没有任何作用，所以，不用理会hashCode()。
举个例子
```
public class DemoNormalTest {

	public static void main(String[] args) {
		// 新建2个相同内容的Person对象，
		// 再用equals比较它们是否相等
		Person p1 = new Person("eee", 100);
		Person p2 = new Person("eee", 100);
		Person p3 = new Person("aaa", 200);

		System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
		System.out.printf("p1.equals(p3) : %s; p1(%d) p3(%d)\n", p1.equals(p3), p1.hashCode(), p3.hashCode());
	}

	private static class Person {

		private String name;

		private int age;

		public Person(String name, int age) {
			super();
			this.name = name;
			this.age = age;
		}

		/**
		 * 重写equals方法
		 */
		@Override
		public boolean equals(Object obj) {
			if (obj == null) {
				return false;
			}

			// 如果是同一个对象返回true，反之返回false
			if (this == obj) {
				return true;
			}

			// 判断是否类型相同
			if (this.getClass() != obj.getClass()) {
				return false;
			}
			Person person = (Person) obj;
			return name.equals(person.name) && age == person.age;
		}
	}
}
```
运行结果：
```
p1.equals(p2) : true; p1(2018699554) p2(1311053135)
p1.equals(p3) : false; p1(2018699554) p3(1735600054)
```
从结果也可以看出：p1和p2相等的情况下，hashCode()也不一定相等。
#### 3.2、会创建“类对应的散列表”
> 这里所说的“会创建类对应的散列表”是说：我们会在HashSet, HashTable, HashMap等等这些本质是散列表的数据结构中，用到该类。例如，创建该类的HashSet集合。

**在这种情况下，该类的“hashCode() 和 equals() ”是有关系的:**
+ 如果两个对象相等，那么它们的hashCode()值一定相同。这里的相等是指，通过equals()比较两个对象时返回true。
+ 如果两个对象hashCode()相等，它们并不一定相等。因为在散列表中，hashCode()相等，即两个键值对的哈希值相等。然而哈希值相等，并不一定能得出键值对相等，此时就出现所谓的**哈希冲突**场景。

举个例子
```
public class DemoConflictTest {

	public static void main(String[] args) {
		// 新建Person对象，
        Person p1 = new Person("eee", 100);
        Person p2 = new Person("eee", 100);
        Person p3 = new Person("aaa", 200);

        // 新建HashSet对象 
        HashSet<Person> set = new HashSet<>();
        set.add(p1);
        set.add(p2);
        set.add(p3);

        // 比较p1 和 p2， 并打印它们的hashCode()
        System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
        // 打印set
        System.out.printf("set:%s\n", set);
	}
	
	private static class Person {

		private String name;

		private int age;

		public Person(String name, int age) {
			super();
			this.name = name;
			this.age = age;
		}

		/**
		 * 重写toString方法
		 */
		@Override
		public String toString() {
			return "("+name + ", " +age+")";
		}

		/**
		 * 重写equals方法
		 */
		@Override
		public boolean equals(Object obj) {
			if (obj == null) {
				return false;
			}

			// 如果是同一个对象返回true，反之返回false
			if (this == obj) {
				return true;
			}

			// 判断是否类型相同
			if (this.getClass() != obj.getClass()) {
				return false;
			}
			Person person = (Person) obj;
			return name.equals(person.name) && age == person.age;
		}
	}
}
```
运行结果：
```
p1.equals(p2) : true; p1(2018699554) p2(1311053135)
set:[(eee, 100), (aaa, 200), (eee, 100)]
```
结果分析：

**我们重写了Person的equals()。但是，很奇怪的发现：HashSet中仍然有重复元素：p1 和 p2。为什么会出现这种情况呢？**

**这是因为虽然p1 和 p2的内容相等，但是它们的hashCode()不等；所以，HashSet在添加p1和p2的时候，认为它们不相等。**

举个例子，我们同时覆盖equals() 和 hashCode()方法。
```
public class DemoConflictTest {

	public static void main(String[] args) {
		// 新建Person对象，
		Person p1 = new Person("eee", 100);
		Person p2 = new Person("eee", 100);
		Person p3 = new Person("aaa", 200);
		Person p4 = new Person("EEE", 100);

		// 新建HashSet对象
		HashSet<Person> set = new HashSet<>();
		set.add(p1);
		set.add(p2);
		set.add(p3);
		set.add(p4);

		// 比较p1 和 p2， 并打印它们的hashCode()
		System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
		// 比较p1 和 p4， 并打印它们的hashCode()
		System.out.printf("p1.equals(p4) : %s; p1(%d) p4(%d)\n", p1.equals(p4), p1.hashCode(), p4.hashCode());
		// 打印set
		System.out.printf("set:%s\n", set);
	}

	private static class Person {

		private String name;

		private int age;

		public Person(String name, int age) {
			super();
			this.name = name;
			this.age = age;
		}

		/**
		 * 重写toString方法
		 */
		@Override
		public String toString() {
			return "(" + name + ", " + age + ")";
		}

		/**
		 * 重写equals方法
		 */
		@Override
		public boolean equals(Object obj) {
			if (obj == null) {
				return false;
			}

			// 如果是同一个对象返回true，反之返回false
			if (this == obj) {
				return true;
			}

			// 判断是否类型相同
			if (this.getClass() != obj.getClass()) {
				return false;
			}
			Person person = (Person) obj;
			return name.equals(person.name) && age == person.age;
		}

		/**
		 * 重写hashCode方法
		 */
		@Override
		public int hashCode() {
			int nameHash = name.toUpperCase().hashCode();
			return nameHash ^ age;
		}

	}
}
```
运行结果：
```
p1.equals(p2) : true; p1(68545) p2(68545)
p1.equals(p4) : false; p1(68545) p4(68545)
set:[(eee, 100), (EEE, 100), (aaa, 200)]
```
结果分析：

**这下，equals()生效了，HashSet中没有重复元素。**
**比较p1和p2，我们发现：它们的hashCode()相等，通过equals()比较它们也返回true。所以，p1和p2被视为相等。**
**比较p1和p4，我们发现：虽然它们的hashCode()相等；但是，通过equals()比较它们返回false。所以，p1和p4被视为不相等。**

* **为什么HashSet会用到hashCode()呢？**

查看HashSet的源码部分
```
/**
 * HashSet部分
 */
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
}


/**
 * map.put方法部分
 */
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}

/**
 * putVal方法部分
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
可以看出，hashSet使用的是hashMap的put方法，而hashMap的put方法，使用hashCode()用key作为参数计算出hash值，然后进行比较，如果相同，再通过equals()比较key值是否相同，如果相同，返回同一个对象。

**所以，如果类使用再散列表的集合对象中，要判断两个对象是否相同，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。**

### 04、有哪些覆写hashCode的诀窍
> 一个好的hashCode的方法的目标：为不相等的对象产生不相等的散列码，同样的，相等的对象必须拥有相等的散列码。

1、把某个非零的常数值，比如17，保存在一个int型的result中；

2、对于每个关键域f（equals方法中设计到的每个域），作以下操作：
* a.为该域计算int类型的散列码；

```
i.如果该域是boolean类型，则计算(f?1:0),
ii.如果该域是byte,char,short或者int类型,计算(int)f,
iii.如果是long类型，计算(int)(f^(f>>>32)).
iv.如果是float类型，计算Float.floatToIntBits(f).
v.如果是double类型，计算Double.doubleToLongBits(f),然后再计算long型的hash值
vi.如果是对象引用，则递归的调用域的hashCode，如果是更复杂的比较，则需要为这个域计算一个范式，然后针对范式调用hashCode，如果为null，返回0
vii. 如果是一个数组，则把每一个元素当成一个单独的域来处理。
```
* b.result = 31 * result + c;

3、返回result

4、编写单元测试验证有没有实现所有相等的实例都有相等的散列码。

给个简单的例子：
```
@Override
public int hashCode() {
  int result = 17;
  result = 31 * result + name.hashCode();
  return result;
}
```

这里再说下2.b中为什么采用`31*result + c`,乘法使hash值依赖于域的顺序，如果没有乘法那么所有顺序不同的字符串String对象都会有一样的hash值，而31是一个奇素数，如果是偶数，并且乘法溢出的话，信息会丢失，31有个很好的特性是`31*i ==(i<<5)-i`,即2的5次方减1，虚拟机会优化乘法操作为移位操作的。