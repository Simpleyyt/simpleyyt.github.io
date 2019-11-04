---
layout: post
category: Java
title: 到底是存在还是不存在之 BloomFilter
tagline: by 子悠
tags:
  - BloomFilter
---

### 01、什么是 BloomFilter（布隆过滤器）

**布隆过滤器**（英语：Bloom Filter）是 1970 年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。主要用于判断一个元素是否在一个集合中。通常我们会遇到很多要判断一个元素是否在某个集合中的业务场景，这个时候往往我们都是采用 Hashmap，Set 或者其他集合将数据保存起来，然后进行对比判断，但是如果元素很多的情况，我们如果采用这种方式就会非常浪费空间。这个时候我们就需要 BloomFilter 来帮助我们了。

<!--more-->

#### 1.1、BloomFilter 原理

BloomFilter 是由一个固定大小的二进制向量或者位图（bitmap）和一系列（通常好几个）映射函数组成的。布隆过滤器的原理是，当一个变量被加入集合时，通过 K 个映射函数将这个变量映射成位图中的 K 个点，把它们置为 1。查询某个变量的时候我们只要看看这些点是不是都是 1 就可以大概率知道集合中有没有它了，如果这些点有任何一个 0，则被查询变量一定不在；如果都是 1，则被查询变量很**可能**在。注意，这里是可能存在，不一定一定存在！这就是布隆过滤器的基本思想。

如下图所示，字符串 "ziyou" 在经过四个映射函数操作后在位图上有四个点被设置成了 1。当我们需要判断 “ziyou” 字符串是否存在的时候只要在一次对字符串进行映射函数的操作，得到四个 1 就说明 “ziyou” 是可能存在的。

![image-20191022235305602](http://www.justdojava.com/assets/images/2019/java/image_ziyou/bloomfilter01.png)



为什么说是可能存在，而不是一定存在呢？那是因为映射函数本身就是散列函数，散列函数是会有碰撞的，意思也就是说会存在一个字符串可能是 “ziyou01” 经过相同的四个映射函数运算得到的四个点跟 “ziyou” 是一样的，这种情况下我们就说出现了误算。另外还有可能这四个点位上的 1 是四个不同的变量经过运算后得到的，这也不能证明字符串 “ziyou” 是一定存在的，如下图框出来的 1 也可能是字符串“张三”计算得到，同理其他几个位置的 1 也可以是其他字符串计算得到。

![image-20191023001036999](http://www.justdojava.com/assets/images/2019/java/image_ziyou/bloomfilter02.png)

#### 1.2 特性

所以通过上面的例子我们就可以明确

- **一个元素如果判断结果为存在的时候元素不一定存在，但是判断结果为不存在的时候则一定不存在**。

- **布隆过滤器可以添加元素，但是不能删除元素**。因为删掉元素会导致误判率增加。

### 02、使用场景

#### 2.1、网页 URL 去重

我们在使用网页爬虫的时候（爬虫需谨慎），往往需要记录哪些 URL 是已经爬取过的，哪些还是没有爬取过，这个时候我们就可以采用 BloomFilter 来对已经爬取过的 URL 进行存储，这样在进行下一次爬取的时候就可以判断出这个 URL 是否爬取过。

#### 2.2、黑白名单存储

工作中经常会有一个特性针对不同的设备或者用户有不同的处理方式，这个时候可能会有白名单或者黑名单存在，所以根据 BloomFilter 过滤器的特性，我们也可以用它来存在这些数据，虽然有一定的误算率，但是在一定程度上还是可以很好的解决这个问题的。

#### 2.3、小结

除了上面说的两种场景，其实还有很多场景，比如热点数据访问，垃圾邮件过滤等等，其实这些场景的统一特性就是要判断某个元素是否在某个集合中，原理都是一样的。

### 03、代码实践

#### 3.1、自己实现

```java
package com.test.pkg;

import java.util.BitSet;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author ziyou<br>
 * <b>Date：</b>2019-10-23 23:21<br>
 * <b>Desc：</b>无<br>
 */
public class BloomFilterTest {

    /**
     * 初始化布隆过滤器的 bitmap 大小
     */
    private static final int DEFAULT_SIZE = 2 << 24;
    /**
     * 为了降低错误率，这里选取一些数字作为基准数
     */
    private static final int[] seeds = {3, 5, 7, 11, 13, 31, 37, 61};
    /**
     * 设置 bitmap
     */
    private static BitSet bitset = new BitSet(DEFAULT_SIZE);
    /**
     * 设置 hash 函数数量
     */
    private static HashFunction[] functions = new HashFunction[seeds.length];


    /**
     * 添加数据
     *
     * @param value 需求加入的值
     */
    public static void put(String value) {
        if (value != null) {
            for (HashFunction f : functions) {
                //计算 hash 值并修改 bitmap 中相应位置为 true
                bitset.set(f.hash(value), true);
            }
        }
    }

    /**
     * 判断相应元素是否存在
     *
     * @param value 需要判断的元素
     * @return 结果
     */
    public static boolean check(String value) {
        if (value == null) {
            return false;
        }
        boolean ret = true;
        for (HashFunction f : functions) {
            ret = bitset.get(f.hash(value));
            //一个 hash 函数返回 false 则跳出循环
            if (!ret) {
                break;
            }
        }
        return ret;
    }

    public static void main(String[] args) {
        String value = "test";
        for (int i = 0; i < seeds.length; i++) {
            functions[i] = new HashFunction(DEFAULT_SIZE, seeds[i]);
        }
        put(value);
        System.out.println(check("value"));
    }
}

class HashFunction {

    private int size;
    private int seed;

    public HashFunction(int size, int seed) {
        this.size = size;
        this.seed = seed;
    }

    public int hash(String value) {
        int result = 0;
        int len = value.length();
        for (int i = 0; i < len; i++) {
            result = seed * result + value.charAt(i);
        }
        int r = (size - 1) & result;
        return (size - 1) & result;
    }

}

```

上面我们自己写了一个简单的 BloomFilter ，通过 put 方法录入数据，通过 check 方法判断元素是否存在，基本能实现功能，代码中注释也写的很清楚，但是自己实现必定效率不高，所以下面我们看下业内大佬帮我们已经实现好的 BloomFilter。

#### 2.4、Guava 中的 BloomFilter

```java
package com.test.pkg;

import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author ziyou<br>
 * <b>Date：</b>2019-10-24 00:17<br>
 * <b>Desc：</b>无<br>
 */
public class BloomFilterTest02 {

    public static void main(String[] args) {
        BloomFilter<Integer> bloomFilter = BloomFilter.create(Funnels.integerFunnel(), 100000, 0.01);
        for (int i = 0; i < 100000; i++) {
            bloomFilter.put(i);
        }
        System.out.println(bloomFilter.mightContain(1));
        System.out.println(bloomFilter.mightContain(2));
        System.out.println(bloomFilter.mightContain(3));
        System.out.println(bloomFilter.mightContain(100001));
    }
}

```

Guava 中已经帮我们实现好了 BloomFilter 的代码，我们只需要在使用的地方调用就好。

这里我们简单解释一下构造方法中的后面两个参数，一个是预计包含的数据量，一个是允许的误差值。代码中会根据我们填入的这两个值，自动帮我们计算出数组的大小，以及需要的散列函数个数，如下图。更多详细的内容，读者可以自行去查看源码，我们这里就不介绍了。

![image-20191024002725539](http://www.justdojava.com/assets/images/2019/java/image_ziyou/bloomfilter03.png)



![image-20191024002803746](http://www.justdojava.com/assets/images/2019/java/image_ziyou/bloomfilter04.png)

### 04、总结

这篇文章给大家介绍了 BloomFilter，一个用来判断元素是否存在与某个集合的高效方法，可以在我们日常的工作中运用起来，结合日常工作的场景，可以进行选择。另外欢迎大家到我们《Java极客技术》知识星球一起进步学习。目前已经有 1300+ 星友了，我们在星球等你。