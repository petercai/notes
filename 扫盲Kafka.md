# 扫盲Kafka
### **为什么要使用 Kafka 消息队列？**

解耦、削峰：传统的方式上游发送数据下游需要实时接收，如果上游在某些业务场景：例如上午十点会流量激增至顶峰，那么下游资源可能会扛不住压力。但如果使用消息队列，就可以将消息暂存在消息管道中，下游可以按照自己的速度逐步处理；

可扩展：通过横向扩展生产者、消费者和broker, Kafka可以轻松处理巨大的消息流；

高吞吐、低延迟：在一台普通的服务器上既可以达到10W/s的吞吐速率；

容灾性：kafka通过副本replication的设置和leader／follower的容灾机制保障了消息的安全性。

### **kafka的高吞吐、低延迟是如何实现的？**

#### **1.顺序读写**

Kafka使用磁盘顺序读写来提升性能

顺序读写和随机读写性能对比：

|   | 顺序读 | 随机读 | 顺序写 | 随机写 |
| --- | --- | --- | --- | --- |
| 机械硬盘 | 84.0MB/s | 0.033MB/s (512字节) | 79.0MB/s | 0.083MB/s (512字节) |
| 固态硬盘 | 220.7MB/s | 5.296MB/s(512字节) | 77.2MB/s | 10.203MB/s(512字节) |

从数据可以看出磁盘的顺序读写速度远高于随机读写的速度，这是因为**传统的磁头探针结构**，随机读写时需要**频繁寻道**，也就需要磁头和探针频繁的转动，而机械结构的磁头和探针的位置调整是十分费时的，这就严重影响到硬盘的寻址速度，进而影响到随机写入速度。

Kafka的message是不断追加到本地磁盘文件末尾的，而不是随机的写入，这使得Kafka写入吞吐量得到了显著提升 。每一个Partition其实都是一个文件 ，收到消息后Kafka会把数据插入到文件末尾。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aeef05f3e3d64cdd85c5d41b85de78f3~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=308&h=188&s=13834&e=png&b=fefefe)

#### **2.页缓存（pageCache）**

PageCache是系统级别的缓存，它把尽可能多的空闲内存当作磁盘缓存使用来进一步提高IO效率；

PageCache同时可以避免在JVM内部缓存数据，避免不必要的GC、以及内存空间占用。对于In-Process Cache，如果Kafka重启，它会失效，而操作系统管理的PageCache依然可以继续使用。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7ec45a554764c178ee5a2328bd7ef78~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1702&h=683&s=39286&e=png&a=1&b=fef5f3)

*   producer把消息发到broker后，数据并不是直接落入磁盘的，而是先进入PageCache。PageCache中的数据会被内核中的处理线程采用同步或异步的方式定期刷盘至磁盘。
*   Consumer消费消息时，会先从PageCache获取消息，获取不到才回去磁盘读取，并且会预读出一些相邻的块放入PageCache，以方便下一次读取。
*   如果Kafka producer的生产速率与consumer的消费速率相差不大，那么几乎只靠对broker PageCache的读写就能完成整个生产和消费过程，磁盘访问非常少

#### **3.零拷贝**

正常过程：

1.  操作系统将数据从磁盘上读入到内核空间的读缓冲区中
    
2.  应用程序（也就是Kafka）从内核空间的读缓冲区将数据拷贝到用户空间的缓冲区中
    
3.  应用程序将数据从用户空间的缓冲区再写回到内核空间的socket缓冲区中
    
4.  操作系统将socket缓冲区中的数据拷贝到NIC缓冲区中，然后通过网络发送给客户端
    

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97dff997617a4378a2c5124d43003597~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2000&h=671&s=187596&e=png&a=1&b=f2f2f2)

在这个过程中，可以发现， 数据从磁盘到最终发出去，要经历4次拷贝，而在这四次拷贝过程中， 有两次拷贝是浪费的。

1.从内核空间拷贝到用户空间；

2.从用户空间再次拷贝到内核空间；

除此之外，由于用户空间和内核空间的切换，会带来Cpu上下文切换，对于Cpu的性能也会造成影响；

零拷贝省略了数据在内核空间和用户空间之间的重复穿梭；用户态和内核态切换时产生中断，耗时；

#### **4.分区分段索引**

Kafka的message是按topic分类存储的，topic中的数据又是按照一个一个的partition即分区存储到不同broker节点。每个partition对应了操作系统上的一个文件夹，partition实际上又是按照segment分段存储的。符合分布式系统**分区分桶的设计思想**。

通过这种分区分段的设计，Kafka的message消息实际上是分布式存储在一个一个小的segment中的，每次文件操作也是直接操作的segment。为了进一步的查询优化，Kafka又默认为分段后的数据文件建立了索引文件，就是文件系统上的.index文件。这种分区分段+索引的设计，不仅提升了数据读取的效率，同时也提高了数据操作的并行度。

#### **5.批量处理**

kafka发送消息不是一条一条发送的，而是批量发送，很大的提高了发送消息的吞吐量。

假设发送一条消息的时间是1ms，而此时的吞吐量就是1000TPS。但是假如我们将消息批量发送，1000条消息需要10ms，而此时的吞吐量就达到了1000*100TPS。而且这样也很大程度的减少了请求Broker的次数，提升了总体的效率。

### **基本概念**

| 名词 | 概念 |
| --- | --- |
| **Producer** | 生产者（发送消息） |
| **Consumer** | 消费者（接收消息） |
| **ConsumerGroup** | 消费者组，可以并行消费同一topic中的消息 |
| **Broker** | 一个独立的kafka服务器被称为broker。broker接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。broker为消费者提供服务，对读取分区的请求作出相应，返回已经提交到磁盘上的消息。可起到负载均衡、容错的作用。 |
| **Topic** | 主题，一个队列，可理解为按照消息的逻辑分类将消息划分为不同的topic |
| **Partition** | topic的物理分组，一个topic可以分为多个partition，每个partition是一个有序队列。可起到提高可扩展性，应对高并发场景的作用。 |
| **replica** | 副本，为保证集群的高可用性，kafka提供副本机制，一个topic的每个分区都有若干个副本，一个leader和若干个follower |
| **leader** | 每个分区多个副本的主节点，生产者发送数据的对象，以及消费者消费数据的对象都是leader |
| **offset** | 对于Kafka中的分区而言，它的每条消息都有唯一的offset，用来表示消息在分区中对应的位置。 |

### **架构图**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06b6a9ab76d44741a7da291f807133f4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1160&h=720&s=139944&e=png&b=fefefe)

**Q1：Topic的分区及副本在broker上是如何分配的呢？**

这里涉及到两个参数：

**startIndex**：第一个分区的第一个副本放置位置（P0-R0）

**nextReplicaShift**：其他分区的副本的放置是依次后移的，间隔距离就是 nextReplicaShift 值。

**Q2：Kafka的架构是基于什么设计思想呢？**

分治思想：

**1\. topic分治**：对于kafka的topic，我们在创建之初可以设置多个partition来存放数据，对于同一个topic的数据，每条数据的key通过哈希取模被路由到不同的partition中（如果没有设置key，则根据消息本身取模），以此达到分治的目的。

**2\. partition分治**：为了方便数据的消费，kafka将原始的数据转化为”索引+数据“的形式进行分治，将一个partition对应一个文件转变为一个partition对应多个人不同类型的文件，分别为：

*   index文件：索引文件，用来记录log文件中真实消息的相对偏移量和物理位置，为了减少索引文件的大小，使用了稀疏索引
*   log文件：用来记录producer写入的真实消息，即消息体本身；
*   timeindex文件：时间索引文件，类比.index文件，用来记录log文件中真实消息写入的时间情况，跟offset索引文件功能一样，只不过这个以时间作为索引，用来快速定位目标时间点的数据；

**3\. 底层文件分治**：不能将partition全部文件都放入一套 ”.index+.log+.timeindex“ 文件中，因此需要对文件进行拆分。kafka对单个.index文件、.timeindex文件、.log文件的大小都有限定（通过不同参数配置），且这3个文件互为一组。当.log文件的大小达到阈值则会自动拆分形成一组新的文件，这种将数据拆分成多个的小文件叫做segment，一个log文件代表一个segment。

### **生产流程：** 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf4516fb93fe4452b8f5549ba12524d4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1763&h=804&s=358527&e=png&b=ffffff)

1.  先从zk获取对应分区的leader在哪个broker
    
2.  broker进程上的leader将消息写入到本地log中
    
3.  follower从leader上拉取消息，写入到本地log，并向leader发送ACK
    
4.  leader接收到所有的ISR中的Replica的ACK后，并向生产者返回ACK
    

### **消费流程：** 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9cd7d924d0f46dba483d4de76fd94dc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1282&h=801&s=247990&e=png&b=ffffff)

1.  每个consumer都可以根据分配策略，获得要消费的分区
    
2.  获取到consumer对应的leader处于哪个broker以及offset
    
3.  拉取数据
    
4.  消费者提交offset
    

相信上面的内容已经让大家大致了解了消息生产及消费的过程：一个topic内的消息会被发送到不同的分区以供不同的消费者拉取消息。

那么在这个过程中就涉及到了两个问题：

1.  生产者按照什么策略将数据分配到分区中呢？
    
2.  消费者按照什么策略去不同的分区拉取消息呢？
    

### **生产者分区策略**

生产者写入消息到topic，Kafka将依据不同的策略将数据分配到不同的分区中：

#### **1\. 轮询分区策略**

即按消息顺序进行分区顺序分配，是默认的策略，也是使用最多的策略，可以最大限度保证所有消息平均分配到一个分区；

key为null，则使用轮询算法均衡地分配分区；

#### **2\. 按key分区分配策略**

key不为null，key.hash() % n

但是按照key决定分区有可能会造成数据倾斜

#### **3\. 随机分区策略**

随机分区，不建议使用

#### **4\. 自定义分区策略**

根据业务需要制定以分区策略

> 乱序问题在Kafka中生产者是有写入策略，如果topic有多个分区，就会将数据分散在不同的partition中存储当partition数量大于1的时候，数据（消息）会打散分布在不同的partition中如果只有一个分区，消息是有序的

### **消费者分区策略**

同一时刻，一条消息只能被组中的一个消费者实例消费：

1.  消费者数=分区数：一个分区对应一个消费者
    
2.  消费者数<分区数：一个消费者对应多个分区
    
3.  消费者数>分区数：多出来的消费者将不会消费任何消息
    

分区分配策略：保障每个消费者尽量能够均衡地消费分区的数据，不能出现某个消费者消费分区的数量特别多，某个消费者消费的分区特别少

**1\. Range分配策略（范围分配策略）**：Kafka默认的分配策略

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1476f0c2e8324766894b9cca474cd60d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=759&h=614&s=46977&e=png&b=ffffff)

计算公式：

n=分区数量/消费者数量

m=分区数量%消费者数量

前m个消费者消费n+1个，剩余消费者消费n个

以上图为例：n=8/3=2m=8%3=2因此前2个消费者消费2+1=3个分区，剩下1个消费者消费2个分区

**2\. RoundRobin分配策略**（轮询分配策略）

消费者挨个分配消费的分区：

如下图，3个消费者共同消费8个分区

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/853477007d874aada7d6a696db1721c6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=887&h=649&s=64703&e=png&b=ffffff)

第一轮：Consumer0-->A-Partition0；Consumer1-->A-Partition1；Consumer2-->A-Partition2

第二轮：Consumer0-->A-Partition3；Consumer1-->B-Partition0；Consumer2-->B-Partition1

第三轮：Consumer0-->B-Partition2；Consumer1-->B-Partition3

**3\. Striky粘性分配策略**

在没有发生rebalance跟轮询分配策略是一致的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe630d91b1fb41c7ac2798295cfa63ec~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=887&h=649&s=64703&e=png&b=ffffff)

发生了rebalance（例如Consumer2故障宕机），轮询分配策略，重新走一遍轮询分配的过程。而粘性会保证跟上一次的尽量一致，只是将新的需要分配的分区，均匀的分配到现有可用的消费者中即可，这样就减少了上下文的切换

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c40839226de448d975a702ea8ff9210~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2021&h=661&s=136556&e=png&b=ffffff)

producer是不断地往Kafka中写入数据，写入数据会有一个返回结果，表示是否写入成功。这里对应有一个ACKs的配置。

*   acks = 0：生产者只管写入，不管是否写入成功，可能会数据丢失。性能是最好的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/642274aaed6843dfb262a563e30d8aae~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1037&h=478&s=97779&e=png&b=fdfdfd)

*   acks = 1：生产者会等到leader分区写入成功后，返回成功，接着发送下一条

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c583e93f9c304848b908558e98f6eff7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1029&h=471&s=96510&e=png&b=fcfcfc)

*   acks = -1/all：确保消息写入到leader分区、还确保消息写入到对应副本都成功后，接着发送下一条，性能是最差的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79ad48fdd0a04f57b114c065a1ef74e8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=978&h=458&s=103723&e=png&b=fcfcfc)

根据业务情况来选择ack机制，是要求性能最高，一部分数据丢失影响不大，可以选择0/1。如果要求数据一定不能丢失，就得配置为-1/all。

分区中是有leader和follower的概念，为了确保消费者消费的数据是一致的，只能从分区leader去读写消息，follower做的事情就是同步数据。

**1\. offset存在哪里？**

0.9版本前默认存在zk，但是由于频繁访问zk，zk需要一个一个节点更新offset，不能批量或分组更新，导致offset更新成了瓶颈。

在新版 Kafka 以及之后的版本，Kafka 消费的offset都会默认存放在 Kafka 集群中的一个叫 \_\_consumer\_offsets 的topic中。offset以消息形式发送到该topic并保存在broker中。这样consumer提交offset时，只需连接到broker，不用访问zk，避免了zk节点更新瓶颈。

**2.leader选举策略？**

2种leader：①broker的leader即controller leader ② partition的leader

1.  Controller leader：当broker启动的时候，都会创建KafkaController对象，但是集群中只能有一个leader对外提供服务，这些每个节点上的KafkaController会在指定的zookeeper路径下创建临时节点，只有第一个成功创建的节点的KafkaController才可以成为leader，其余的都是follower。当leader故障后，所有的follower会收到通知，再次竞争在该路径下创建节点从而选举新的leader
    
2.  Partition leader ：由controller leader执行
    

*   从Zookeeper中读取当前分区的所有ISR(in-sync replicas)集合
*   调用配置的分区选择算法选择分区的leader

> 如何处理所有Replica都不工作？在ISR中至少有一个follower时，Kafka可以确保已经commit的数据不丢失，但如果某个Partition的所有Replica都宕机了，就无法保证数据不丢失了。这种情况下有两种可行的方案：等待ISR中的任一个Replica“活”过来，并且选它作为Leader选择第一个“活”过来的Replica（不一定是ISR中的）作为Leader这就需要在可用性和一致性当中作出一个简单的折衷。如果一定要等待ISR中的Replica“活”过来，那不可用的时间就可能会相对较长。而且如果ISR中的所有Replica都无法“活”过来了，或者数据都丢失了，这个Partition将永远不可用。选择第一个“活”过来的Replica作为Leader，而这个Replica不是ISR中的Replica，那即使它并不保证已经包含了所有已commit的消息，它也会成为Leader而作为consumer的数据源（前文有说明，所有读写都由Leader完成）。Kafka0.8.*使用了第二种方式。根据Kafka的文档，在以后的版本中，Kafka支持用户通过配置选择这两种方式中的一种，从而根据不同的使用场景选择高可用性还是强一致性。

**3.分区可以提高扩展性以及吞吐量，那分区越多越好吗？**

分区并不是越多越好，分区过多存在着以下弊端：

*   producer内存成本增大、consumer线程开销增大partition是kafka管理数据的基本逻辑组织单元，越多的partition意味着越多的数据存储文件（一个partition对应至少3个数据文件）
*   分区增多数据连续性下降
*   可用性降低

**4.与数据库相比kafka的优势？**

*   业务场景不同，底层数据结构不同，kafka数据存储对于功能要求较少，因此读写更快kafka存储数据（消息本身）的文件的数据结构是数组，数组的特点就是：数据间位置连续，如果按照顺序读取，或者追加写入的话，其时间复杂度为O(1)，效率最高。

**5.消费偏移的更新方式** 无论是kafka默认api，还是java的api，offset的更新方式都有两种：自动提交和手动提交

1.  自动提交（默认方式）

Kafka中偏移量的自动提交是由参数enable\_auto\_commit和auto\_commit\_interval\_ms控制的，当enable\_auto\_commit=True时，Kafka在消费的过程中会以频率为auto\_commit\_interval\_ms向Kafka自带的topic(\_\_consumer\_offsets)进行偏移量提交，具体提交到哪个Partation：Math.abs(groupID.hashCode()) % numPartitions。

这种方式也被称为at most once，fetch到消息后就可以更新offset，无论是否消费成功。

2\. 手动提交

鉴于Kafka自动提交offset的不灵活性和不精确性(只能是按指定频率的提交)，Kafka提供了手动提交offset策略。手动提交能对偏移量更加灵活精准地控制，以保证消息不被重复消费以及消息不被丢失。

对于手动提交offset主要有3种方式：

同步提交：提交失败的时候一直尝试提交，直到遇到无法重试的情况下才会结束，同步方式下消费者线程在拉取消息会被阻塞，在broker对提交的请求做出响应之前，会一直阻塞直到偏移量提交操作成功或者在提交过程中发生异常，限制了消息的吞吐量。

异步提交：异步手动提交offset时，消费者线程不会阻塞，提交失败的时候也不会进行重试，并且可以配合回调函数在broker做出响应的时候记录错误信息。 对于异步提交，由于不会进行失败重试，当消费者异常关闭或者触发了再均衡前，如果偏移量还未提交就会造成偏移量丢失。

异步+同步：针对异步提交偏移量丢失的问题，通过对消费者进行异步批次提交并且在关闭时同步提交的方式，这样即使上一次的异步提交失败，通过同步提交还能够进行补救，同步会一直重试，直到提交成功。 通过finally在最后不管是否异常都会触发consumer.commit()来同步补救一次，确保偏移量不会丢失

参考资料：

[www.jianshu.com/p/90a15fe33…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F90a15fe33551 "https://www.jianshu.com/p/90a15fe33551")

[www.jianshu.com/p/da3f19cee…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fda3f19cee0e2 "https://www.jianshu.com/p/da3f19cee0e2")

[www.modb.pro/db/373502](https://link.juejin.cn/?target=https%3A%2F%2Fwww.modb.pro%2Fdb%2F373502 "https://www.modb.pro/db/373502")

[blog.csdn.net/anryg/artic…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fanryg%2Farticle%2Fdetails%2F123579937 "https://blog.csdn.net/anryg/article/details/123579937")

[toutiao.io/posts/ptwuh…](https://link.juejin.cn/?target=https%3A%2F%2Ftoutiao.io%2Fposts%2Fptwuho%2Fpreview "https://toutiao.io/posts/ptwuho/preview")
