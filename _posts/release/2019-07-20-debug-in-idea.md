---
layout: post
categories: IDEA
title: 深入学习 Intellij IDEA 调试技巧
tagline: by 小黑
tags: 
  - 小黑

---

程序员的日常工作除了写代码之外，很大一部分时间将会在查找 BUG，解决问题。查找 BUG，离不开在 IDE 中调试代码。熟练的掌握调试技巧，可以帮助我们减少查找时间，快速定位问题。

在 IDEA 中调试代码很方便，只要开启调试，在相应代码处使用断点，然后进行相应调试即可。正是因为只要掌握这几个功能，就可以调试代码，所以在很长一段时间内，不了解其他调试技巧。后续深入使用 IDEA 过程中，不断学会 IDEA 其他调试技巧，很大程度上加快调试速度。

<!--more-->

## Step 系列按钮

日常调试的过程中为了为了找到程序的bug，通常需要单步调试，一步一步跟踪程序执行的流程，根据变量的值，找到最终原因。

IDEA 中提供一系列 Step 功能按钮用于单步调试，可以在 Run 菜单栏找到全部 Step 功能。

![Run 菜单栏](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-5fa9ce2f.png)

另外在调试窗口也可以找到，不过这里只会显示一部分常用按钮。

![调试按钮](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-65e22daa.png)

调试过程中，使用频率最高的可能就是 **Setp Over**。**Setp Over** 可以单步调试代码，但是当碰到方法执行时，不会进入方法内，直接调用了方法，然后到下一行。

如果需要进入方法内，需要使用  **Step Into** 。**Step Into**  可以进入调用的方法内，不过默认情情况下并不会进入 `java.*` 等类的方法中。如果需要进入，可以在 `Setting-Stepping` 中设置,排除即可。。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-885584bb.png)

另外还可以使用 **Force Step Into**，强制进入。

当碰到一行代码调用多个方法时，使用 **Step Into** 就比较蛋疼了。**Step Into** 会按照调用顺序进入调用的方法，但是往往我们只想进入其中一个方法而已。

这种场景下我们可以使用 **Smart Step Into** ,选择指定方法进入。

![smart-step-into-pup-up-window-new.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/smart-step-into-pup-up-window-new.png)

有些情况下，在进入一个方法后，可以使用 **Setp Out**，快速执行方法，然后跳回到调用处。

另外在进入 **for/while** 循环里面的后，如果不在循环代码外面打断点出来，就需一步步随着循坏代码执行结束，特别麻烦。在这里就可以使用 **Setp Out Of Code Block**，快速跳出循环代码块。

## 断点使用技巧

### 临时断点

一般调试的时候，我们会在代码行使用断点，然后运行测试，等到程序遇到断点暂停时，查看相关变量内容。

在 IDEA 中，默认断点将会一直存在。有时候仅仅想需要暂停第一次，临时查看，这个时候我们可以使用临时断点。使用快捷键 Ctel + Alt + Shift +F8 可以快速创建临时断点或者按住 `ALt`，然后再创建断点。。

### 条件断点

有时候我们在循环处理数据的过程中，可能只关心某个条件的数据，这种情况下我们使用条件断点。

右击断点，弹出设置断点属性的窗口，我们就可以在 **condition** 处设置条件。IDEA 会在满足这个条件时候，才会暂停程序。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-de1a1060.png)

另外设置的条件断点样式也会比较特殊，![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-0adff1b6.png)。

### 多线程断点

在调试多线程的项目时候，如果在多个线程代码处打上了断点，调试的时候你会发现，只能串行的调试。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-424b48f9.png)

上面多线程程序，两个线程启动之后，程序也许会在 `thread1`暂停,也许会在 `thread2`处暂停。只要任一个断线暂停之后，另一处断点线程就会被阻塞。这时只能调试完暂停处线程的断点，才能跳到第二处断点,调试起来非常不方便。

我们可以设置断点线程属性，改变这个特性。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-f06e3abc.png)

我们可以在断点上使用鼠标右键弹出断点设置框，在 **Suspend** 选择 **Thread** 选项。重新启动上面的程序，然后在调试窗口 **Frames** 可以看到 `thread1`，`thread2 `两个线程。其中两个线程均可进入调试，此时程序调试位于 `thread1` 处。这时选择框选择`thread2`，就可以调试 `thread2`.

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-60a31794.png)

另外，断点的多线程属性可以设置默认属性，点击 Make Default 按钮，设置默认 **Suspend**属性。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-e78ce1b4.png)

## 回退调试程序

调试的时候，有时候会进错方法或者错过一个方法调用过程，这个时候往往只能重新开始从头开始执行调试一遍。从新开始调试，可能还需要设置相关参数，修改数据库状态等，非常麻烦。所以如果在调试过程中可以回退就可以不用重新开始。 

IDEA 中可以使用 **Drop Frame** 与 **Force Return**  回退调试程序。

### Drop Frame

**Drop Frame** 字面意思为丢栈。JVM 中使用栈帧用于进行方法调用数据结构，每次方法调用，对应着一个栈帧在虚拟机中从入栈到出栈的过程。调用方法之后，该方法栈帧将会位于栈顶。

执行 **Drop Frame**，丢弃栈顶栈帧，于是程序将会回到调用方法处。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-3c45448d.png)

调试上面的程序，在上面断点处，使用 **Drop Frame**，就可以跳回方法调用处，即第三行。查看输出窗口，发现并没有任何输出。

**Drop Frame**  可以在调试窗口栏找到。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-0b675a73.png)

注意与 **Step Out** 区别，执行 **Step Out**，方法将会执行结束之后再返回到调用处。


### Force Return

**Force Return** 强制返回，可以在程序执行到一半时强制返回结束程序。可以在 **Frames** 找到。

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-c14605bb.png)

![image.png](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20190720/image-7973ba34.png)

点击 **Force Return**，可以设置返回值。





