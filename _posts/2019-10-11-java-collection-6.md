---
layout: post
title: 【集合系列】- 深入浅出的分析 TreeMap
tagline: by 炸鸡可乐
categories: Java
tags: 
  - Java
---

前面介绍了 Map 接口的实现类 LinkedHashMap，LinkedHashMap 存储的元素是有序的，可以保持元素的插入顺序，但不能对元素进行自动排序。在某些场景，如果在数据的存储过程中，能够自动对数据进行排序，将会极大提高编程效率。而 Map 接口有一个重要的实现类 TreeMap，TreeMap 可以实现存储元素的自动排序。

<!--more-->
### 01、摘要
在集合系列的第一章，咱们了解到，Map 的实现类有 HashMap、LinkedHashMap、TreeMap、IdentityHashMap、WeakHashMap、Hashtable、Properties 等等。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/10bfb0b009e142c7a3aa11f5f92a25f5.jpg)

本文主要从数据结构和算法层面，探讨 TreeMap 的实现。

### 02、简介
> Java TreeMap 实现了 SortedMap 接口，也就是说会按照 key 的大小顺序对 Map 中的元素进行排序，key 大小的评判可以通过其本身的自然顺序（natural ordering），也可以通过构造时传入的比较器（Comparator）。

TreeMap 底层通过红黑树（Red-Black tree）实现，所以要了解 TreeMap 就必须对红黑树有一定的了解，在《集合系列》文章中，如果你已经读过红黑树的讲解，其实本文要讲解的 TreeMap，跟其大同小异。

红黑树又称红-黑二叉树，它首先是一颗二叉树，它具有二叉树所有的特性。同时红黑树更是一颗自平衡的排序二叉树。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/8cd42f0b3479462dafea1d22f9ba7af9.jpg)

对于一棵有效的红黑树二叉树，主要有以下规则：

* **1、每个节点要么是红色，要么是黑色，但根节点永远是黑色的；**
* **2、每个红色节点的两个子节点一定都是黑色；**
* **3、红色节点不能连续（也即是，红色节点的孩子和父亲都不能是红色）；**
* **4、从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点；**
* **5、所有的叶节点都是是黑色的（注意这里说叶子节点其实是上图中的 NIL 节点）；**

这些约束强制了红黑树的关键性质：从根到叶子的最长的可能路径不多于最短的可能路径的两倍长。结果是这棵树大致上是平衡的。因为操作比如插入、删除和查找某个值的最坏情况时间都要求与树的高度成比例，这个在高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。所以红黑树它是复杂而高效的，其检索效率为`O(log n)`。下图为一颗典型的红黑二叉树。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/bd324fdebe3d4d8892ef9869585a7644.jpg)

在树的结构发生改变时（插入或者删除操作），往往会破坏上述规则3或规则4，需要通过调整使得查找树重新满足红黑树的条件。

**调整方式主要有：左旋、右旋和颜色转换！**

#### 2.1、左旋
左旋的过程是将 x 的右子树绕 x 逆时针旋转，使得 x 的右子树成为 x 的父亲，同时修改相关节点的引用。旋转之后，二叉查找树的属性仍然满足。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/038ecb3f01aa4b74b33a969c8eaf8fc3.jpg)

#### 2.2、右旋
右旋的过程是将 x 的左子树绕 x 顺时针旋转，使得 x 的左子树成为 x 的父亲，同时修改相关节点的引用。旋转之后，二叉查找树的属性仍然满足。

![](http://www.justdojava.com/assets/images/2019/java/image-jay/e9607d4384184fca8d74cb5c0b0c442d.jpg)

#### 2.3、颜色转换
颜色转换的过程是将红色节点转换为黑色节点，或者将黑色节点转换为红色节点，以满足红黑树二叉树的规则！

![](http://www.justdojava.com/assets/images/2019/java/image-jay/d333d2b5d07142b3a13b83df04868028.jpg)

### 03、常用方法介绍
#### 3.1、get方法
get 方法根据指定的 key 值返回对应的 value，该方法调用了`getEntry(Object key)`得到相应的 entry，然后返回`entry.value`。

算法思想是根据 key 的自然顺序（或者比较器顺序）对二叉查找树进行查找，直到找到满足`k.compareTo(p.key) == 0`的`entry`。

源码如下：
```
final Entry<K,V> getEntry(Object key) {
		//如果传入比较器，通过getEntryUsingComparator方法获取元素
        if (comparator != null)
            return getEntryUsingComparator(key);
		//不允许key值为null
        if (key == null)
            throw new NullPointerException();
		//使用元素的自然顺序
            Comparable<? super K> k = (Comparable<? super K>) key;
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)
				//向左找
                p = p.left;
            else if (cmp > 0)
				//向右找
                p = p.right;
            else
                return p;
        }
        return null;
}
```
如果传入比较器，通过 getEntryUsingComparator 方法获取元素
```
final Entry<K,V> getEntryUsingComparator(Object key) {
            K k = (K) key;
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            Entry<K,V> p = root;
            while (p != null) {
				//通过比较器顺序，获取元素
                int cmp = cpr.compare(k, p.key);
                if (cmp < 0)
                    p = p.left;
                else if (cmp > 0)
                    p = p.right;
                else
                    return p;
            }
        }
        return null;
}
```
测试用例：
```
public static void main(String[] args) {
        Map initMap = new TreeMap();
        initMap.put("4", "d");
        initMap.put("3", "c");
        initMap.put("1", "a");
        initMap.put("2", "b");
        //默认自然排序，key为升序
        System.out.println("默认 排序结果:" + initMap.toString());

		//自定义排序，在TreeMap初始化阶段传入Comparator 内部对象
        Map comparatorMap = new TreeMap<String, String>(new Comparator<String>() {

            @Override
            public int compare(String o1, String o2){
                //根据key比较大小，采用倒叙，以大到小排序
                return o2.compareTo(o1);
            }
        });
        comparatorMap.put("4", "d");
        comparatorMap.put("3", "c");
        comparatorMap.put("1", "a");
        comparatorMap.put("2", "b");

        System.out.println("自定义 排序结果:" + comparatorMap.toString());
}
```
输出结果：
```
默认 排序结果:{1=a, 2=b, 3=c, 4=d}
自定义 排序结果:{4=d, 3=c, 2=b, 1=a}
```
#### 3.2、put方法
put 方法是将指定的 key, value 对添加到 map 里。该方法首先会对 map 做一次查找，看是否包含该元组，如果已经包含则直接返回，查找过程类似于 getEntry() 方法；如果没有找到则会在红黑树中插入新的 entry，如果插入之后破坏了红黑树的约束，还需要进行调整（旋转，改变某些节点的颜色）。

源码如下：
```
public V put(K key, V value) {
        Entry<K,V> t = root;
		//如果红黑树根部为空，直接插入
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
		//如果比较器，通过比较器顺序，找到插入位置
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
			//通过自然顺序，找到插入位置
            if (key == null)
                throw new NullPointerException();
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
		//创建并插入新的entry
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
		//红黑树调整函数
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
}
```
红黑树调整函数`fixAfterInsertion(Entry<K,V> x)`
```
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;
    while (x != null && x != root && x.parent.color == RED) {
		//判断x是否在树的左边，还是右边
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
			//如果x的父亲的父亲的右子树是红色，违反了红色节点不能连续
            if (colorOf(y) == RED) {
				//进行颜色调整，以满足红色节点不能连续规则
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK); 
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
				//如果x的父亲的右子树等于自己，那么需要进行左旋转
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                setColor(parentOf(x), BLACK);  
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
			//跟上面的流程正好相反
			//获取x的父亲的父亲的左子树节点
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
			//如果y是红色节点，违反了红色节点不能连续
            if (colorOf(y) == RED) {
				//进行颜色转换
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
				//如果x的父亲的左子树等于自己，那么需要进行右旋转
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
	//根节点一定为黑色
    root.color = BLACK;
}
```
上述代码的插入流程：

* **1、首先在红黑树上找到合适的位置；**
* **2、然后创建新的entry并插入；**
* **3、通过函数fixAfterInsertion()，对某些节点进行旋转、改变某些节点的颜色，进行调整；**


**调整图解：**

![](http://www.justdojava.com/assets/images/2019/java/image-jay/fe3f8f5b702b46b1bef74da5be9df385.jpg)

#### 3.3、remove方法
remove 的作用是删除 key 值对应的 entry，该方法首先通过上文中提到的`getEntry(Object key)`方法找到 key 值对应的 entry，然后调用`deleteEntry(Entry<K,V> entry)`删除对应的 entry。由于删除操作会改变红黑树的结构，有可能破坏红黑树的约束，因此有可能要进行调整。

源码如下：
```
public V remove(Object key) {
        Entry<K,V> p = getEntry(key);
        if (p == null)
            return null;

        V oldValue = p.value;
        deleteEntry(p);
        return oldValue;
}
```
删除函数 deleteEntry()
```
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;
    if (p.left != null && p.right != null) {// 删除点p的左右子树都非空。
        Entry<K,V> s = successor(p);// 后继
        p.key = s.key;
        p.value = s.value;
        p = s;
    }
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);
    if (replacement != null) {// 删除点p只有一棵子树非空。
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;
        p.left = p.right = p.parent = null;
        if (p.color == BLACK)
            fixAfterDeletion(replacement);// 调整
    } else if (p.parent == null) {
        root = null;
    } else { //删除点p的左右子树都为空
        if (p.color == BLACK)
            fixAfterDeletion(p);// 调整
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```
删除后调整函数 fixAfterDeletion() 的具体代码如下：
```
private void fixAfterDeletion(Entry<K,V> x) {
    while (x != root && colorOf(x) == BLACK) {
		//判断当前删除的元素，是在x父亲的左边还是右边
        if (x == leftOf(parentOf(x))) {
            Entry<K,V> sib = rightOf(parentOf(x));
			//判断x的父亲的右子树，是红色还是黑色节点
            if (colorOf(sib) == RED) {
				//进行颜色转换
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateLeft(parentOf(x));
                sib = rightOf(parentOf(x));
            }
			//x的父亲的右子树的左边是黑色节点，右边也是黑色节点
            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
				//设置x的父亲的右子树为红色节点，将x的父亲赋值给x
                setColor(sib, RED);
                x = parentOf(x);
            } else {
				//x的父亲的右子树的左边是红色节点，右边也是黑色节点
                if (colorOf(rightOf(sib)) == BLACK) {
					//x的父亲的右子树的左边进行颜色调整，右旋调整
                    setColor(leftOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateRight(sib);
                    sib = rightOf(parentOf(x));
                }
				//对x进行左旋，颜色调整
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(rightOf(sib), BLACK);
                rotateLeft(parentOf(x));
                x = root;
            }
        } else { // 跟前四种情况对称
            Entry<K,V> sib = leftOf(parentOf(x));
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateRight(parentOf(x));
                sib = leftOf(parentOf(x));
            }
            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateLeft(sib);
                    sib = leftOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(leftOf(sib), BLACK);
                rotateRight(parentOf(x));
                x = root;
            }
        }
    }
    setColor(x, BLACK);
}
```
上述代码的删除流程：

* **1、首先在红黑树上找到合适的位置；**
* **2、然后删除 entry；**
* **3、通过函数 fixAfterDeletion()，对某些节点进行旋转、改变某些节点的颜色，进行调整；**

### 04、总结
TreeMap 默认是按键值的升序排序，如果需要自定义排序，可以通过`new Comparator`构造参数，重写`compare`方法，进行自定义比较。


以上，主要是对 java 集合中的 TreeMap 做了写讲解，如果有理解不当之处，欢迎指正。

### 05、参考
1、JDK1.7&JDK1.8 源码

2、[博客园 - chenssy - TreeMap分析 ](https://www.cnblogs.com/chenssy/p/3746600.html)

3、[知乎 - CarpenterLee - TreeMap讲解 ](https://zhuanlan.zhihu.com/p/24795143?refer=dreawer)
