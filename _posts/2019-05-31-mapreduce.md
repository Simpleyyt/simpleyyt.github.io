---
layout: post
category: BigData
title: MapReduce 编程模型 & WordCount 示例
tagline: by 乔二爷
tags: 
  - bigdata
  
published: true
---

学习大数据接触到的第一个编程思想 MapReduce。

<!--more-->

## 前言
之前在学习大数据的时候，很多东西很零散的做了一些笔记，但是都没有好好去整理它们，这篇文章也是对之前的笔记的整理，或者叫输出吧。一来是加深自己的理解，二来是希望这些东西能帮助想要学习大数据或者说正在学习大数据的朋友。如果你看到里面的东西，让你知道了它，这也是一种进步嘛。说不定就开启了你的另一扇大门呢？

## 先来看一个问题
在讲 MapReduce 之前，我们先来看一个问题。我们都知道，在大数据场景中，最先让人了解到的就是数据量大。当数据量大了以后，我们在处理一些问题的时候，可能就没办法按照以前我们传统的方式去解决问题。

我们以一个简单的单词计数来看这个问题。

比如现在我们有一个文件，就10M，里面存放的是一篇英文文档，我们现在的需求就是计算单词出现的次数。

按照我们以前写 Java 代码的套路来做，大概就是读取文件，把数据加载到内存，然后new 一个map来存最后的结果。key 就是单词，value 就是单词出现的次数。

然后从文件中读取一行数据，然后对这行数据按空格进行切割，然后对切割后的一个一个的单词进行处理，看map 中是否存在，存在就 value + 1,不存在就设置 value 为 1 。

然后再读取一行数据重复上面的操作，直到结束。很简单吧。

是的，没问题，刚才文件是 10M，处理完成秒秒钟的事情，但是现在我的文件是 2T 的大小，看清楚呃，是两个 T 的文件需要处理，那你现在要怎么做？还去加载到内存么？

想想你公司的机器配置，内存多大，8G，16G，32G ...,顶起天 128G 吧。先不说多大，再想想现在内存价格是多少，128G 的内存得花多少钱。很显然，现在这么玩儿，玩不了吧。

但是，现在一般你公司的机器都还是有不少台吧。那么如果说我们现在把这些机器组成一个 N 节点的集群，然后把这 2T 的文件切分成很多个小文件，然后丢到这些机器上面去计算执行统计，最后再进行一个汇总，是不是就解决了上面的内存不足的问题。

## MapReduce 思想

MapReduce 是一种编程模型，用于大规模数据集（大于1TB）的并行运算，源于 Google 一篇论文，它充分借鉴了 “分而治之” 的思想，将一个数据处理过程拆分为主要的Map(映射)与Reduce(化简)两步。

对比上面的例子来说，Map 阶段就是每个机器处理切好的数据片的阶段，Reduce 阶段则是最后统计汇总的阶段。

那么，针对前面说的例子大概可以用下面这个图来描述它：

![](http://www.justdojava.com/assets/images/2019/java/image_qry/20190530-mapreduce/1.png)

简单说一下上面的思路：

第一步：把两个T 的文件分成若干个文件块（block）分散存在整个集群上，比如128M 一个。

第二步：在每台机器上运行一个map task 任务，分别对自己机器上的文件进行统计：

1. 先把数据加载进内存，然后一行一行的对数据进行读取，按照空格来进行切割。
2. 用一个 HashMap 来存储数据，内容为 <单词，数量>
3. 当自己本地的数据处理完成以后，将数据进行输出准备
4. 输出数据之前，先把HashMap 按照首字母范围分成 3 个HashMap 
5. 将3个 HashMap 分别发送给 3个 Reduce task 进行处理，分发的时候，同一段单词的数据，就会进入同一个 Reduce task 进行处理，保证数据统计的完整性。

第三步： Reduce task 把收到的数据进行汇总，然后输出到 hdfs 文件系统进程存储。

## 上面的过程可能遇到的问题

上面我们只是关心了我们业务逻辑的实现，其实系统一旦做成分布式以后，会面临非常多的复杂问题，比如：

* 你的 Map task 如何进行任务分配？
* 你的 Reduce task 如何分配要处理的数据任务？
* Map task 和 Reduce task 之间如何进行衔接，什么时候去启动Reduce Task 呀？
* 如果 Map task 运行失败了，怎么处理？
* Map task 还要去维护自己要发送的数据分区，是不是也太麻烦了。
* 等等等等等

## 为什么要用 MapReduce 

可见在程序由单机版扩成分布式时，会引入大量的复杂工作。为了提高开发效率，可以将分布式程序中的公共功能封装成框架，让开发人员可以将精力集中于业务逻辑。

而 MapReduce 就是这样一个分布式程序的通用框架。

## WordCount 示例

用一个代码示例来演示，它需要3个东西，一个是map task ，一个是 reduce task ，还有就是启动类，不然怎么关联他们的关系呢。

首先是 map task :

```
package com.zhouq.mr;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import java.io.IOException;

/**
 * KEYIN 默认情况下，是MR 框架中读取到的一行文本的起始偏移量，long 类型
 * 在hadoop 中有自己更精简的序列化接口，我们不直接用Long ，而是用 LongWritable
 * VALUEIN : 默认情况下，是MR 中读取到的一行文本内容，String ，也有自己的类型 Text 类型
 * <p>
 * KEYOUT ： 是用户自定义的逻辑处理完成后的自定义输出数据的key ,我们这里是单词，类型为string 同上，Text
 * <p>
 * VALUEOUT： 是用户自定义的逻辑处理完成后的自定义输出value 类型，我们这里是单词数量Integer,同上，Integer 也有自己的类型 IntWritable
 * <p>
 */
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    /**
     * map 阶段的业务逻辑就写在map 方法内
     * maptask 会对每一行输入数据 就调用一次我们自定义的map 方法。
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //拿到输入的这行数据
        String line = value.toString();
        //根据空格进行分割得到这行的单词
        String[] words = line.split(" ");
        //将单词输出为 <word,1>
        for (String word : words) {
            //将单词作为key ，将次数 做为value输出，
            // 这样也利于后面的数据分发，可以根据单词进行分发，
            // 以便于相同的单词落到相同的reduce task 上,方便统计
            context.write(new Text(word), new IntWritable(1));
        }
    }
}
```

接下来是 reduce task 逻辑：

```
/**
 * KEYIN VALUEIN 对于map 阶段输出的KEYOUT VALUEOUT
 * <p>
 * KEYOUT :是自定义 reduce 逻辑处理结果的key
 * VALUEOUT : 是自定义reduce 逻辑处理结果的 value
 */
public class WordcountReduce extends Reducer<Text, IntWritable, Text, IntWritable> {
    /**
     * <zhouq,1>,<zhouq,1>,<zhouq,2> ......
     * 入参key 是一组单词的kv对 的 key
     */
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        //拿到当前传送进来的 单词
//        String word = key.toString();
        //
        int count = 0;
        for (IntWritable value : values) {
            count += value.get();
        }
        //这里的key  就是单词
        context.write(key, new IntWritable(count));
    }
}
```

最后是启动类：

```
/**
 * wc 启动类
 */
public class WordCountDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        // mapreduce.framework.name 配置成 local 就是本地运行模式,默认就是local
        // 所谓的集群运行模式 yarn ,就是提交程序到yarn 上. 要想集群运行必须指定下面三个配置.
//        conf.set("mapreduce.framework.name", "yarn");
//        conf.set("yarn.resoucemanager.hostname", "mini1");
        //conf.set("fs.defaultFS","com.zhouq.hdfs://mini1:9000/");

        Job job = Job.getInstance(conf);

        //指定本程序的jar 包 所在的本地路径
        job.setJarByClass(WordCountDriver.class);

        //指定本次业务的mepper 和 reduce 业务类
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordcountReduce.class);

        //指定mapper 输出的 key  value 类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        
        //指定 最终输出的 kv  类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //指定job的输入原始文件所在目录
        FileInputFormat.setInputPaths(job,new Path(args[0]));
        //指定job 输出的文件目录
        FileOutputFormat.setOutputPath(job,new Path(args[1]));

        boolean waitForCompletion = job.waitForCompletion(true);
        System.exit(waitForCompletion ? 0 : 1);
    }
}

```

配置启动类参数：填写输入目录和输出目录，注意**输出目录不能存在**，不然会执行失败的。

![](http://www.justdojava.com/assets/images/2019/java/image_qry/20190530-mapreduce/2.png)

执行我们就用编辑器执行，用本地模式，不提交到hadoop 集群上，执行完成后，去到输出目录下可以看到这些文件：

![](http://www.justdojava.com/assets/images/2019/java/image_qry/20190530-mapreduce/3.png)

然后输出一下 part-r-00000 这个文件：

![](http://www.justdojava.com/assets/images/2019/java/image_qry/20190530-mapreduce/4.png)

代码地址：[https://github.com/heyxyw/bigdata/blob/master/bigdatastudy/mapreduce/src/main/java/com/zhouq/mr/WordCountDriver.java](https://github.com/heyxyw/bigdata/blob/master/bigdatastudy/mapreduce/src/main/java/com/zhouq/mr/WordCountDriver.java)

## 最后

希望对你有帮助。后面将会去讲 MapReduce 是如何去运行的。
