# Kafka Consumer
Consumer api 澄清
---------------

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eed607c4193b484090ad1704bb945a84~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
 在 **kafka 0.9.0** 版本之前，**Consumer** 有两套 **API** ，一套高级 **API**，一套低级 **API**

[高级 Consumer API 和低级 Consumer API](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fbeipiaodiaosi%2Fp%2F11311507.html "https://www.cnblogs.com/beipiaodiaosi/p/11311507.html")

但从 **kafka 0.9.0** 版本开始提供了新的 **Consumer API** ，而从 **kafka 0.9.0** 版本到现在 **Consumer API** 又发生了很多新变化，学习的时候注意辨别，防止还在学习一些已经过时、废弃的知识

例如上面的链接，虽然总结的非常好，但实际已经老掉牙了，加上文章发表日期是 2019年8月 ，会让人误以为这是较新的知识点，其实我就被误导过，但我在学习的过程中非常关心版本相关的问题，所有不久就发现了它是过时的知识

Consumer
--------

**Consumer** 是 **kafka** 的消费端，用于消费 **Topic** 中的消息，具体来说是消费 **Topic** 的 **partition** 的消息，再进一步说，**Consumer** 消费的是 **Partition** 中 **Leader Replication** 的消息

> 一个 **Topic** 由多个 **Partition** 组成，一个 **Partition** 由一个 **Leader Replication** 和多个 **Follower Replication** 组成

**Consumer** 的工作方式是向它想要消费的分区的 **broker** 发出 `fetch` 请求。**Consumer** 在每个请求中指定想要消费的消息，在日志中的偏移量，并接收从该位置开始的日志块。因此，消费者对这个位置有很大的控制权，如果需要，甚至可以倒带重新消费

push 还是 pull
------------

一般，消息中间件的消息有两种消费方式

*   **Push** : **server** 端主动推送数据给 **consumer** 端
*   **Pull** : **consumer** 端主动拉取 **server** 端的消息

这两种方式各有优劣，如果是选择 **Push** 的方式最大的问题在于 **server** 端不清楚 **consumer** 端的消费速度，可能会导致 **consumer** 端消费不过来。

而采用 **Pull** 的方式则可以解决这种情况， **consumer** 端可以根据自己的状态来拉取数据，可以对 **server** 端的数据进行延迟处理。但是这种方式也有一个劣势就是 **server** 端没有数据的时候可能会一直轮询

**Kafka Consumer** 采用的是 **Pull** 的方式，默认每隔 100ms 主动拉取一次消息，对于 **server** 端没有数据的时候可能会一直轮询的问题，**Kafka** 允许消费者请求在 “长轮询” 中阻塞，等待数据到达 (并且可选地等待直到给定数量的字节可用以确保传输大小)。

Consumer Group
--------------

### Consumer Group 是什么

**Consumer Group** 是 **Consumer** 的分组，从 **Kafka 2.2.1** 开始，**Consumer** 必须显式指定消费者组id, 否则将无法订阅 **Topic** 和提交 **offset**

### 为什么需要 Consumer Group

主要出于两点

1.  将多个 **Consumer** 组织在一起，并发消费同一个 **topic** ，提供消费性能
2.  实现消息广播

### 并发消费

先说第一点，我们知道一个在 **kafka** 中一个 **Topic** 由多个 **Partition** 组成，如果只有一个 **Consumer** 那么该 **topic** 下的所有 **Partition** 都由它来消费。

如果消息的生产速度特别快，一个 **Consumer** 可能消费不过来，消息就会在消息队列堆积，长时间可能会将消息队列压垮，怎么办？一个 **Consumer** 消费不过来，我就用多个 **Consumer** 一起来消费这一个 **topic**。

> 这也是面试中常问的，消息堆积该怎么处理的答案 —— 增加消费者

例如，下面的 Topic 1 有三个 **partition**，我们启用 3 个 **Consumer** 去同时消费，每个

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6036d201be5b4afcbef2cd8bbeb78448~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**Consumer** 只消费一个 **partition** ，这种由多个 **Consumer** 来共同消费一个 **Topic** 的组织形式，就被称为 **Consumer Group**

注意 **Consumer** 并不是越多越好，在一个**Consumer Group** 中，如果一个 **Topic** 的消费者数量，超过了该 **Topic** 的 **Partition** 数量，那么多出来的 **Consumer** 是消费不到数据的

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c05ab86e36464e81917fde7d847a10ea~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

> **kafka** 通过 **Consumer Group** 的并发消费的设计，大大提高了 **Consumer** 客户端的消费性能

### 消息广播

有时候 Topic 的一条消息，需要被多方消费。比如订单的 Topic，广告服务可能需要分析用户的喜好，实现精准推送，而反欺诈服务，可能会判别该订单是否存在欺诈行为等

当然，以上需求也能实现，把他们都放在一个消费流程里就行，但 **kafka** 为我们 提供了更加优雅的实现，**消息广播** 。当有消息到达时，**kafka** 会将消息广播给所有订阅了该 **Topic** 的 **Consumer Group**

我们可以定义多个 **Consumer Group** ，每一个 **Consumer Group** 就是一个消费方，他们之间彼此隔离，互不影响，每个 **Consumer Group** 都可以消费到完整的 **Topic** ，而且消费进度也相互隔离，可以一个 **Consumer Group** 消费的快，另一个 **Consumer Group** 消费的慢

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f30989d305bc441fb3b5112d773f41d6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### Consumer Group 总结

*   从 **Kafka 2.2.1** 开始，每个 **Consumer** 必须指定 **Consumer Group**
*   **Consumer Group** 之间相互隔离互不影响，每个 **Consumer Group** 都可以消费到完整的 **Topic**
*   一个 **Consumer Group** 内，一个 **Topic** 的消费者数量不要超过该 **Topic** 的 **Partition** 数量，避免造成资源浪费

Consumer 与 Topic 之间的分区分配策略
--------------------------

从上面我们了解到，一个 **Topic** 是由 **Consumer Group** 中的所有 **Consumer** 共同来消费的，那问题来了，假设某个 **Consumer Group** 中有三个 **Consumer**， 某个 **Topic** 有 7 个分区，那么这三个 **Consumer** 怎么分配消费这 7 个分区呢？

针对这一问题 **kafka Consumer** 客户端提供了三种分配策略:

*   **RangeAssignor** 默认值
*   **RoundRobinAssignor**
*   **StickyAssignor**

### RangeAssignor 分配策略

**RangeAssignor** 策略的原理是按照消费者总数和分区总数进行整除运算来获得一个跨度，然后将分区按照跨度进行平均分配，以保证分区尽可能均匀地分配给所有的消费者。

假设某个 **Topic** 有 7 个分区，当分别有1、2、3 个消费者消费时的分配情况如下 ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0023eb5ec1540c09cd8c2e331551526~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

当消费者数量超过分区数量时，就会有消费者分配不到分区 ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb69d457eebb43f2a584e4951b67a7e3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### RoundRobinAssignor 分配策略

**RoundRobinAssignor** 策略的原理是将消费组内所有消费者以及消费者所订阅的所有 **topic** 的 **partition** 按照字典序排序，然后通过轮询方式逐个将分区以此分配给每个消费者。

**kafka** 客户端可以将 `partition.assignment.strategy` 参数设置为`org.apache.kafka.clients.consumer.RoundRobinAssignor` 来启动该策略

如果同一个消费组内所有的消费者的订阅信息都是相同的，那么 **RoundRobinAssignor** 策略的分区分配会是均匀的。

例如，消费组中有 2 个消费者 C0 和 C1，都订阅了主题 topic 0 和 topic 1，并且每个主题都有 3 个分区，则分配情况如下

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba52467fab564098828c46ca751c018a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

但如果同一个消费组内的消费者所订阅的信息是不相同的，那么在执行分区分配的时候就不是完全的轮询分配，有可能会导致分区分配的不均匀

例如，消费组内有 3 个消费者 C0、C1 和 C2，它们共订阅了 3 个主题：t0、t1、t2，这 3 个主题分别有 1、2、3 个分区。消费者 C0 订阅的是主题 t0，消费者 C1 订阅的是主题 t0 和 t1，消费者 C2 订阅的是主题 t0、t1 和 t2，那么最终的分配结果为 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a466340f8bc1463dbc08e2afa5d611f0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### StickyAssignor 分配策略

**Kafka** 从 0.11.x 版本开始引入 **StickyAssignor** 分配策略，它主要有两个目的：

1.  分区的分配要尽可能的均匀
2.  分区的分配尽可能的与上次分配的保持相同

当两者发生冲突时，优先第一个目标

上面的那个场景，如果使用 **StickyAssignor** 分配策略，则结果如下 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b88027538b2401aba5a87f4bad0909d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### CooperativeStickyAssignor 分配策略

在 **StickyAssignor** 分配策略的基础上，再增加**渐进式的重平衡**

重平衡(Rebalance)
--------------

分区分配完了就一劳永逸了吗，并不是，有时候一些 **Consumer** 的加入或退出 **Group**，会打破平衡，需要重新再分配一次，这也是这一过程被称为 **重平衡(Rebalance)** 的原因

### 渐进式重平衡(Rebalance)

在 **Kafka 0.9.x** 之前，发生 **Rebalance** 时候会释放 **Consumer Group** 的所有资源，造成比较长的服务终止，之后 **kafka** 经过多次迭代实现了**渐进式的重平衡**

**渐进式重平衡(Rebalance)** 的核心思想是，使用连续的 **Rebalance**，每次**Rebalance**时只撤消或迁移分区总数的几个子集

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf76a5a8832b4beb885299f19a3084b8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

如上图所示，**渐进式 Rebalance 协议**，在 **Rebalance** 的时候不需要当前所有的 **Consumer** 释放所拥有的资源，而是当需要触发 **Rebalance** 的时候对当前资源进行登记，然后进行渐进式的 **Rebalance**。

这样每次 **Rebalance** 对资源的消耗都是比较少，发生迁移的分区也比之前更少了， **Consumer** 在 **Rebalance** 期间可以继续运行

### 重平衡(Rebalance)的发生时机

主要分为两大类

*   **Topic** 的分区发生变化
    
    *   **Topic** 的分区增加
*   **Consumer Group** 中 **Consumer** 的数量发送变化
    
    *   **Consumer** 退出消费组，如 **Consumer** 宕机
    *   有新 **Consumer** 加入消费组

避免 **重平衡(Rebalance)** 也是从这两大块入手，但作为用户， **service** 端我们一般控制不了，只能从客户端这边入手，如固定 **Consumer Group** 中的 **Consumer** 数量

Consumer Configs
----------------

客户端的 **consumer** 与 服务端 **consumer coordinator** 之间的心跳，单位毫秒，默认值 3000。 **consumer coordinator** 超过该时间没有收到 **consumer** 的心跳，会将该 **Consumer** 移出 **Group** ，并执行 **Rebalance** 。该值需要小于 `session.timeout.ms`，建议为 `session.timeout.ms` 的 1/3

```ini
heartbeat.interval.ms=3000

```

**kafka** 客户端与服务端的超时时间，单位毫秒，默认值 10000。 如果服务端超过该时间没有收到 客户端的心跳，会将该客户端移出 **Group** ，并执行 **Rebalance** 。

注意，该值必须在 `group.min.session.timeout.ms` 和 `group.max.session.timeout.ms` 之间

```ini
session.timeout.ms=10000

```

**Consumer** 一次 **poll** ，拉取多少条数据

```ini
max.poll.records=500

```

**Consumer** 处理这批数据的超时时间，单位毫秒，默认值由 300000。若超过该时间，则该 **Consumer** 会被移出 **Group**

```ini
max.poll.interval.ms=300000

```

参考
--

*   [apache documentation](https://link.juejin.cn/?target=https%3A%2F%2Fkafka.apache.org%2Fdocumentation%2F "https://kafka.apache.org/documentation/")
*   [Incremental Cooperative Rebalancing in Apache Kafka: Why Stop the World When You Can Change It?](https://link.juejin.cn/?target=https%3A%2F%2Fwww.confluent.io%2Fblog%2Fincremental-cooperative-rebalancing-in-kafka%2F "https://www.confluent.io/blog/incremental-cooperative-rebalancing-in-kafka/")
*   [KIP-429: Kafka Consumer Incremental Rebalance Protocol](https://link.juejin.cn/?target=https%3A%2F%2Fcwiki.apache.org%2Fconfluence%2Fdisplay%2FKAFKA%2FKIP-429%253A%2BKafka%2BConsumer%2BIncremental%2BRebalance%2BProtocol "https://cwiki.apache.org/confluence/display/KAFKA/KIP-429%3A+Kafka+Consumer+Incremental+Rebalance+Protocol")
*   [Incremental Cooperative Rebalancing: Support and Policies](https://link.juejin.cn/?target=https%3A%2F%2Fcwiki.apache.org%2Fconfluence%2Fdisplay%2FKAFKA%2FIncremental%2BCooperative%2BRebalancing%253A%2BSupport%2Band%2BPolicies "https://cwiki.apache.org/confluence/display/KAFKA/Incremental+Cooperative+Rebalancing%3A+Support+and+Policies")