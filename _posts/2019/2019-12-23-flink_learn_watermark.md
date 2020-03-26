---
layout: post
categories: Flink
title: Flink 基础学习(九) 再谈 Watermark
tags:
  - 惊奇
---

# 1 前言

**在时间 `Time` 那一篇中，介绍了三种时间概念 `Event`、`Ingestin` 和 `Process`， 其中还简单介绍了乱序 `Event Time` 事件和它的解决方案 `Watermark` 水位线**

（看过多篇文章后，决定喊它水位线，因为窗口触发条件是 `Watermark` > `Window_end_time`，有点像水流到达水位线后溢出，当然喊它水印也是可以的，全看个人爱好咯~）

前文请翻 [时间 Time 和 Watermark](http://www.justdojava.com/2019/11/27/flink_learn_time/#)，不过前面介绍比较浅，没能很好领会水位线的概念，所以本篇是作为补充，来加深理解~

<!--more-->

---
# 2 Watermark 理论

## 2.1 Watermark 的概念

`Watermark` 是一种衡量 `Event Time` 进展的机制，它是数据本身的隐藏属性。通常基于 `Event Time` 事件的数据，数据自身有一个时间戳 `timestamp` 类型的属性，例如 `Message` 对象中有个属性 `timestamp` = 1575090298299(2019-11-30 13:04:58)，如果设定的可延时时间为 3s，那么从该事件提取到的水位线可能如下：

```java
water(1575090298299) = 1575090298299 - 3000(2019-11-30 13:04:55)
```

如果该水位线被采纳，那么表示全部事件中，`timestamp` 小于 1575090298299 - 3000(2019-11-30 13:04:55)的事件，都已经到达了 `Flink` 程序中。

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/watermark/watermark_ideal.png)

**`Ideal` 表示理想情况下的，事件是按序到达程序的，与处理时间相吻合，但是真实情况下，会有各种其他因素导致事件延迟，造成了 `Reality` 那条线，其中 `Skew` 就是它们倾斜，代表可能延迟到达的时间。**

---
## 2.2 Watermark 的作用

前面提到事件 `Event` 有可能延迟，也就是事件乱序，`Watermark` 就是用来解决该问题的利器，通常搭配 `window` 一起出现。

`watermark` 本质上是一个时间戳，它是单调递增，只要有源源不断的事件到达程序，水位线就有可能被更新，比较替换成更大的值。

然后程序 **根据 `watermark` 识别程序处理到什么位置、进度**，比水位线小的数据表示都已经到达，后续接收的事件时间应该都要比它大。

默认情况下，后续比水位线小的事件会被认为是迟到数据，`Flink` 默认策略是舍弃它们，不进行计算（但还有其它机制去搜集这些被舍弃的迟到数据，详细可去了解 SideOutputLateData)。

**在一些情况下，我们想要延迟窗口几秒才触发计算，例如当前有个时间窗口 [00:01]-[00:04]，但可能在 05s 时，出现了 03 的事件，我们想稍微等一下出现延迟的事件，将 03 这个事件加入到正确的窗口中一起计算**

下面用消息队列来说明下 `watermark`（事件上的数字表示自身携带的 `timestamp`）

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/watermark/message_queue1.png)

上图设定的时间窗口大小为 4s，时间属性为 `Event Time` 事件，消息队列中的数据是乱序到达程序，分割线 `w(4)、w(9)` 表示的是水印（**结合下图分析，延时时间大概率设置为 3s**）

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/watermark/message_queue2.png)

上图例子中，数据 7 进入第二个窗口而水印未生成前，数据 3 进入了第一个窗口后，然后前面水印生成了，全局水印更新成了 `w(4)`，由于 `w(4) >= window1_end_time`，于是触发了第一个窗口的计算。

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/watermark/message_queue3.jpeg)

后续数据 5、6 继续进入第二个窗口，数据 9、12 进入第三个窗口。

接着数据 12 进入程序中，被分配到第三个窗口中，计算得到水印 `w(9)` 大于第二个窗口结束时间，又会触发第二个窗口的计算，以此类推。

从上面看出，数据 3 在数据 7 后面才到，属于迟到数据，但由于设定了 `watermark`，允许了一定时间的迟到事件，所以设定 `watermark` 可以解决一定程度上的乱序事件。

---
## 2.3 算子对 Watermark 的处理

> 引用自 Louisvv 的博文

`Watermark` 是可以被算子处理，算子内部会有个时间记录器，记录各个 `Window` 的结束时间。

当算子收到一个 `Watermark` 时，算子会根据这个 `Watermark` 的时间戳更新内部的 `Event Time Clock`，当前记录的时间与 `Watermark` 进行比较，如果 `Watermark` 大于记录的时间，则会更新该记录为最新的 `Watermark` 值。

---
## 2.4 Watermark 的设定

各种情况导致的事件乱序，我们需要设置 `Watermark`，允许一定时间段的延迟（不过不能无限等待下去，时效性和内存使用率还是很重要的）。

一般会在接收到 `DataSource` 的数据后，立刻生成 `watermark`，也可以在 `source` 之后，进行简单的 `map` 或者 `filter` 操作后，再生成 `watermark`。

`Watermark` 设定方法有两种：

**在代码中，可以通过调用 `DataStream` 中的两个 `API` 来提取时间和分配水印，分别是 `AssignerWithPunctuatedWatermarks` 和 `AssignerWithPeriodicWatermarks`。**

-  **`Punctuated Watermark`：数据流中每一个递增的 `EventTime` 都会产生一个 `Watermark`**

在实际的生产环境中，在 `TPS` 很高的情况下会产生大量的 `Watermark`，可能在一定程度上对下游算子造成一定的压力，所以只有在实时性很高的场景才会选择这种方式来进行生成水印。

- **`PeriodicWatermarks`：周期性（一定时间间隔或者达到一定的记录条数）生成水印**

在实际的生产环境中，使用这种 `PeriodicWatermarks` 较多，它会周期性（通过 setAutoWatermarkInterval(...)，设置间隔的毫秒数）生成水印，但是必须结合时间或者积累条数两个维度，否则会在极端情况下会有很大的延时。

来看下它们在代码中的位置和结构吧：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/watermark/structure_watermarks.png)

下面来结合代码例子看下常用的 `PeriodicWatermarks`

---
# 3 PeriodicWatermarks Demo

先来交代下程序的逻辑：

1、**往 9010 端口发送数据：`nc -l 9010`**
2、**写程序，监听 9010 端口，按行读取，map 分词处理成 `tuple2` 类型（key, time)**
3、**提取时间和生成水印**
4、**`event time` 窗口是 4s，允许延迟时间是 3s**

其中，自定义的 `Watermark` 生成规则实现了 `AssignerWithPeriodicWatermarks`，实现 `getCurrentWatermark`：生成水印 和 `extractTimestamp`：提取时间戳

---
## 3.1 程序主要逻辑

```java
int port = 9010;
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//设置使用eventtime，默认是使用processtime
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
env.setParallelism(1);
DataStream<String> text = env.socketTextStream("127.0.0.1", port, "\n");
//解析输入的数据
DataStream<Tuple2<String, Long>> inputMap = text.map(new MapFunction<String, Tuple2<String, Long>>() {
    @Override
    public Tuple2<String, Long> map(String value) throws Exception {
        String[] arr = value.split(",");
        return new Tuple2<>(arr[0], Long.parseLong(arr[1]));
    }
});
DataStream<Tuple2<String, Long>> waterMarkStream = inputMap.assignTimestampsAndWatermarks(new WordCountPeriodicWatermarks());
DataStream<String> window = waterMarkStream.keyBy(0)
        //按照消息的EventTime分配窗口，和调用TimeWindow效果一样
        .window(TumblingEventTimeWindows.of(Time.seconds(4)))
        .apply(new WindowFunction<Tuple2<String, Long>, String, Tuple, TimeWindow>() {
            @Override
            public void apply(Tuple tuple, TimeWindow window, Iterable<Tuple2<String, Long>> input, Collector<String> out) throws Exception {
                String key = tuple.toString();
                List<Long> arrarList = new ArrayList<>();
                List<String> eventTimeList = new ArrayList<>();
                Iterator<Tuple2<String, Long>> it = input.iterator();
                while (it.hasNext()) {
                    Tuple2<String, Long> next = it.next();
                    arrarList.add(next.f1);
                    eventTimeList.add(String.valueOf(next.f1).substring(8,10));
                }
                Collections.sort(arrarList);
                SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
                String result = "\n 键值 : " + key + "\n              " +
                        "触发窗内数据个数 : " + arrarList.size() + "\n              " +
                        "触发窗起始数据： " + sdf.format(arrarList.get(0)) + "\n              " +
                        "触发窗最后（可能是延时）数据：" + sdf.format(arrarList.get(arrarList.size() - 1))
                        + "\n              " +
                        "窗口内的事件数据：" + Joiner.on(",").join(eventTimeList) + "\n" +
                        "实际窗起始和结束时间： " + sdf.format(window.getStart()) + "《----》" + sdf.format(window.getEnd()) + " \n \n ";
                out.collect(result);
            }
        });
window.print();
env.execute("eventtime-watermark");
```

上面贴出来的代码作用就是接收 9010 端口写入的数据，然后进行 `map` 提取处理，接着进行水印处理，最后进入窗口运算符，进行窗口计算。

---
## 3.2 水印生成器

```java
public class WordCountPeriodicWatermarks implements AssignerWithPeriodicWatermarks<Tuple2<String, Long>> {
    private Long currentMaxTimestamp = 0L;
    // 最大允许的乱序时间是 3 s
    private final Long maxOutOfOrderness = 3000L;
    private SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
    @Override
    public Watermark getCurrentWatermark() {
        return new Watermark(currentMaxTimestamp - maxOutOfOrderness);
    }
    @Override
    public long extractTimestamp(Tuple2<String, Long> element, long previousElementTimestamp) {
        //定义如何提取timestamp
        long timestamp = element.f1;
        currentMaxTimestamp = Math.max(timestamp, currentMaxTimestamp);
        long id = Thread.currentThread().getId();
        System.out.println("线程 ID ："+ id +
                " 键值 :" + element.f0 +
                ",事件事件:[ "+sdf.format(element.f1)+
                " ],currentMaxTimestamp:[ "+
                sdf.format(currentMaxTimestamp)+" ],水印时间:[ "+
                sdf.format(getCurrentWatermark().getTimestamp())+" ]");
        return timestamp;
    }
}
```

`extractTimestamp()` 方法从数据本身属性中提取 `Event Time`，该方法返回的是 `Math.max(timestamp, currentMaxTimestamp)`，比较了当前时间戳和事件时间戳，返回较大者。

**`getCurrentWatermark` 是获取当前的水印，这里定义的最大延迟时间为 3s，生成的水印会减去它，例如数据 7 进来后，水印计算规则：`w(7 - 3) = w(4)`。后面如果窗口计算触发后，超过水印时间的事件，默认情况下会被舍弃掉。**

---
## 3.3 运行程序和测试数据

记住运行程序前，需要在终端中打开 9010 端口：`nc -l 9010`

**测试数据：2 3 1 7 3 5 9 6 12 17 10 16 19 11 18**
第一列为 `key`，为了便于辨认，都是用 001，第二列是时间戳，按照上面测试数据，设定成对应的秒数
```shell
001,1575129602000
001,1575129603000
001,1575129601000
001,1575129607000
001,1575129603000
001,1575129605000
001,1575129609000
001,1575129606000
001,1575129612000
001,1575129617000
001,1575129610000
001,1575129616000
001,1575129619000
001,1575129611000
001,1575129618000
```

---
## 3.4 理想中的验证结果

在第一次试验中，我将这批数据全量拷贝到终端，程序进行了处理：

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/watermark/batch_processing.jpg)

整理的输出结果如下：

| Event Time              | CurrentMaxTimeStamp     | Watermark               | Window_Start_Time | Window_End_Time |
| ----------------------- | ----------------------- | ----------------------- | ----------------- | --------------- |
| 00:00:02  | 00:00:02  | 23:59:59  | 00:00:00       | 00:00:04     |
| 00:00:03             | 00:00:03             | 00:00:00  |                   |                 |
| 00:00:01             | 00:00:01             | 00:00:00             |                   |                 |
| 00:00:07             | 00:00:07             | 00:00:04             | 00:00:04       | 00:00:08     |
| 00:00:03             | 00:00:03             | 00:00:04             | 00:00:00       | 00:00:04     |
| 00:00:05             | 00:00:05             | 00:00:04             | 00:00:04       | 00:00:08     |
| 00:00:09             | 00:00:09             | 00:00:06             | 00:00:08       | 00:00:12     |
| 00:00:06             | 00:00:06             | 00:00:06             | 00:00:04       | 00:00:08     |
| 00:00:12             | 00:00:12             | 00:00:09             | 00:00:12       | 00:00:16     |
| 00:00:17             | 00:00:17             | 00:00:14             | 00:00:16       | 00:00:20     |
| 00:00:10             | 00:00:17             | 00:00:14             |                   |                 |
| 00:00:16             | 00:00:17             | 00:00:14             |                   |                 |
| 00:00:19             | 00:00:19             | 00:00:16             |                   |                 |
| 00:00:11             | 00:00:19             | 00:00:16             |                   |                 |
| 00:00:18             | 00:00:19             | 00:00:16             |                   |                 |

**我们关注一下第一个窗口的例子，数据 3 在数据 7 之后才进来，但还是正确进入到了第一个窗口中运算，表示设定了 `Watermark` 后，能够解决乱序事件。**

---
## 3.5 留下一个疑问

**同样的测试数据，前面的是批量拷贝过去的，程序和水印 `Watermark` 正常起了作用，但是在单条数据，一条一条发送的情况下，出现了另一种窗口触发情况：**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/watermark/single_processing.png)

也是关注第一个窗口，前面例子中，输出的结果是窗口数据有四个，本次只有三个，我们下一条数据 3 被舍弃了，这个情况我无法解释清楚，也算留个坑，等之后再去处理。

---
## 3.6 Watermark 和 Window 结合
从上面的例子可以看出，使用水印时，需要设置延迟时间 `maxOutOfOrderness`，如果设置过大的话，容易造成程序中出现多个窗口，一直在等待延迟数据，然后窗口一直不被触发。

服务器内存使用过多容易导致 `OOM` 不说，而且窗口不被触发计算，会造成统计数据的实时性变差，影响业务输出。

**按照 `zhisheng` 的建议，在之后的使用中，要注意以下两点：**

- **合理的设置 `maxOutOfOrderness`，避免过大**
- **不太依赖 `Event Time` 的场景就不要设置时间策略为 `EventTime`**

---
# 4 延迟数据如何处理

## 4.1 默认策略：丢弃

这个是默认处理方式，从前面的例子中也能看到，数据 10 在数据 17 后面进入，此时水印比 10 要大，所以数据 10 所处的窗口 [08 <--> 12] 已经被触发，于是数据 10 被舍弃，不参与计算

## 4.2 剩下的内容

- **allowedLateness  再次指定允许数据延迟的时间**
- **sideOutputLateData 搜集迟到的数据**

前面两个真正深入去学，估计也是一篇大学问，所以推荐给大家看别人写的，之后我学到再跟大家分享吧：

- [Flink流计算编程--Flink中allowedLateness详细介绍及思考](https://blog.csdn.net/lmalds/article/details/55259718)

---
# 5 总结

- Flink 如何处理乱序

使用 `Watermark` + `Window` 机制

- Flink 何时触发 Window

普通设定，没有设定 `allowedLateness` 更多延迟处理时间

> Watermark >= Event Time 

关于 `allowedLateness`，请看参考资料 4

- Flink 使用 Watermark 的建议

> **1. 合理的设置 `maxOutOfOrderness`，避免过大**
> **2. 不太依赖 `Event Time` 的场景就不要设置时间策略为 `EventTime`**


本篇开门见山，直接介绍了 `Watermark` 水印的概念，因为乱序事件的情况，`Flink` 设计了水印用来处理延迟事件。介绍了两种生成水印的方法 `Periodic`：周期性 和 `Punctuated`：按次递增生成，以常用的 `Periodic` 水印 作为例子，通过代码和测试数据验证了延迟事件被正确处理了。

在进一步学习 `Watermark` 时，参考了很多文章，大致了解了它的概念和使用，不过还存在一些疑问和坑，希望各位有了解的可与我分享。如有其它学习建议或文章不对之处，请与我讨论吧~

---
# 6 项目地址

[https://github.com/Vip-Augus/flink-learning-note](https://github.com/Vip-Augus/flink-learning-note)

```sh
git clone https://github.com/Vip-Augus/flink-learning-note
```

---
# 7 参考资料

1. [Flink Window分析及Watermark解决乱序数据机制深入剖析-Flink牛刀小试](https://juejin.im/post/5bf95810e51d452d705fef33)
2. [Flink流计算编程--watermark（水位线）简介](https://blog.csdn.net/lmalds/article/details/52704170)
3. [Flink WaterMark简介](http://www.louisvv.com/archives/2189.html)
4. [Flink流计算编程--Flink中allowedLateness详细介绍及思考](https://blog.csdn.net/lmalds/article/details/55259718)










