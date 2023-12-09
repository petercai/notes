# 详解Kafka分区机制原理｜Kafka 系列 二
上一篇文章介绍了 Kafka 的基本概念和术语，里面有个概念是 **分区(Partition)**。

kafka 将 一个Topic 中的消息分成多份，分别存储在不同的 Broker 里，这每一段消息被 kafka 称为分区，其中每条消息只会保存在一个分区中。

如果不太理解请回顾上一篇：

为什么有分区？
-------

为什么要有分区呢？

Kafka 的分区机制的本质就是将一个大的 Topic 进行拆分，将一组很大的队列拆分成了多组队列。这样做有以下几个好处：

1.  因为一个 Topic 中的消息可能非常多，多到一台Broker存不下，因此需要拆分成多段存储在不同的机器里，实现负载均衡。
2.  拆分成多个队列，可以在多个生产者和消费者的情况下发挥多机性能，可以分流和并行处理消息，从而**提高读写性能**，提升系统的吞吐力。
3.  有利于系统扩缩容，提高系统的可扩展性。不同分区在不同的broker上，可以通过增加新机器提高吞吐，并且增加新机器的时候可以通过调整分区的分布来调配负载。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ccf4a36a8bf451fb77d161c841f7a46~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1294&h=656&s=479843&e=png&b=fdfdfd)

但是分区数不是越多越好，需要根据系统具体情况来设置。比如3个Broker就应该至少有3个分区，如果broker性能之间有差异，可以调大分区数进行调配。也可以通过broker的倍数来设置分区数，并且进行性能压测，测试集群的吞吐量。

分区数过多会带来资源管理上的消耗，清除日志时间变长，集群broker故障后分区leader重选时间变长，客户端消费端线程数需求增加，甚至导致连接所需的socket消耗增加。

**分区策略**就是决定生产者将会把消息发送到具体哪个分区的算法，分区策略由 `Partitioner` 接口实现。

自定义分区策略
-------

用于分区的 `partition` 方法定义如下：

```java
    
     * Compute the partition for the given record.
     *
     * @param topic topic名 The topic name
     * @param key 用于分区的key The key to partition on (or null if no key)
     * @param keyBytes 用于分区的序列号key The serialized key to partition on( or null if no key)
     * @param value 用于分区的值 The value to partition on or null
     * @param valueBytes 用于分区的序列号值 The serialized value to partition on or null
     * @param cluster 当前集群元数据 The current cluster metadata
     */
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);


```

可以看出，这里提供了 Topic 和一些跟消息有关的key参数，cluster 是集群信息，包含Kafka 当前的Node 数据以及Topic、partition数据等。有了这些数据，具体拿到一条消息该发往哪个分区，我们就可以根据已有信息制定自己的分区策略。

我们实现了自定义的 Partition 类之后，就可以设置 `partitioner.class` 为目标策略类，Producer 就会按照我们的自定义策略来对消息进行分区。

默认分区策略
------

Kafka 提供了默认分区策略 `DefaultPartitioner`，策略内容如下：

1.  如果在消息中指定了分区，优先使用指定的分区。
    
2.  如果没有指定分区，但存在分区键，则根据序列化key使用murmur2哈希算法对分区数取模。
    
3.  如果没有指定分区或分区键，则会使用**粘性分区策略**。（关于粘性分区策略后面讲解）
    

在实际生产中，我们一般都默认使用此策略，无需修改。

```java
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
    return partition(topic, key, keyBytes, value, valueBytes, cluster, cluster.partitionsForTopic(topic).size());
}
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster,
                         int numPartitions) {
    if (keyBytes == null) {
        return stickyPartitionCache.partition(topic, cluster);
    }
    
    return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
}

```

注意，这里指的分区键是序列化后的key，也就是变量 keyBytes，其他key、value、valueBytes 并没用到。

```scss
byte[] keyBytes = keySerializer.serialize(topic, record.headers(), record.key());
default byte[] serialize(String topic, Headers headers, T data) {
		
    return serialize(topic, data);
}

```

看到 key 等序列化方法我们可以明白，key 的序列号值只受到 record.key() 的影响，所以同样的key会被固定分配到同样的partition中。（注意这里的key是指用于分区的key，而不是topic）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3747cdd0463e41c4bc0281a9d244447a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1298&h=498&s=83170&e=png&b=fdfdfd)

粘性分区策略
------

实现类为 `UniformStickyPartitioner` ,他与默认分区策略的区别是：

*   DefaultPartitionerd 默认分区策略：如果有分区键的话，会按照分区键来决定分区，这个时候并不会使用粘性分区策略。
*   UniformStickyPartitioner粘性分区策略：无论有没有分区键，都用粘性分区来分配。

```java
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
    return stickyPartitionCache.partition(topic, cluster);
}

```

### 什么是粘性分区策略？

我们需要知道，在Producer在发送消息的时候，会将消息放到一个ProducerBatch中， 然后多条消息批量发送。这样可以减少网络请求次数，提高消息的发送效率。

所以批量发送消息有两个条件：

1.  一个batch满了，与 `batch.size`有关，一般大小是16k。
2.  `linger.ms`时间到了。

满足任意一个条件，都会触发sender线程的发送。如果生产的消息较少，batch没有满，就必须等到等待时间到了，这就导致了较长的延迟。

因为ProducerBatch是多个，为了让消息尽可能快的发送，就需要让其中一个ProducerBatch先变满。假设现在有三个分区，会优先分配给

```swift
private final ConcurrentMap<TopicPartition, Deque<ProducerBatch>> batches;

```

注意：一个分区对应一个双端队列`Deque<ProducerBatch>>`。

粘性分区策略就是在相同的分区中，优先填满一个ProducerBatch，发送，再去填充另一个ProducerBatch。参见下图，第一个分区会被优先塞满并发送。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1227923fdd749acab58fef9338d8449~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1360&h=538&s=98533&e=png&b=fefdfd)

在一个 ProducerBatch 发送结束，选择新分区的时候，是随机选择的，之后便会继续优先填满新的分区。

*   可用分区<1 ，所有分区中随机选择。
*   可用分区=1，选择这个分区。
*   可用分区>1，所有可用分区中随机选择。

```ini
public int nextPartition(String topic, Cluster cluster, int prevPartition) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic)
        Integer oldPart = indexCache.get(topic)
        Integer newPart = oldPart
        // Check that the current sticky partition for the topic is either not set or that the partition that 
        // triggered the new batch matches the sticky partition that needs to be changed.
        if (oldPart == null || oldPart == prevPartition) {
            List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic)
            if (availablePartitions.size() < 1) {
                Integer random = Utils.toPositive(ThreadLocalRandom.current().nextInt())
                newPart = random % partitions.size()
            } else if (availablePartitions.size() == 1) {
                newPart = availablePartitions.get(0).partition()
            } else {
                while (newPart == null || newPart.equals(oldPart)) {
                    int random = Utils.toPositive(ThreadLocalRandom.current().nextInt())
                    newPart = availablePartitions.get(random % availablePartitions.size()).partition()
                }
            }
            // Only change the sticky partition if it is null or prevPartition matches the current sticky partition.
            if (oldPart == null) {
                indexCache.putIfAbsent(topic, newPart)
            } else {
                indexCache.replace(topic, prevPartition, newPart)
            }
            return indexCache.get(topic)
        }
        return indexCache.get(topic)
    }

```

轮询分区策略
------

Kafka 中提供了轮训策略的实现 `RoundRobinPartitioner`。当用户希望将写操作均匀地分发到所有分区时，可以使用此分区策略。

举例，有三个分区，针对于同一个producer，第一条消息发送到partition1，第二条消息发送到partition2，第三条发送到partition3，以此类推。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f1d1fbb01634c45bf9d4b0397259640~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1260&h=462&s=53065&e=png&b=fdfdfd)

```scss
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
    List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
    
    int numPartitions = partitions.size();
    
    int nextValue = nextValue(topic);
    
    List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
    if (!availablePartitions.isEmpty()) {
        
        int part = Utils.toPositive(nextValue) % availablePartitions.size();
        return availablePartitions.get(part).partition();
    } else {
        
        
        return Utils.toPositive(nextValue) % numPartitions;
    }
}

```

hash 键的值并不会影响到数据的分布，这应该是数据均匀度最好的策略，可以保证消息最大程度的平均分配到所有分区。

除了官方提供的策略，我们还可以实现自己的分区策略，比如随机策略，实现起来也很简单；比如按照业务键去分区的策略；比如按照ip分区的策略等。