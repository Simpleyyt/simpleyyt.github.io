---
layout: post
categories: MQ
title: 还不知道事务消息吗？这篇文章带你全面扫盲！
tagline: by 小黑
published: false
tags: 
  - 小黑
---

阿粉最近碰到个一个需求，正好需要使用 RocketMQ 事务消息，所以阿粉专门学习了一下。

> 文末有彩蛋，看完再走

<!--more-->

## 为什么需要事务消息？

很多同学可能不知道事务消息是什么，没关系，阿粉举一个真实业务场景，先来带你了解一下普通的消息存在问题。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcw1scwm91j31lo0lqdj7.jpg)

上面业务场景中，当用户支付成功，将会更新支付订单，然后发送 MQ 消息。手续费系统将会通过拉取消息，计算手续费然后保存到另外一个手续费数据库中。

由于计算手续费这个步骤可以离线计算，所以这里采用 MQ 上面支付与计算手续费的流程。

上面的流程主要涉及三个步骤：

- 更新订单数据
- 发送消息给 MQ
- 手续费系统拉取消息

上面提到的步骤，任何一个都会失败，如果我们没有处理，就会使两边数据不一致，将会造成下面两种情况：

- 订单数据更新了，手续费数据没有生成
- 手续费数据生成，订单数据却没有更新

这可是涉及到真正的钱，一旦少计算，就会造成资损。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcx7w5ba8zj305k04n744.jpg)

对于最后一步来讲，比较简单。如果消费消息失败，只要没有提交消息确认，MQ 服务端将会自动重试。

这里**最大的问题**就是我们无法保证更新操作与发送消息一致性。无论我们采用先更新订单数据，再发送消息，还是先发送消息，再更新订单数据，都在存在一个成功，一个失败的可能。

如下所示，我们先发送消息，然后再更新数据库。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcw27tz54qj30k20a4wf1.jpg)

上面流程消息发送成功之后，再进行本地事务的提交。这个流程看起来很完美，但是想象一下，如果在提交事务时数据库执行失败，导致事务回滚了。

然而此时消息已经发送出去，无法撤回。这就导致手续费系统紧接会消费消息，计算手续费并更新到数据库中。这就造成支付数据未更新，手续费系统却生成的不一致的情况。

那如果我们流程反一下，是不是就好了呢？

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcw27y4c5qj30k20a40ta.jpg)

我们使用下面的伪码表示：

```java
// 开始事务
try {
    // 1.执行数据库操作
    // 2.提交事务
}catch (Exception e){
    // 3.回滚事务
}
// 4.发送 mq 消息
```

这里如果事务提交成功，但是 mq 消息发送失败，就会导致支付数据更新但是手续费数据未生成的的不一致情况。

这里有的同学可能会想到，将发送 mq 消息步骤移动到事务中，消息发送失败，回滚事务，不就完美了吗？

伪码如下：

```java
// 开始事务
try {
  // 1.执行数据库操作
  // 2.发送 mq 消息
  // 3.提交事务
}catch (Exception e){
  // 4.回滚事务
}
```

上面代码看起来确实没什么问题，消息发送失败，回滚事务。

但是实际上第二步有可能存在消息已经发送到 MQ 服务端，但是由于网络问题未及时收到 MQ 的响应消息，从而导致消息发送失败。

这就会导致订单事务回滚了，但是手续费系统却能消费消息，两边数据库又不一致了。

熟悉 MQ 的同学，可能会想到，消息发送失败，可以重试啊。

是的，我们可以增加重试次数，重新发送消息。但是这里我们需要注意，由于消息发送耦合在事务中，过多的重试会拉长数据库事务执行时间，事务处理时间过长，导致事务中锁的持有时间变长，影响整体的数据库吞吐量

实际业务中，不太建议将消息发送耦合在事务中。

## 事务消息

事务消息是 RocketMQ 提供的事务功能，可以实现分布式事务，从而保证上面事务操作与消息发送要么都成功，要么都失败。

使用事务消息，整体流程如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcw334m383j30k20a4jsm.jpg)

首先我们将会发送一个半(**half**) 消息到 MQ 中，通知其开启一个事务。这里半消息并不是说消息内容不完整，实际上它包含所有完整的消息内容。

这个半消息与普通的消息唯一的区别在于，在事物提交之前，这个消息对消费者来说是**不可见**的，消费者不会消费这个消息。

一旦半消息发送成功，我们就可以执行数据库事务。然后根据事务的执行结果再决定提交或回滚事务消息。

如果事务提交成功，将会发送确认消息至 MQ，手续费系统就可以成功消费到这条消息。

如果事务呗回滚，将会发送回滚通知至 MQ，然后 MQ 将会删除这条消息。对于手续费系统来说，都不会知道这条消息的存在。

这就解决了要么都成功，要么都失败的一致性要求。

实际上面的流程还是存在问题，如果我们**提交/回滚**事务消息失败怎么办？

对于这个问题，RocketMQ 给出一种事务反查的机制。我们需要需要注册一个回调接口，用于反查本地事务状态。

RocketMQ 若未收到提交或回滚的请求，将会定期去反查回调接口，然后可以根据反查结果决定回滚还是提交事务。

RocketMQ 事务消息流程整体如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcw40gmvljj30qg09e0tq.jpg)

事务消息示例代码如下：

```java
public class TransactionMQProducerExample {
    public static void main(String[] args) throws MQClientException, InterruptedException, UnsupportedEncodingException {
        TransactionMQProducer producer = new TransactionMQProducer("test_transaction_producer");
        // 不定义将会使用默认的
        ExecutorService executorService =
                new ThreadPoolExecutor(2, 5, 100,
                        TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2000), 
                                       new ThreadFactory() {
                    @Override
                    public Thread newThread(Runnable r) {
                        Thread thread = new Thread(r);
                        thread.setName("client-transaction-msg-check-thread");
                        return thread;
                    }
                });
        producer.setExecutorService(executorService);
        TransactionListener transactionListener = new TransactionListenerImpl();
        producer.setTransactionListener(transactionListener);
        // 改成自己的地址
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();

        Order order = new Order("66666", "books");

        Message msg =
                new Message("transaction_tp",
                        JSON.toJSONString(order).getBytes(RemotingHelper.DEFAULT_CHARSET));
        // 发送半消息
        SendResult sendResult = producer.sendMessageInTransaction(msg, null);
        System.out.println(sendResult.getSendStatus());
        producer.shutdown();
    }

    public static class TransactionListenerImpl implements TransactionListener {

        /**
         * 半消息发送成功将会自动执行该逻辑
         *
         * @param msg
         * @param arg
         * @return
         */
        @Override
        public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
            // 执行本地事务
            Order order = null;
            try {
                order = JSON.parseObject(new String(msg.getBody(),
                        RemotingHelper.DEFAULT_CHARSET), Order.class);
                boolean isSuccess = updateOrder(order);
                if (isSuccess) {
                    // 本地事务执行成功，提交半消息
                    System.out.println("本地事务执行成功，提交事务事务消息");
                    return LocalTransactionState.COMMIT_MESSAGE;
                } else {
                    // 本地事务执行成功，回滚半消息
                    System.out.println("本地事务执行失败，回滚事务消息");
                    return LocalTransactionState.ROLLBACK_MESSAGE;
                }
            } catch (Exception e) {
                System.out.println("本地事务执行异常");
            }
            // 异常情况返回未知状态
            return LocalTransactionState.UNKNOW;
        }

        /**
         * 更新订单
         * 这里模拟数据库更新，返回事务执行成功
         *
         * @param order
         * @return
         */
        private boolean updateOrder(Order order) throws InterruptedException {
            TimeUnit.SECONDS.sleep(1);
            return true;
        }

        /***
         * 若提交/回滚事务消息失败，rocketmq 自动反查事务状态
         * @param msg
         * @return
         */
        @Override
        public LocalTransactionState checkLocalTransaction(MessageExt msg) {
            try {
                Order order = JSON.parseObject(new String(msg.getBody(),
                        RemotingHelper.DEFAULT_CHARSET), Order.class);
                boolean isSuccess = queryOrder(order.getOrderId());
                if (isSuccess) {
                    // 本地事务执行成功，提交半消息
                    return LocalTransactionState.COMMIT_MESSAGE;
                } else {
                    // 本地事务执行成功，回滚半消息
                    return LocalTransactionState.ROLLBACK_MESSAGE;
                }

            } catch (Exception e) {
                System.out.println("查询失败");
            }
            // 异常情况返回未知状态
            return LocalTransactionState.UNKNOW;
        }

        /**
         * 查询订单状态
         * 模拟返回查询成功
         *
         * @param orderId
         * @return
         */
        private boolean queryOrder(String orderId) throws InterruptedException {
            TimeUnit.SECONDS.sleep(1);
            return true;
        }
    }

    @Data
    public static class Order {
        private String orderId;

        private String goods;

        public Order(String orderId, String goods) {
            this.orderId = orderId;
            this.goods = goods;
        }
    }
}
```

上面代码中：

1. 我们需要为生产者指定一个**唯一**的 `ProducerGroup`
2. 需要继承 `TransactionListener` 注解回调接口，其中 `executeLocalTransaction` 方法执行本地事务，`checkLocalTranscation` 用来执行检查本地事务。
3. 返回事务状态有三种：
   - LocalTransactionState.UNKNOW 中间状态，RocketMQ 将会反查
   - LocalTransactionState.COMMIT_MESSAGE 提交事务，消息这后续将会消费这条消息
   - LocalTransactionState.ROLLBACK_MESSAGE，回滚事务，RocketMQ 将会删除这条消息

### 事务消息使用注意点

**事务消息最大反查次数**

由于单个消息反查次数过多，将会导致半消息队列堆积，影响性能。 RocketMQ 默认将单个消息的检查次数限制为 15 次。

我们可以通过修改 `broker` 配置文件，增加如下配置：

```conf
# N 为最大检查次数
transactionCheckMax=N
```

当检查次数超过最大次数后，RocketMQ 将会丢弃消息并且打印错误日志。

若想自定义丢弃消息行为，需要修改 RocketMQ broker 端代码，继承 `AbstractTransactionalMessageCheckListener` 重写 `resolveDiscardMsg` 方法，加入自定义逻辑。

**同步的双重写入机制**

为了确保事务消息不丢失，并且保证事务完整性，建议使用同步双重写入机制。

**事务反查时间设置**

我们可以设置以下参数，设置 broker 多久之后开始反查事务消息（自事务消息保存成功之后开始计算）。

```java
msg.putUserProperty(MessageConst.PROPERTY_CHECK_IMMUNITY_TIME_IN_SECONDS, "10");
```

或者我们可以在 `broker.conf` 设置以下参数：

```conf
# 单位为 ms,默认为 6 s
transactionTimeout=60000
```

发送端主动设置配置参数优先级大于 `broker` 端配置。

另外 RocketMQ 还有一个配置用于控制事务性消息检查间隔：

```conf
## 默认为 60s
transactionCheckInterval=5000
```

如果自定义配置如上,事务消息检查间隔为 5 秒，事务消息设置检查时间为 60 s。

这就代表 broker 每隔 5s 检查一次事务消息，如果此时事务消息到 broker 时间还未超过 60s，此时将不会反查，直到时间大于 60s。


## 彩蛋

阿粉在查找事务消息资料的时候，发现 RocketMQ 文档存在相关错误。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcx9sz8tcwj31iw0nyqdy.jpg)

> 文档地址：https://github.com/9526xu/rocketmq/blob/master/docs/cn/RocketMQ_Example.md

如上两处实际是错误的，应该修改为：`AbstractTransactionalMessageCheckListener` 与 `transactionTimeout`。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcx9uw0qftj31960c00v6.jpg)

> issue 地址：https://github.com/apache/rocketmq/issues/481

阿粉顺手修改了一下，提交 **PR** 。哈哈，阿粉也为开源项目贡献了一份力量。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200318/00831rSTly1gcx9xfqh3hj305k05k0sn.jpg)

## Reference

1. https://github.com/apache/rocketmq/issues/481
2. https://github.com/9526xu/rocketmq/blob/master/docs/cn/RocketMQ_Example.md