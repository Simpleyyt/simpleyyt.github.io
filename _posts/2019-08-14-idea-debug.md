---
layout: post
title: idea debug高级特性看这篇就够了
tagline: by 小马
category: java
tag: 
    - java
---

* content
{:toc}

 
所谓工欲善其事必先利其器，从eclipse转idea也有一段时间了。一直想总结下idea调试的一些高级技巧。debug过程如果高效，撸代码也会爽很多，不是吗？

<!--more-->
 
## 多线程调试
  
直接上例子说明，比如下面这段代码
  
![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/1-1.jpg)

  
debug模式下调试的时候，发现断点并不会按照我预想的执行，子线程里的断点根本没有执行，两个子线程直接悄无声息的就跑完了。说白了就是我们没有办法进入到线程里断点调试。
  
有解决方案吗？ 当然
  
我们只需要对断点做一些设置即可，
  
![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/1-1.jpg)

  
如上图，把断点的suspend模式改为thread，然后我们再去调试，发现断点可以按照我们预想的执行了。在debug窗口可以通过子线程名选择进入子线程的流程单步调试。
  
![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/1-2.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/1-3.jpg)

## 循环遍历条件断点
 
有时候在遍历一个循环的时候，我们只是想看某个条件下的变量状态。通常情况下都是一遍遍执行直到到达我们想要的条件为止。
 
其实可以不用这么麻烦，idea提供了条件断点的功能，比如下面这段代码：
 
![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/2-1.jpg)

 
我们右键断点为止，设置i=5的条件断点，然后debug执行。会发现程序在i=5的情况下才会进入断点。
 
![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/2-2.jpg)

 
##  显示方法返回值

比如下面这段代码

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/3-1.jpg)


正常情况下我们调试的时候，是看不到random到底是多少的。解决方法有一个，就是把random的结果赋值到一个变量里，然后在调试窗口观察这个变量的值。

其实idea支持更方便的方法解决上面的问题。调试窗口里的Settings -> Show Method Return Values开关可以显示方法的返回值。

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/3-2.jpg)

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/3-3.jpg)


## 调试过程中动态修改变量的值

比如下面这段代码，

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/4-1.jpg)

当执行到if分支判断时，可能不会进入我们想调试的分支。通常情况下我们是会在代码里给number写死一个值进行调试。但是这种方式修改了业务代码，如果发布的时候忘记删除，结果不堪设想。

idea有更加优雅的方式解决此类需求。

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/4-2.jpg)

只需要在debug窗口，通过set value动态修改变量的值，就可以按照我们希望的分支继续执行。是不是很给力！


## 调试内存泄露

调试内存泄露的关键是能查看堆内存的使用详情，有了详细的信息才好定位出现问题的代码。

我们先写一段示例代码，代码中有各种类型的内存分配。

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/5-1.jpg)


然后开始debug，在idea的右下窗口打开memory view，

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/5-2.jpg)

 我一般是选择打开这几个选项，
 
![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/5-3.jpg)

接下来在单步的过程中，就可以看到对象的分配情况。diff栏显示的是每一步对象数目的变化，这个非常有用。因为通过这个你可以看到你的代码对堆内存分配和回收的动作。

![](http://www.justdojava.com/assets/images/2019/java/image_xiaoma/idea-debug/5-4.jpg)

参考:

https://www.jetbrains.com/help/idea/debugging-code.html



 









