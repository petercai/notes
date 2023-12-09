# 【Kafka技术专题】「开发实战篇」深入实战探索Kafka的生产者的开发实现及实战指南
定义
--

> **Kafka是一个分布式的基于发布/订阅模式的消息队列（message queue），主要应用于大数据的实时处理领域。** 

消息队列
----

### 传统的消息队列&新式的消息队列的模式

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44a47854e57e47fe8319713a7777af9e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

上面是传统的消息队列，比如一个用户要注册信息，当用户信息写入数据库后，后面还有一些其他流程，比如发送短信，则需要等这些流程处理完成后，在返回给用户

而新式的队列是，比如一个用户注册信息，数据直接丢进数据库，就直接返回给用户成功

kafka的基础架构
----------

kafka的基础架构主要有broker、生产者、消费者组构成，当前还包括zookeeper

> 生产者负责发送消息发送到分区。

> **kafka有主题（Topic）的概念，它是承载真实数据的逻辑容器，而在主题之下还分为若干个分区，也就是说kafka的消息组织方式实际上是三级结构： 主题---分区---消息。主题下的每条消息只会保存在某一个分区中，而不会在多个分区中保存多份。官网上的这张图非常清晰地展示了kafka的三级结构**，如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c587ee34ad804162a0b00b8bf7cd5d2c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

*   **其实分区的作用就是提供负载均衡的能力，或者说对数据进行分区的主要原因，就是为了实现系统的高伸缩性**。
    
*   **不同的分区能够放置到不同节点机器上，而数据库的读写操作也都是针对分区这个粒度而进行的，这样每个节点的机器都能独立的执行各自分区的读写请求处理。** 
    
*   **并且，我们还可以通过添加新的节点记起来增加整体系统的吞吐量。** 
    

分区的原因
-----

1.  **方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个 Partition 组成，因此整个集群就可以适应任意大小的数据了。** 
    
2.  **可以提高并发，因为可以以 Partition 为单位读写了。** 
    

分区的原则
-----

**我们需要将producer发送的数据封装成一个 ProducerRecord 对象。** 

```java
public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value) {
    this(topic, partition, timestamp, key, value, (Iterable)null);
}

public ProducerRecord(String topic, Integer partition, K key, V value, Iterable<Header> headers) {
    this(topic, partition, (Long)null, key, value, headers);
}

public ProducerRecord(String topic, Integer partition, K key, V value) {
    this(topic, partition, (Long)null, key, value, (Iterable)null);
}

public ProducerRecord(String topic, K key, V value) {
    this(topic, (Integer)null, (Long)null, key, value, (Iterable)null);
}

public ProducerRecord(String topic, V value) {
    this(topic, (Integer)null, (Long)null, (Object)null, value, (Iterable)null);
}

```

1.  **指明partition的情况下，直接将指明的值直接作为partiton值。** 
2.  **没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition数进行取余得到 partition 值。** 
3.  **既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition值，也就是常说的 round-robin 算法。** 

### RoundRobinPartitioner（轮询策略）源码：

```java
public class RoundRobinPartitioner implements Partitioner {

    private final ConcurrentMap<String, AtomicInteger> topicCounterMap = new 
		ConcurrentHashMap();

    public RoundRobinPartitioner() {
    }

    public void configure(Map<String, ?> configs) {
    }

    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] 
					valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        int nextValue = this.nextValue(topic);
        List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
        if (!availablePartitions.isEmpty()) {
            int part = Utils.toPositive(nextValue) % availablePartitions.size();
            return ((PartitionInfo)availablePartitions.get(part)).partition();
        } else {
            return Utils.toPositive(nextValue) % numPartitions;
        }
    }

    private int nextValue(String topic) {
        AtomicInteger counter = (AtomicInteger)this.topicCounterMap.computeIfAbsent(topic, (k) -> {
            return new AtomicInteger(0);
        });
        return counter.getAndIncrement();
    }

    public void close() {
    }
}

```

> **为保证 producer 发送的数据，能可靠的发送到指定的 topic，topic 的每个 partition 收到producer 发送的数据后，都需要向 producer 发送 ack（acknowledgement 确认收到），如果producer 收到 ack，就会进行下一轮的发送，否则重新发送数据。** 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94540bd38ccb41f182ec78a2fb2e76b8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c82cff5d61a8497ab8ac3211c64a9b1d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

Kafka 选择了第二种方案，原因如下：

1.  **同样为了容忍 n 台节点的故障，第一种方案需要 2n+1个副本，而第二种方案只需要 n+1个副本，而 Kafka 的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。** 
    
2.  **虽然第二种方案的网络延迟会比较高，但网络延迟对Kafka的影响较小。** 
    

ISR
---

*   **采用第二种方案之后，设想以下情景：leader收到数据，所有follower都开始同步数据，但有一个follower，因为某种故障，迟迟不能与 leader 进行同步，那 leader 就要一直等下去，直到它完成同步，才能发送ack。这个问题怎么解决呢？**
    
*   **Leader维护了一个动态的in-sync replica set (ISR)，意为和 leader 保持同步的 follower 集合。当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。** 
    
*   如果follower长时间 未 向 leader 同 步 数 据 ， 则 该 follower 将 被 踢 出 ISR ， 该 时 间 阈值由**replica.lag.time.max.ms**参数设定。Leader 发生故障之后，就会从 ISR 中选举新的 leader。
    

### ack应答机制

> **对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等ISR中的follower全部接收成功。** 

所以Kafka为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡， 选择以下的配置。

*   acks = 0：生产者只负责发消息，不管Leader 和Follower 是否完成落盘就会发送ack 。这样能够最大降低延迟，但当Leader还未落盘时发生故障就会造成数据丢失。
    
*   acks = 1：Leader将数据落盘后，不管Follower 是否落盘就会发送ack 。这样可以保证Leader节点内有一份数据，但当Follower还未同步时Leader发生故障就会造成数据丢失。
    
*   acks = -1(all)：生产者等待Leader 和ISR 集合内的所有Follower 都完成同步才会发送ack 。但当Follower同步完之后，broker发送ack之前，**Leader发生故障时，此时会重新从ISR内选举一个新的Leader，此时由于生产者没收到ack，于是生产者会重新发消息给新的Leader，此时就会造成数据重复。** 
    

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52177e3dba9946f99835e21904b9a56e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 故障处理细节

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/435b3fa61ecc4d7abdcf44ce1fa579b5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

* * *

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35a9330268a84cc7ae7749470cbb83aa~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

*   **LEO——Log End Offset：Producer 写入到 Kafka 中的最新一条数据的 offset。** 
*   **HW——High Watermark：：指的是消费者能见到的最大的 offset，ISR 队列中最小的 LEO**。

#### follower 故障

> **follower 发生故障后会被临时踢出 ISR，待该 follower 恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。等该 follower 的 LEO 大于等于该 Partition 的 HW，即 follower 追上 leader 之后，就可以重新加入 ISR 了。** 

### leader 故障

> **leader 发生故障之后，会从 ISR 中选出一个新的 leader之后，为保证多个副本之间的数据一致性，其余的 follower会先将各自的log文件高于HW的部分截掉，然后从新的 leader同步数据。** 

> **注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。** 

Exactly Once 语义
---------------

在分布式消息传递一致性语义上有下面三种：

*   **At Least Once：消息不会丢，但可能会重复。** 
*   **At Most Once：消息会丢，但不会重复。** 

> *   **Exactly Once：消息不会丢，也不会重复。** 

在kafka0.11版本之前是无法在kafka内实现exactly once 精确一次性保证的语义的，在0.11之后的版本，**我们可以结合新、幂等性以及acks=-1 来实现kafka生产者的exactly once。** 

*   **acks=-1**：**在上面ack机制里已经介绍过，他能实现at least once语义，保证数据不会丢，但可能会有重复数据。** 

> **幂等性：0.11版本之后新增的特性，针对生产者，指生产者无论向broker发送多少次重复数据，broker 只会持久化一条；**

> **在0.11版本之前要实现exactly once语义只能通过外部系统如hbase的rowkey实现基于主键的去重。** 

幂等性解读：
------

在生产者配置文件producer.properties 设置参数 **enable.idompotence = true** 即可启用幂等性。

Kafka的幂等性其实就是将原来需要在下游进行的去重操作放在了数据上游。

*   **开启幂等性的生产者在初始化时会被分配一个PID（producer ID），该生产者发往同一个分区（Partition）的消息会附带一个序列号（Sequence Number），Broker端会对<PID, Partition, SeqNumber>作为该消息的主键进行缓存，当有相同主键的消息提交时，Broker 只会持久化一条。** 
    
*   **但是生产者重启时PID就会发生变化，同时不同的分区（Partition）也具有不同的编号，所以生产者幂等性无法保证跨分区和跨会话的 Exactly Once**。
    

> **事务：kafka在0.11 版本引入了事务支持。事务可以保证 Kafka 在 Exactly Once 语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。** 

*   **Kafka引入了一个新的组件Transaction Coordinator，它管理了一个全局唯一的事务ID（Transaction ID），并将生产者的PID和事务ID进行绑定，当生产者重启时虽然PID会变，但仍然可以和Transaction Coordinator交互，通过事务ID可以找回原来的PID，这样就保证了重启后的生产者也能保证Exactly Once了**。
    
*   **同时，Transaction Coordinator 将事务信息写入Kafka的一个内部Topic，即使整个kafka服务重启，由于事务状态已持久化到topic，进行中的事务状态也可以得到恢复，然后继续进行。** 
    

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d145763073b4cc59daf17788752a8fa~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

由上图可以看出：KafkaProducer有两个基本线程：

*   主线程：**负责消息创建，拦截器，序列化器，分区器等操作，并将消息追加到消息收集器RecorderAccumulator中（这里可以看出拦截器确实在序列化和分区之前执行）。** 
    
*   **消息收集器RecoderAccumulator为每个分区都维护了一个 `Deque<ProducerBatch>` 类型的双端队列**。
    

> **ProducerBatch 可以暂时理解为是 ProducerRecord 的集合，批量发送有利于提升吞吐量，降低网络影响。** 

* * *

*   **由于生产者客户端使用 java.io.ByteBuffer 在发送消息之前进行消息保存，并维护了一个 BufferPool 实现 ByteBuffer 的复用**；
    
*   **该缓存池只针对特定大小（ batch.size 指定）的 ByteBuffer进行管理，对于消息过大的缓存，不能做到重复利用**。
    

> **每次追加一条ProducerRecord消息，会寻找/新建对应的双端队列，从其尾部获取一个ProducerBatch，判断当前消息的大小是否可以写入该批次中。** 

**若可以写入则写入；若不可以写入，则新建一个ProducerBatch，判断该消息大小是否超过客户端参数配置 batch.size 的值，**

**不超过，则以 batch.size建立新的ProducerBatch，这样方便进行缓存重复利用；**

**若超过，则以计算的消息大小建立对应的ProducerBatch ，缺点就是该内存不能被复用了。** 

### Sender线程：

> **该线程从消息收集器获取缓存的消息，将其处理为 <Node, List 的形式， Node 表示集群的broker节点**。

> **进一步将<Node, List转化为<Node, Request>形式，此时才可以向服务端发送数据。** 

> **在发送之前，Sender线程将消息以 Map<NodeId, Deque> 的形式保存到 InFlightRequests 中进行缓存，可以通过其获取 leastLoadedNode ,即当前Node中负载压力最小的一个，以实现消息的尽快发出。** 

#### 写入流程

producer 写入消息序列图如下所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9aef27d24009455b881ac8f5012bfee9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

流程说明：
-----

1.  **producer 先从 zookeeper 的 "/brokers/.../state" 节点找到该 partition 的 leader**。
2.  **producer 将消息发送给该 leader**。
3.  **leader 将消息写入本地 log**。
4.  **followers 从 leader pull 消息，写入本地 log 后 leader 发送 ACK**。
5.  **leader 收到所有 ISR 中的 replica 的 ACK 后**，**增加 HW（high watermark，最后 commit的offset） 并向producer发送 ACK**。