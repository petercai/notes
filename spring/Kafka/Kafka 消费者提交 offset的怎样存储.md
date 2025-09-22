# Kafka 消费者提交 offset的怎样存储
### Kafka 消费者提交 offset的怎样存储

#### Kafka 消费者消费消息需要提交 offset，为什么要存储 offset？

想象一下，你正在读一本书📖。你是不是需要一个书签来标记你读到的地方？在 Kafka 世界里，offset 就是那个书签！消费者存储 offset 是为了确保即使在重启或崩溃后，它也能从正确的位置继续消费消息，不会漏掉或重复消费。这样就像你拿起书签从上次读到的地方继续阅读一样，确保消息处理的 **准确性** 和 **一致性**。

#### 那么 offset 是存储在哪里的？

**Offset** 通常存储在一个名为 `__consumer_offsets` 的特殊 Kafka 主题中。这个主题就像一个图书馆管理员，帮你记住每个消费者组的阅读进度📚。这使得 Kafka 内部能够高效地管理和检索偏移量信息。

##### 那么主题是存储在哪里的？📁

Kafka 的主题和它们的分区实际上存储在 Kafka 集群的各个 **broker**（即 Kafka 服务器）上。每个分区对应一个或多个日志文件，这些日志文件存储在 broker 的本地文件系统中。Kafka 使用高效的存储和检索机制来确保消息的快速读写和持久化。具体来说：

1.  **Broker**：Kafka 集群中的每个服务器称为一个 broker。主题的分区会分布在多个 broker 上，以实现负载均衡和高可用性。
2.  **分区**：每个主题被划分成多个分区，分区是 Kafka 的并行单元。分区内的消息按顺序存储。
3.  **日志文件**：每个分区的数据存储在 broker 的本地文件系统中的一系列日志文件中。这些文件按消息的生产顺序进行追加写入。

这样设计的好处是，即使某个 broker 发生故障，只要有其他 broker 存在副本，Kafka 就能确保数据的可靠性和一致性🛠️。

#### 什么是 `__consumer_offsets`？

`__consumer_offsets` 是 Kafka 用来存储消费者组偏移量信息的内部主题。它是消费者提交 offset 的地方，就像一个专门为书签设计的图书馆📚。

#### OffsetCommitRequest 的过程 🚀

当消费者读取消息后，它会发送一个 `OffsetCommitRequest` 请求给 Kafka 服务器。这个请求包含了消费者组 ID、主题、分区和对应的 offset 信息。这就像你告诉图书馆管理员你读到了哪一页📄。

#### OffsetCommitResponse 的过程 ✉️

Kafka 服务器收到 `OffsetCommitRequest` 后，会处理并返回一个 `OffsetCommitResponse`，告诉消费者提交是否 **成功** ✅ 或 **失败** ❌。这相当于图书馆管理员确认你的书签是否被正确地记录下来。

#### 查看 `__consumer_offsets` 中的消息 🕵️‍♂️

可以使用 Kafka 自带的工具 `kafka-console-consumer` 来查看 `__consumer_offsets` 主题中的消息。例如：

```sh
kafka-console-consumer --bootstrap-server <kafka-broker>:9092 --topic __consumer_offsets --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --from-beginning

```

这条命令会从头开始读取并格式化显示 `__consumer_offsets` 主题中的偏移量信息，让你清晰地看到每个消费者组的阅读进度📊。

#### Offset 要提交到 `__consumer_offsets` 的哪个分区？🤔

Kafka 内部通过消费者组 ID 的哈希值来确定偏移量提交到 `__consumer_offsets` 主题的哪个分区。具体过程如下：

1.  对消费者组 ID 进行哈希运算。
2.  将哈希值对 `__consumer_offsets` 主题的分区数量取模。
3.  结果就是偏移量将要提交到的分区编号。

这就像图书馆管理员根据你的读者 ID 将书签存放在特定的文件夹中📂，确保每次都能准确找到你的阅读进度。