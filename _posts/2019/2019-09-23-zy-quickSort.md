---
layout: post
categories: 算法
title: 你应该知道的常用排序算法之快速排序
tagline: by 子悠
tags: 
  - 子悠

---

**人生有涯，学海无涯**

今天跟大家分享一个常用的排序算法——快速排序。我想大家在日常的工作或者面试的时候肯定经常会遇到很多排序算法，而且快速排序算法往往是这里面相对更好的排序算法，并且实现也非常简单。下面我们一起看下吧。

<!--more-->

### 01、快速排序

我们先看看维基百科的解释:

>  **快速排序**（英语：QuickSort），又称**划分交换排序**（partition-exchange sort），简称**快排**，一种排序算法，最早由东尼·霍尔提出。在平均状况下，排序 n 个项目要 O(nlogn) 次比较。在最坏状况下则需要 O(n^2) 次比较，但这种状况并不常见。事实上，快速排序通常明显比其他算法更快，因为它的内部循环（inner loop）可以在大部分的架构上很有效率地达成。



### 02、算法思想

快速排序的算法思想是分而治之，将一个大的待排序列，分成两个子序列，然后采用递归的方式，依次将子序列也分成更小的子序列，依次进行，最后得到排序好的序列。算法的实现主要分成三步

1. 找到基准点：
2. 排列序列，将比基准点小的放在左边的子序列，将比基准点大的放在右边的子序列；
3. 采用递归，依次重新选取基准点，在重复进行 1，2 步骤，得到最终的顺序序列



### 03、算法实现

```java
/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author 子悠<br>
 * <b>Date：</b>2019-09-23 01:07<br>
 * <b>Desc：</b>无<br>
 */
public class QuickSort {

    public static void main(String[] args) {
        int[] array = new int[]{2, 3, 1, 4, 7, 8, 3, 5, 2, 6, 8, 9, 1};
        quickSort(array, 0, array.length - 1);
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + " ");
        }
    }

    /**
     * 递归排序
     *
     * @param array 待排序列
     * @param left  左边起始位置
     * @param right 右边结束位置
     */
    private static void quickSort(int[] array, int left, int right) {
        if (left < right) {
            //根据基准点，找到分隔左右子序列的位置索引
            int position = position(array, left, right);
            //分别进行左右的递归
            quickSort(array, left, position - 1);
            quickSort(array, position + 1, right);
        }
    }

    /**
     * 找到中间
     *
     * @param array 待排序列
     * @param left  左边起始位置
     * @param right 右边结束位置
     * @return
     */
    private static int position(int[] array, int left, int right) {
        //找到基准点, 这里使用的是序列的第一个元素
        int base = array[left];
        while (left < right) {
            while (right > left && array[right] >= base) {
                right--;
            }
            //交互位置
            array[left] = array[right];
            while (left < right && array[left] <= base) {
                left++;
            }
            //交互位置
            array[right] = array[left];
        }
        //此时 left 与 right 是相等的
        array[left] = base;
        return left;
    }
}

```

运行结果：

> 1 1 2 2 3 3 4 5 6 7 8 8 9 

从运行的结果我们看到，已经正常的排序结束了，说明这个算法已经满足了我们的要求，而且详细的代码分析也已经加上了注释，我想大家应该都能看懂。只要记住核心的几个点就可以了，这里我在重复说明一下：

1. 先找基准点 base；
2. 比较大小，比 base 小的放在左边序列，比 base 大的放在右边序列；
3. 递归左右序列。

注意上面内部的两个 `while` 循环，这里是使用类似两个指针，分别从序列的左右两个端点开始往中间进行遍历，主要进行的第二步比较和赋值的操作。

### 04、总结

这篇文章简单给大家介绍了一下快速排序的思想和实现，排序算法是大家日常工作和学习不可避免要学习的知识，我们《Java 极客技术》也会在后续的文章中慢慢的给大家分享很多优质的，算法和数据结构的文章，便于大家学习和成长。最后欢迎大家到我们《Java 极客技术》知识星球中一起学习和进步，扫描下放二维码还有优惠，我们在知识星球中等你。

![子悠-知识星球](http://justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)

