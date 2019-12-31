---
layout: post
categories: Flink
title: Flink 基础学习(十一) Slot 插槽和 Parallelism 并行度
tags:
  - 惊奇
---

# 1 总结
这次一上来就讲结论吧，在实际应用时，需要注意以下几个要点：

- **`slot` 是静态的概念，表示 `TaskManager` 具有多少并发执行能力。**
- **`parallelism` 是动态的概念，表示程序运行时实际使用时的并发能力。**
- **设置合适的 `parallelism` 可以提高运行效率，大小要适中**
  例如设置了 `slot` 为 4，但设置 `parallelism` 为 1，那么只使用了一个 `slot`，空闲了 3 个，这样也是不可取的。
- **设置 `parallelism` 有多种方式，优先级为 `api -> env -> -p 参数 -> file（flink-conf.yaml）`（从低到高）**
- **设置的 `parallelism` 不能高于 `slot` 数量，不然将会出现计算资源不够用的情况，程序报错。**

<!--more-->

---
# 2 前言

前面一直都在学习的是 `Flink` 的基础使用，基础的套路 `source` --> `transformation` --> `sink`，了解过上面的基础语义和相关 `API`，就能够进行初步的开发，在 `Flink UI` 或者 `IDE` 的控制台就能看到输出结果。

但是对于它的计算资源管理和调度机制还不清晰，所以这次特意查找了相关资料，来学习 `Flink` 是如何对计算资源进行管理的。

详细可以查看英文文档：[Distributed Runtime Environment](https://ci.apache.org/projects/flink/flink-docs-master/concepts/runtime.html)

---
# 3 任务 Task 和算子链 Operator Chain

在分布式计算中，`Flink` 将算子（`operator`）的 `subtask` 链接（`chain`）成 `task`。每个 `task` 由一个线程执行。把算子链接成 `tasks`，也就是 `Operator Chain`， 能够减少线程间切换和缓冲的开销，在降低延迟的同时提高了整体吞吐量。

**下图的 `dataflow` 工作流，`source` 的并行度 `parallelism` 为 2，`map` 算子的并行度为 2，`keyBy()/window()/apply()` 的并行度为 2， `Sink` 算子的并行度为  1，运行在有 2 个 `Slot` 的 `TaskManager` 上。**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/operator_chain.png)

可以看到，`source` 和 `map` 这两个 `operator` 进行了合并，在第二个视图中，每个虚框表示一个子任务，最终使用了 5 个并行的线程来执行。

如果没有开启 `Operator Chain`，那么使用的线程数将会增加到 7，线程数增加后，会增加开销，所以开启算子链是有好处的。

## 3.1 Slot Sharing

继续以上面的的图作为例子，如果在 `slotSharingGroup` 使用默认或者相同组名时，当前 `Job` 运行需要 2 个 `slot` (**小于或等于 `Job` 中单个算子设定的最大 `parallelism`**）

`Slot Sharing`：来自同一个 `Job` 且拥有相同 `slotSharingGroup`（默认：`default`）名称的不同 `Task` 的 `SubTask` 之间可以共享一个 `Slot`，这使得一个 `Slot` 有机会持有 `Job` 的一整条 `Pipeline`，所以默认情况下，所有的 `operatro` 有可能共享同一个 `slot`。

同样可以通过 `api` 来指定特定的算子共享组：

```java
source.map(...).slotSharingGroup("otherGroup");
```

上述例子通过 `slotSharingGroup()` 方法，指定 `map` 算子到 `otherGroup` 分组。

## 3.2 Chain 条件

不是任意两个算子都能够进行链接，链接需要遵守以下原则：

- **上下游的并行度一致**
- **下游节点的入度为 1（也就是说下游节点没有来自其他节点的输入）**
- **上下游节点都在同一个 `slot group` 中**
- **下游节点的 `chain` 策略为 `ALWAYS`（可以与上下游链接，`map`/`flatMap`/`filter` 等默认是 `AWLAYS`）**
- **上游节点的 `chain` 策略为 `ALWAYS` 或 `HEAD`(只能与下游链接，不能与上游链接， `Source` 默认是 `Head`）**
- **两个节点间数据分区方式是 `forward`**
- **用户没有禁用 `chain`**

## 3.3 配置方式

根据优先级，从低到高，依次有以下指定并行度的方式

- **`API` 设定**

在单个算子上，通过调用 `setParallelism()` 方法来指定：

```java
source.setParallelism(2)
      .map(xxxx).setParallelism(2)
      .keyBy(xxxx).window(xxx).apply(xxx)setParallelism(2)
      .sink(xxx).setParallelism(1);
```


- **`Env` 层次**

`Flink` 程序运行在执行环境的上下文中。执行环境为所有执行的算子、数据源、数据接收器 (`data sink`) 定义了一个默认的并行度。可以显式配置算子层次的并行度去覆盖执行环境的并行度。


```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(3);
```

- **`-p` 任务提交设定参数**

在提交作业时，可以通过 `Flink UI` 进行上传 `jar` 包启动，也可以在终端，使用 `-p` 参数指定并行度：

```shell
$./bin/flink run -p 10 .../demo/WordCount.jar
```

- **`flink-conf.yaml` 系统层次**

定位到 `${FLINK_HOME}/conf` 目录，可以通过设置 `flink-conf.yaml` 文件中的 `parallelism.default` 参数

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/parallelism_conf.png)

默认并行度为 1，通过修改该配置，在系统层次来指定所有执行环境的默认并行度（**推荐并行度为服务器的 `CPU` 核数**）

---
# 4 运行环境

要了解 `Slot`，就得熟悉 `Job` 运行的环境，有三个核心组件需要了解：

**`Job Managers`、`Task Managers` 和客户端 `Clients`**

- **Job Managers**：也称为 `master`，协调分布式计算。它们负责调度任务、协调 `checkpoints`、协调故障恢复等
- **Task Managers**：也称为 `worker`，执行 `dataflow` 中的 `tasks` （准确来说是 `subtasks`），并且缓存和交换数据流。
- **Clients**：它不属于运行时（`runtime`）和作业执行时的一部分，但它是被用作准备和提交作业的工具。提交完成之后，客户端可以断开连接，也可以保持连接来接收进度报告。（例如 `CLI` 客户端，通过 `flink run ...` 提交任务）
  

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/flink_run_structure.png)

---
# 5 Task Slot 和 Resources

每个 `worker` （`TaskManager`）是一个 `JVM` 进程，并且可以在单独的线程中执行一个或多个子任务(`subtask`)。为了控制 `worker` 接受多少个任务，`worker` 具有所谓的 `task slot`（至少一个）。

每一个 `Task Slot` 代表了 `TaskManager` 所拥有的计算资源的一个固定的子集。例如，一个拥有 3 个 `slot` 的 `TaskManager`，那么每个 `slot` 可以使用 1/3 的内存。这样，运行在不同 `slot` 中的 `subtask` 不会竞争内存资源。目前 `Flink` 还不支持 `CPU` 的隔离，只支持内存的隔离。

通过调整 `slot` 的数量，用户可以定义子任务如何相互隔离。每个 `TaskManager` 具有一个 `slot`，这意味着每个任务组都在单独的 `JVM` 中运行（例如，可以在单独的容器中启动）。

具有多个 `slot` 意味着允许多个子任务共享一个 `JVM`。可以在同一个 `JVM` 中共享 `TCP`连接（通过多路复用）和心跳消息。他们还可以共享数据集和数据结构，从而减少每个任务的开销。

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/tasks_slots.png)

默认情况下，`Flink` 允许子任务共享 `slot`，即使它们是不同任务的子任务也是如此，只要它们来自同一任务 `job`，并且不是同一个 `operator` 的子任务。通过 `slot sharing` 的结果是一个 `slot` 可以容纳整个作业流水线 `pipeline`。

允许 `slot` 共享有两个主要好处：

- `Flink` 集群所需的 `slot` 与作业中使用的最高并行度 `parallelism` 恰好一样多。无需计算一个程序总共包含多少个 `job` （具有不同的并行度）。
- 更好的资源利用率。如果没有 `slot sharing`，则非密集型 `source / map()`子任务将阻塞与资源密集型窗口子任务一样多的资源。而通过 `slot sharing`，可以充分利用插槽资源，同时确保沉重的 `subtask` 在 `TaskManager` 之间公平分配。

**以一个作业调度为例子：**

一个由 `source`、`MapFunction` 和 `ReduceFunction` 组成的 `Job`，其中数据源和 `MapFunction` 的并行度 `parallelism` 为 4，`ReduceFunction` 的并行度为 3。

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/job_parallelism.jpeg)

在上图中，流水线 `pipeline` 由一系列的 `source` - `Map` - `Reduce` 组成，运行在两个 `TaskManager` 组成的集群上，每个 `TaskManager` 包含 3 个 `slot`。

其中每条线连着的，表示有可能的 `pipeline`，然后在右边显示了哪些 `task` 共享一个 `slot`。

Flink 内部通过 [SlotSharingGroup](https://github.com/apache/flink/blob/master//flink-runtime/src/main/java/org/apache/flink/runtime/jobmanager/scheduler/SlotSharingGroup.java) 和 [CoLocationGroup](https://github.com/apache/flink/blob/master//flink-runtime/src/main/java/org/apache/flink/runtime/jobmanager/scheduler/CoLocationGroup.java) 来定义哪些 `task` 可以共享一个 `slot`， 哪些 `task` 必须严格放到同一个 `slot`。

关于分配的原理和实现，可以参考[云邪写的分配 slot 过程](http://wuchong.me/blog/2016/05/09/flink-internals-understanding-execution-resources/)

---
# 6 Slot 和 Parallelism

回归主题，前面学习了 `slot` 和 `parallelism`，那么将两者联系起来会是怎样呢，可以看下云星数据的博文介绍，用来加深理解：

## 6.1 **Slot 是指 Task Manager 的并发执行能力**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/parallelism1.png)

- 在 `flink-config.yaml` 设置了默认 `slot` 数量为 3
- 图中有三个 `Task Manager`，结合上面的参数，每个 `worker` 分配了 3 个 `slot`，整个集群就有 9 个 `Task Slot`

## 6.2  **Parallelism 是指 Task Manager 实际使用的并发能力**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/parallelism2.png)

- 在配置文件中，设置了默认并行度 `parallelism.default: 1`
- `Example1` 提交的 `Job` 只使用了 1 个并行度，占用 1 个 `slot`，于是还剩下 8 个空闲的 `Task Slot`，设定合适的并行度大小能提升效率和降低空转时间

## 6.3 **Parallelism 是可配置、可指定的**

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/parallelism3.png)

关于并行度的设定方式，前面也提高过的，这里说下图中想要展示信息：

- `Example 2` 设置的并发度为 2，`Example 3` 设置的并发度为 9
- 由于设置的是全局并发度，默认会链接成 `Operator Chain`，然后串成 `pipeline`，每个流水线占有 1 个 `slot`

![](http://www.justdojava.com/assets/images/2019/java/image_yjq/Flink/runtime/parallelism4.png)

- `Example 4` 中，除了 `sink` 算子的并行度为 1，其它算子的并行的都是 9，于是出现了，只能在一个 `slot` 中看到 `sink` 算子
- 其它 `slot` 计算后的结果，将会输送到 `Task Manager 3` 的 `sink` 算子中


---

# 7 小总结

这次再次将 `Flink` 的运行结构和算子执行的工作地 `Task Manager` 熟悉了一下，也了解到多个子任务链接成的 `Operator Chain`，通过算子链，优化了计算资源的利用，减少不必要的开销。


在进一步学习 `slot` 和 `parallelism` 时，参考了很多文章，可以看下参考文献，加深理解。如有其它学习建议或文章不对之处，请与我讨论吧~

---
# 8 项目地址

[https://github.com/Vip-Augus/flink-learning-note](https://github.com/Vip-Augus/flink-learning-note)

```sh
git clone https://github.com/Vip-Augus/flink-learning-note
```

---
# 9 参考文献

1. [Flink 原理与实现：理解 Flink 中的计算资源](http://wuchong.me/blog/2016/05/09/flink-internals-understanding-execution-resources/)
2. [Flink 源码阅读笔记（6）- 计算资源管理](https://blog.jrwang.me/2019/flink-source-code-resource-manager/)
3. [作业调度](https://ci.apache.org/projects/flink/flink-docs-master/zh/internals/job_scheduling.html)
4. [Flink中slot的一点理解](https://blog.csdn.net/a6822342/article/details/77531000)
5. [Flink 从 0 到 1 学习 —— Flink parallelism 和 Slot 介绍](http://www.54tianzhisheng.cn/2019/01/14/Flink-parallelism-slot/)
6. [Slot和Parallelism的深入分析004](https://blog.csdn.net/liguohuaBigdata/article/details/78574888)
7. [并行执行](https://ci.apache.org/projects/flink/flink-docs-master/dev/parallel.html)
8. [Flink Slot 详解与 Job Execution Graph 优化](https://www.infoq.cn/article/ZmL7TCcEchvANY-9jG1H)















