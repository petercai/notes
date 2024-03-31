# 一文搞懂 Kafka consumer 与 broker 交互机制与原理 
**01 前言**
---------

今天给大家带来的是 **Kafka Consumer 与 Kafka Broker 之间的交互机制解析，并简要介绍其背后的主要工作机制**，参考的 Kafka 源码版本为 3.4。

Kafka Consumer 是 Kafka 事件（消息）的消费端客户端，它是 Kafka 的关键组件之一。为了确保 Kafka 集群的高效运行，Kafka 的客户端被设计为富客户端，例如，消费者组中的分区分配就是在客户端完成的。无论你是 Kafka 的用户还是开发者，都有必要了解 Kafka Consumer 的基本工作原理。

**02 消费者的角色**
-------------

Kafka consumer 一般是以 group 的形式消费的，group 中的每个成员称为一个 consumer member。根据分配到的角色，可以进一步划分为：

**◾ leader：**  特殊的一个 member，负责分配所有 member 到 topic partition 的映射；

**◾ follower：**  除了 leader 以外的其他所有 member；

**03 消费流程涉及的核心组件**
------------------

**broker 侧：** 

Kafka consumer 会不断与 kafka broker 通信。其中 broker 侧涉及以下组件：

**◾ group coordinator：**  负责同步 consumer member 状态、监听心跳、触发 rebalance、挑选 consumer leader 等行为；

**◾ replica manager:** 负责 topic partition 副本的管理（读、写等）；

**consumer 侧：** 

**◾ metadata：**  Kafka 集群的元信息；

**◾ client：**  ConsumerNetworkClient 实例，负责网络层读写；

**◾ assignors：**  consumer leader 中负责指定所有 consumer member 到 topic partition 的映射；

**◾ coordinator：**  ConsumerCoordinator 实例，负责与 broker 侧的 group coordinator 交互；

**◾ fetcher：**  负责拉取消息；

**04 常用接口**
-----------

Kafka 的 consumer 的常用接口：

**◾ subscribe：**  指定 consumer 订阅的 topic

**◾ poll：**  拉取消息；

**◾ close：**  优雅退出 consumer；

**◾ commit:** 手动提交消费位点；

在 Kafka 中，subscribe 主要用于更新消费者状态信息，而 commit 则是将特定位点发送给 broker。这两个接口的逻辑相对简单，我们不会在本文中详细展开讨论。接下来的章节将重点介绍 poll 和 close 两个接口的交互和原理。

**05 consumer 与 broker 交互流程解析**
-------------------------------

下图展示了 consumer 和 broker 在消费过程中的交互逻辑：

![](https://image.automq.com/20240329bot/NGM4z1.png)

上图的交互流程总体可以分为“消费过程”和“退出过程”，在接下来的几个小节中我们将对其做详细的介绍。

### **5.1 消费过程**

消费过程大体可以分为两块逻辑：

◾ 加入 consumer group，获取负责的 topic partition；

◾ 基于负责的 topic partition，向所在的 broker 拉取消息；

#### **5.1.1 加入 consumer group**

##### **5.1.1.1 FindCoordinator 阶段**

![](https://image.automq.com/20240329bot/UErFSK.png)

每次调用 KafkaConsumer#poll 时，都会触发 ConsumerCoordinator#poll 的调用，确保 consumer 到 GroupCoordinator 的通信是正常的。在 consumer 第一次 poll 时，肯定是找不到 GroupCoordinator 的，于是：

1.  Consumer 向最近通信过的 broker 发送 FindCoordinator 请求；
    
2.  该 broker 根据 group.id 进行 hash，再对 \_\_consumer\_offsets 的 partition 数目取模，找到负责该 group 的 partition 后，返回 partition leader 所在的 broker 地址；
    
3.  Consumer 从 FindCoordinator response 中解析出负责本 group 的 broker 的地址，后续 Consumer 侧的 coordinator 组件会与新 broker 通信，同步 consumer group 的状态；
    

在本阶段执行到最后时，HeartBeatThread 线程将会启动，该线程主要负责向 broker 侧的 GroupCoordinator 发送心跳。GroupCoordinator 会在 HeartBeat response 附带一些信息，例如指向了错误的 GroupCoordinator、consumer group 正在重平衡等信息。

注意：此时 consumer 还没有加入 group，HeartBeatThread 虽然启动了，但没有 enable，**还不会向 GroupCoordinator 发送心跳**。

##### **5.1.1.2 JoinGroup 阶段**

![](https://image.automq.com/20240329bot/SF3MSs.png)

如果 consumer 还没有加入 consumer group，那么会向 GroupCoordinator 请求加入 group：

1.  Consumer 发送 JoinGroup 请求；
2.  GroupCoordinator 会检查 JoinGroup 请求的合法性。consumer 在构造的时候是没有 member id 的，因此 JoinGroup 请求中没有附上 member id。此时，**GroupCoordinator** 会为这个新 consumer **生成一个 member id**，随 **MEMBER\_ID\_REQUIRED 异常**一并返回；
3.  Consumer 填入 member id，再次发送 JoinGroup 请求；
4.  GroupCoordinator 会在 JoinGroup response 中告知 consumer 当前 group leader 的 member id 以及 consumer 自己的 member id。对于 leader，会额外返回所有 consumer 的 member id，以便 leader 进行后续的 partition 分配工作。

在该阶段最后，GroupCoordinator 会将该 consumer group 置为 **rebalance** 状态，从而触发 group 内其他 member 的 **rejoin group** 动作。此时，HeartBeatThread 也会被 enable，开始与 GroupCoordinator 的心跳通信。

开始 rebalance 后， broker 会等待 consumer 加入 group。等待会有超时时间，超时后 broker 会踢出没有及时加入 group 的旧 member，将当前的 group 元数据持久化。

提示1：一般来说，group 的 consumer leader 是第一个向 GroupCoordinator 发起 JoinGroup 请求的 consumer。

提示2：member id 是**不可手动设置**的。Consumer 侧有个类似的配置是 **group.instance.id** ，用于声明 consumer 为 静态 consumer。静态 consumer 与普通 consumer 的最大区别在于**退出时不会发送 LeaveGroup 请求**。在用户业务升级时， 普通 consumer 退出后再拉起会导致较频繁的 rebalance，静态 consumer 就可以规避这种情况（通常会搭配较大的 session timeout 配置）。

##### **5.1.1.3 SyncGroup 阶段**

![](https://image.automq.com/20240329bot/b8eRjS.png)

1.  **在 consumer member 中分配 partition:** 在收到 JoinGroup response 后，consumer group leader 会根据指定的 partition assignment strategy（由 partition.assignment.strategy 参数设置），进行 topic partition 在各个 member 中的分配。
    
2.  **consumer 执行 SyncGroup 请求：**  leader consumer 会发送 leader SyncGroup 请求，附上 topic partition 与 member 的映射结果；其他 member 会发送 follower SyncGroup 请求，尝试获取自己需要负责的 topic partition。
    

在该阶段最后，GroupCoordinator 会持久化 group 的 metadata 到该 group 绑定的某个 \_\_consumer\_offsets 的 partition 中。

#### **5.1.2 拉取消息**

##### **5.1.2.1 OffsetFetch 阶段**

![](https://image.automq.com/20240329bot/Uioo1Q.png)

各个 consumer member 收到 SyncGroup response 以后，需要确定 partition 消费的起始位点。consumer 会向 GroupCoordinator 查询该 group 关于指定 partition 已经提交的 commited offset，此时：

◾ 如果该 partition 查询到了 commited offset 记录，那么 consumer 会从该 offset 开始继续消费；

◾ 否则，根据 consumer 配置的 **auto.offset.reset**，决定起始消费位点。

##### **5.1.2.2 ListOffset 阶段**

![](https://image.automq.com/20240329bot/eDHviR.png)

如果上一步中，partition 没有查询到 commited offset 记录，那么 consumer 会利用 ListOffset 请求（基于 auto.offset.reset 对应的策略指定请求中的 timestamp 字段）的 response，确定起始消费位点。

##### **5.1.2.3 Fetch 阶段**

![](https://image.automq.com/20240329bot/pUrfC2.png)

基于此前的 offset 信息，consumer 向 partition 所在的 broker 发起拉取消息的请求，拉取成功后会更新下次需要拉取的位点。

##### **5.1.2.4 OffsetCommit 阶段**

![](https://image.automq.com/20240329bot/kYPQXG.png)

在消费过程中， consumer 或自动或手动地提交当前消费位点到 GroupCoordinator 处。类似于 SyncGroup 请求，GroupCoordinator 会将该位点信息持久化。

### **5.2 退出过程**

![](https://image.automq.com/20240329bot/khhNBl.png)

Consumer 调用 close，进入优雅退出逻辑：

1.  Consumer 同步提交位点信息；
2.  关闭 Heartbeat 线程；
3.  Consumer 发送 LeaveGroup 请求到 GroupCoordinator，但**不会阻塞式等待 response**；
4.  GroupCoordinator 收到 LeaveGroup 请求后，将 group 置为 rebalance 状态，触发该 group 中其他 member 的重平衡。

注意: 由于 Consumer 关闭时不会阻塞式等待 LeaveGroup 的 response，在“consuemr 关闭”和“group coordinator 清除该 Consumer 信息” 两个事件之间会存在一小段时间间隙。不等 response 的设计是为了**加速 consumer 的关闭**，即使 broker 没有收到 Consumer 发送的 LeaveGroup 请求，也会由于心跳超时被踢出 consumer group。

**06 broker 侧 consumer group 状态管理**
-----------------------------------

本节我们分析下 broker 是如何管理 consumer group 状态的，来进一步强化对消费过程的理解。broker 侧 group metadata 存在一个字段，标志当前 group 的状态：

**◾ Empty：**  group 没有 member，等待 offsets 信息失效。常作为初始状态；

**◾ PreparingRebalance：**  rebalance 开始；前文中提到的 broker 会通知所有 member 重平衡，就是在这个状态下通知的；

**◾ CompletingRebalance：**  等待 group leader 提交分配结果；

**◾ Stable：**  group 稳态（所有 consumer 都在正常消费）；

**◾ Dead：**  group 没有 member，且 offsets 信息为空；Dead 是最终状态，不可转化为其他状态；

状态机视图如下：

![](https://image.automq.com/20240329bot/RQwFOa.png)

上图中为了简略性，只列出了两个常见的转化为 Dead 状态的情况，实际上以下四种情况都会导致状态转为 Dead：

◾ Empty Group（没有 member） 的手动删除；

◾ Group metadata 失效（offsets 信息为空）。原因一般是定时任务清理掉了所有 offsets（已失效）；

◾ OffsetsDelete 或 PartitionsDelete 之后，如果 offsets 被清空且 Group 是 Empty；

◾ GroupUnload，即\_\_consumer\_offsets 的某个 partition 的 leader 从本机切出去，将内存中 cache 的相关 Group metadata 置为 Dead；

注意：图上所谓“join completed”，指的是 rebalance 结束。rebalance 结束的原因可能是超时或者旧 member 都已经重新加入了。

**07 rebalance 实现原理**
---------------------

rebalance 是 Kafka 中 consumer group 中的一个关键操作，用于在 consumer group 中实现负载均衡和容错。理解其原理对于理解 Kafka 的消费原理至关重要。

### **触发 rebalance 的时机：** 

◾ Group 刚创建（第一个 consumer 发起 JoinGroup 请求）；

◾ consumer 到 GroupCoordinator 的心跳超时，被移除出 group；

◾ 新的 consumer 加入 group；

◾ Consumer Group 订阅的某个 topic 的 partition 数目增加了；

◾ Consumer Group 使用通配符订阅规则，并且有新的匹配的 topic 被创建了；

### **broker 广播 rebalance 状态的方式：** 

附着在 HeartbeatResponse 或者 OffsetCommitResponse 中，以 error code 形式告知 consumer 需要 rejoin group。

**08 重平衡 Q&A**
--------------

该小节，我们再通过一些问题来巩固对 rebalance 的理解：

**Q:** PreparingRebalance 状态下是否会停止消费？

**A：**  当且仅当 consumer 感知到自己需要 rejoin group 才会停止消费。PreparingRebalance 状态下可以正常消费和提交位点。不过 CompletingRebalance 状态下不允许提交位点，会抛出 Errors.REBALANCE\_IN\_PROGRESS，触发 consumer 的 rejoin 动作。

**Q:** Consumer 手动 assign 和 rebalance 两种模式的区别？

**A：**  手动 assgin 模式使用的是 Kafka consumer 的 assign 接口：

`consumer.assign(Collections.singleton(new TopicPartition("test-topic", 0)));`

rebalance 模式下，Kafka consumer 会订阅指定的 topic，使用的是 Kafka consumer 的 subscribe 接口：

`consumer.subscribe(Collections.singleton("test-topic"));`

二者的主要区别如下：

**◾ Topic partition 分配者不同：**  前者是在调用 assign 接口时手动指定的，后者是 consumer group leader 分配的；

**◾ 重平衡行为不同：**  手动 assign 时 Kafka consumer 跟 topic partition 是静态绑定的，Kafka consumer 不会参与重平衡；rebalance 模式会根据 consumer 加入、退出等情况触发重平衡，调整各个 Kafka consumer 分配到的 topic partition；

**◾ Group 元数据包含信息不同：**  assign 模式下的 group metadata 是没有 member 信息的，仅用于存储位点信息；

需要注意的是，两种模式互斥。assign 模式下，Kafka consumer 不支持动态扩容，当生产速率突增时，无法及时加入新的消费者来提升消费的速率。如果业务希望完全避免消费过程中出现 topic partition 漂移（一种可能的场景是，生产者将 user_id 作为 record key，且消费时要求只能有一个 consumer 处理同一个 user 的数据），那么才有必要考虑使用 assign 模式。此外，assign 模式还需要注意避免 group id 与其他 group id 碰撞，否则有可能导致 commited offset 的污染。

**09 总结**
---------

本文详细阐述了 Kafka consumer 的主要生命周期背后的原理，重点介绍了 consumer 在消费和退出过程中与 broker 之间的交互机制。此外，还对 group 状态管理、rebalance 原理做了分析，使得读者对 consumer 与 broker 的交互有了全面的了解。
