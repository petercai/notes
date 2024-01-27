# Kafka Consumer Group
快速介绍消费者组的特点：

1.  kafka 提供的可扩展且具有容错性的消费者机制
2.  一个组内含有多个消费者实例
3.  消费配置的时候，是指定 `group-id`。也就是共享
4.  组内的消费者一起协调消费订阅的主题的所有分区
5.  每个分区只能由一个 group 内的一个 Consumer Instance 消费

> 总结：一个分区只能由一个组内的消费者消费；但是一个消费者可以消费多个分区；这是一个 N:1 的模型

消费模型
----

1.  p2p
2.  pub/sub

而 Consumer Group 统一了这两个模型：

*   如果所有 Consumer Instance 都属于同一个Group，那么它实现的就是消息队列模型；
*   如果所有 Consumer Instance 分别属于不同的Group，那么它实现的就是 `pub/sub`

**理想情况下，Consumer实例的数量应该等于该Group订阅主题的分区总数【也可以小于，但最好不要大于，浪费消费者】** 

消费控制
----

首先消费者消费，需要提交消费位移的 -\> 也就是 **位移提交**。这是对单个消费者。

如果在 Consumer Group，那这个 Group 需要管理下属的消费者实例，也就是意味需要记录这些 Consumer 的消费位移。

**很容易想到这是一个 KV pair，Key是分区，V对应Consumer消费该分区的最新位移。大致可以理解为：`Map<TopicPartition, Long>`**，不过其实这个结构很复杂。之后讲吧。。。

### 消费者版本区别

*   先说区别：

1.  老版本的 Consumer Group 把位移保存在 ZooKeeper 中
2.  新版本将位移提交保存在 Kafka 内部主题中，也就是：`__consumer_offsets`

*   原因是什么？

位移提交即便是异步提交，但是对 `zk` 依然是一个频繁写。所有 kafka0.8.2.x 在新版本 Consumer 推出了 **位移主题** 这种设计思路。对于位移提交，要求只有两个：

1.  高持久化
2.  高频写

显然，Kafka自己本身主题设计天然就满足这两个条件。

### \_\_consumer\_offsets

`__consumer_offsets` 的位移管理也比较简单：既然是主题，那也就是将 Consumer 的位移提交作为一条普通的 **位移msg** 提交到这个 `__consumer_offsets` 主题即可。**本质来说：`__consumer_offsets` 保存的就是 Consumer 的位移信息**

> 虽然它是一个主题，但同时是一个内部主题，开发者不需要管理它，不需要操作它【不允许写，不允许删除】

先简单提一下它的消息格式，简单来说消息格式就是一种 kv pair 组织结构：

1.  key：<Group ID，主题名，分区号>
2.  value：简单地认为消息体就是保存了位移值

位移提交一般都建议是：手动提交，即设置 `enable.auto.commit = false`。需要开发者显式调用 `kafka` 提供的位移提交方法：`consumer.commitSync`。

如果采取的是自动提交，就会出现一个问题：只要 `comsummer` 进程不停止，就会不停的向 `__consumer_offsets topic` 写入提交消息。

出现的问题：一个位移的提交可能会被提交多次。但是对于 `broker` 而言，只需要最新的这条位移提交就OK了。这就需要 `kafka` 对 `__consumer_offsets topic` 中位移提交消息做一个删除，不然：

1.  无效消息过多，压爆 `broker` 磁盘
2.  消费者重复消费 (当然这个可以通过业务逻辑规避)

所以 `kafka` 使用了 `Compact策略` 来删除 `__consumer_offsets topic` 中的过期消息。**Kafka提供了专门的后台线程定期地巡检待Compact的主题，看看是否存在满足条件的可删除数据。这个后台线程叫 `Log Cleaner`**

还有一些其他注意点：

1.  位移主题的位移由 `Kafka` 内部的 `Coordinator` 自行管理

### Rebalance

*   先说含义：

> 本质是一个协议，规定了 Consumer Group 下所有的 Consumer 如何达成一致，从而分配订阅 Topic 下的每个分区。

那么 Consumer Group 何时进行 Rebalance 呢？触发条件有3个：

1.  group 中的 Consumer Instance 数量发生变更：新的加入/之前的退出（结束消费了）
2.  group 的 topic 数发生变更：`consumer.subscribe(Pattern.compile("t.*c"))` 如果采用正则的方式消费主题，新的 topic 加入，那么该 group 需要指派多的分区到 consumer instance 去消费
3.  group 的 topic 中 partition 数量发生变更

*   有什么性能问题呢？

1.  Rebalance 过程类似 STW。当前组内的全部 Consumer Instance 停止消费，等待这个过程完成
2.  整个过程需要全体 Consumer 参与，全部重新分配所有分区【按照常理应该是现有的分配不变，多退少补。这样的话，Consumer 连接这些分区所在Broker的TCP连接就可以继续用，不用重新创建连接其他Broker的Socket资源】
3.  整个过程很慢，很慢。这个是关键，如何结合 \[1\] 就更是如此了

> 在 `Rebalance` 过程中，是所有同一个 `Consumer Group` 下的所有 `Comsumer` 实例共同参与，在 `Coordinator` 组件帮助下，完成 `partition` 的分配。但是在整个过程中所有实例都不能消费消息，因此对 `Comsumer TPS` 影响很大。

上面提到的 `Coordinator`：是专门为 `Consumer Group` 服务，负责 `Group` 中的位移提交和 `Consumer Instance` 管理，同时也提供 `Consumer Rebalance`。

Coordinator
-----------

每一个 `Broker` 启动时，会去创建和开启相应的 `Coordinator` 组件，一对一关系。那么 `Consumer Group` 是如何确定服务自己的 `Coordinator`？消费消息对应的哪一个 `broker`？主要有两个步骤：

1.  在 `__consumer_offsets` 中计算 `partitionId=Math.abs(groupId.hashCode() % offsetsTopicPartitionCount)`，通过对 `Consummer Group groupId` hash，然后对 50 (默认`__consumer_offsets`分区数) 取余，计算出 `partitionId`
2.  根据上一步的 `partitionId`，比如 15，对应 `__consumer_offsets` 分区15，然后再找到分区15的 `leader` 在哪一个 `broker` ，就知道对应的 `Coordinator`

以上其实是帮助我们开发者确定，出问题可以快速确认问题节点。但是在实际运行时，`kafka` 会自动确认。

然后我们来解决一下上面的 `Rebalance` 问题。可惜的是：不能解决，但是不能解决可以尽可能避开。所有回到之前引发的3个场景：

1.  组成员数量变化
2.  组订阅的主题发生变化
3.  分区内主题变化

对于订阅本身发生的变化几乎在运维层面上：可能是业务需要扩展或者是消费速率等问题。基本上我们遇到 `Rebalance` 都是组成员数变化。

一般发生在：向一个已经存在的 `group_id` 加入 `Consummer Instance`，也就是我们自己启动了一个消费者程序。然后 `Coordinator` 会接纳这个新实例，将其加入到组中，并重新分配分区。

所有问题就出现了。这个其实也算正常，其实可以规避，_可以采取多线程(协程)消费_ 。