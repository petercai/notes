# Kafka架构设计深入解析
kafka架构设计包括生产者设计，服务端设计（包括元数据管理，副本一致性等），消费者架构设计。首先，看一下整体架构：

整体架构
----

消息从发送到消费的整体流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/851dacc0d48f429fa784fda9d82b26de~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1159&h=926&s=97587&e=png&b=e6ebed)

1.  Producer 根据设置的集群节点信息随机取一个节点来获取 topic A 的路由信息。
    
2.  根据获取的路由信息 producer 向 broker 1 和broker2 发送信息。
    
3.  Consumer 根据设置的集群节点信息随机取一个节点来获取 topic A 的路由信息和 coordinator节点地址。
    
4.  Consumer 访问 coordinator 节点地址获取要消费的partition 的offset。
    
5.  Consumer 访问 broker1 和 broker2 并根据获取的partition offset 消费消息。
    

生产者架构设计
-------

生产者为了保证高吞吐量，设计上使用了批量发送，异步发送等设计。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb7d115294cf43c4a707741d61739f1b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=886&h=845&s=69549&e=png&a=1&b=ffb266)

*   生产者分为两个线程，分别为业务主线程和真正网络发送的sender线程。业务主线程把消息发送到本地缓存中就直接返回了（可以通过listening监听发送结果），真正发送信息的是sender线程。
*   RecordAccumulator 是本地缓存的容器，数据结构为map<partition,deque>,一个元素代表要往某个分区发送的消息，发送的最小单位不是一个消息，而是消息集合，也就是批量发送。
*   Sender 会根据发送规则从 RecordAccumulator 拿到发送数据，然后根据 broker Node节点再次排队发送。
*   发送成功后会调用业务主线程的 listener，这样主线程就能知道发送是否成功，然后判断是否重发。

元数据管理
-----

元数据指broker信息，controller信息，toptic信息，消费组信息

### 方式1：Zookeeper 管理元数据

0.9之后 2.6 之前的方案是通过下图的 方案来管理元数据。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/738f3ec25be547bb9a4a502e196fad5b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=733&h=415&s=33123&e=png&a=1&b=dbf5f5)

*   controller broker 通过选举产生。
*   元数据由controller broker 持久化到 zookeeper 作为数据备份。
*   其他非 controller broker 节点会主动从 controller broker 拉取元数据
*   客户端请求元数据（如主题路由信息），只要随机访问一个broker 就能获得。

好处：只有 controller 负责和 zookeeper 交互，减少了 zookeeper 同步的客户端，避免了惊群现象。

坏处：仍然要把主题分区信息放入zookeeper,当有大量分区的时候，如果发生了 controller 节点崩溃，新的controller 节点选出后，会有大量的数据拷贝问题，这个过程中服务是停的。

Zookeeper 是强一致组件，当大多数节点同步到数据才认为数据改动成功，造成很大的延迟。

如果 zookeeper 节点挂的过多会造成脑裂，最终停服。

### 方式2：集群内部通过 raft 分布式协调机制管理元数据(Kafka 3.3.1及以后版本这个特性可上生产)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c3959ba4eaf4381a3395ff6056aee1a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=802&h=549&s=49929&e=png&b=daf4f4)

副本数据一致性
-------

### ISR

Follower 副本同步leader副本的时候，既要考虑到一致性，也要考虑到吞吐量。

分区中所有的副本称为AR（Assigned Replicas）。所有与leader副本保持一定程度同步的副本（包括leader副本）组成ISR（In-Sync Replicas），也就是说 **ISR 是 AR 的一个子集**。

所有与leader副本保持一定程度同步，同步期间内follower副本相对于leader副本而言会有一定程度的滞后。所谓“一定程度的同步”是指一定时间范围内从leader拉取消息成功的follower节点都可以进入ISR，这个范围可以通过参数进行配置默认是10s。leader副本负责维护和跟踪ISR集合中所有follower副本的滞后状态，当follower副本落后太多或失效时，leader副本会把它从ISR集合中剔除。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca738b54667841768d7d87960a6f7784~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=535&h=394&s=21152&e=png&b=f6edc1)

*   服务节点分为两类：controller 和 broker：controller 负责管理元数据，主题分区路由，broker节点信息等
*   controller 集群通过raft算法选出leader, oberver 从 leader同步元数据，如果leader 挂了会重新选举。
*   当客户端要获取元数据时，通过访问controller leader 节点可以获得，同时broker 也会同步必要的元数据。

### HW

HW（highwater mark）：高水位。**作用是大于等于 HW 的offset消息对于消费者是不能消费的**。高水位由leader副本负责管理，高水位的取值是所有副本（leader副本和follower副本）LEO的最小值。下图反映了HW和LEO的关系。leader副本负责更新HW，会选取三个副本中最小的LEO作为HW，同时follower副本在向leader副本拉取消息时，会把HW也返回给Follower，Follower副本也会维持一个HW的值，目的是防止leader挂了以后，follower选上leader时有HW可用。

### follower 什么时候会得到要更新的 HW 呢？

Kafka 中的复制协议大体有两个阶段。第一阶段：follower副本从leader副本同步数据，它取到了偏移量=4这条消息。第二阶段：在下一轮的RPC调用中follower会确认收到了偏移量=4这条消息，假定其他的follower副本也都确认成功收到了这条消息，Leader副本才会更新其高水位HW，并且会在follower再次从Leader副本同步获取数据的时候把这个高水位值放在请求响应中回传给follower副本。由此可以看出，leader副本控制着高水位HW的进度，并且会在随后的RPC调用中回传给follower副本。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2698ef7079846d7a77b7c617536ceb6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=680&h=434&s=18100&e=png&a=1&b=ffffff)

如上图所示，一共有5个步骤。

第一步：初始阶段，这时候leader和follower的LEO和HW都指向LEO=4的位置。

第二步：这是生产者向Leader发送了一个偏移量为4的消息，于是LEO更新为5，而HW还是4，因为follower的LEO并没有发生改变。

第三步：这时候follower定时发出请求要拉取LEO=4的消息，follower收到消息后把LEO改为5。

第四步：follower再次定时发出请求要拉取偏移量为5的消息。leader知道follower把LEO=4的消息更新成功了，于是leader的HW为5。

第五步：Leader没有偏移量为5的消息但是把HW=4返回给了follower，但是于是follower的HW改为5。

这时候，Leader和Follower完全同步完成。

你可能发现了，Leader的HW=4是在follower发出拉取LEO=5的请求时才更新的。为什么要这样设计呢？

### 为什么 Leader 的 HW 更新是在下一次 Follower 拉取 Leader 的请求时更新的？

原因是为了**节省网络开销**，因为消息系统的并发量很高，如果follower副本每次拉取leader副本都还要给leader副本返回拉取是否成功的响应，那么网络开销太高了。但是Leader必须要知道Follower是否同步成功到哪个偏移量了，这样才能更新HW。于是做了个折中的方案：**在下次拉取请求的信息里面带上上次拉取成功的信息**。比如上图中的第四步拉取LEO=5的请求Leader收到后会知道偏移量=4的消息follower已经成功接收到了，于是更新HW。

如果HW和LEO不一致的Follower被选为leader会发生什么？

Follower会根据HW来截断日志文件，一般LEO会大于HW。为什么要截断呢？因为Follower的信息有限，根本无法判断其他的follower的LEO也更新了，除了从leader获得HW根本没有别的办法确定，如果不截断会造成与其他副本数据不一致。也就是说，HW也是用来保证数据一致性的。

也就是说Leader和Follower的HW同步有一个间隙，这样会造成什么问题呢？

#### 造成的问题一：丢失消息

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fed5834fed9144ec8dc6d3bd11951f79~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1479&h=1191&s=74757&e=png&a=1&b=fefefe)

第一步：也就是Follower要带着LEO=5的请求去拉取消息时，Leader更新了HW，但这时还没返回给Follower新的HW。

第二步：Follower重启，会根据HW截断日志文件。

第三步：Leader也挂了，原来的Follower经过选举成为了新的Leader。

第四步：这时原来的Leader又恢复了，成为了新的Follower，这时发现比Leader的HW高，所以又进行了一次截断。这样偏移量=4的消息就没了。

总结：不该截断的时候截断了。

截断会发生在两个场景：

1.  follower副本重启发现HW和LEO不一致时，截断大于等于HW的日志。
2.  follower副本从Leader副本拉取Leader的HW比follower副本上的HW小，截断大于等于Leader的HW的日志。

#### 造成的问题二：消息错乱

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1fa90a630d5479b9e9f85b5d429a0e2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=999&h=1882&s=91948&e=png&a=1&b=fefefe)

第一步：生产者向leader副本发送偏移量为4的消息m4，同时Follower副本来拉取消息，leader收到出Follower要拉取偏移量为4的消息，于是把HW设置为5。

第二步：两个机器同时挂了，Follower并没有收到移量为4的消息m4。

第三步：两个机器同时重启，Follower先恢复这时leader还没恢复，Follower变为Leader，同时生产者向新的Leader发送了偏移量为4的消息m5。

第四步：这时候原来的Leader变为Follower，然后做截断，但是HW=LEO就不做任何截断。

这时候两个副本间的数据就不一致了。

总结：该截断的时候没有截断。

#### 解决方案：引入 leader epoch

**所谓leader epoch实际上是一对值：（epoch，offset）** 。epoch表示leader的版本号，从0开始，当leader变更过1次时epoch就会+1，而offset则对应于该epoch版本的leader写入第一条消息的位移。因此假设有两对值：

*   (0, 0)
*   (1, 120)

则表示第一个leader从位移0开始写入消息；共写了120条\[0, 119\]；而第二个leader版本号是1，从位移120处开始写入消息。

leader broker中会保存这样的一个缓存，并定期地写入到一个checkpoint文件中。当leader写底层log时它会尝试更新整个缓存——如果这个leader首次写消息，则会在缓存中增加一个条目；否则就不做更新。而每次副本重新成为leader时会查询这部分缓存，获取出对应leader版本的位移，这就不会发生数据不一致和丢失的情况。

在场景1中，当follower重启以后，它会向leader发送一个LeaderEpochRequest请求，来获取自身所处的leader epoch最新的LEO是多少，因为follower和Leader所处的时代相同（leader epoch编码都是0），Leader会返回自己的LEO，也就是5给follower副本。请注意，与高水位不同的是，follower副本上的offset值是0，follower副本不会截断任何消息，m2得以保留不会丢失。当followerA选为leader的时候就保留了所有已提交的日志，日志丢失的问题得到解决。

在场景2中，开始的时候副本A是leader副本A，当两个broker在崩溃重启后，brokerB先成功重启，follower副本B 成为Leader副本B。它会开启一个新的领导者纪元LE1，开始接受消息 m3。然后brokerA又成功重启，此时副本A很自然成为follower副本A，接着它会向leader B发送一个LeaderEpoch request请求，用来确定自己应该处于哪个领导者时代，leader B会返回LE1时代的第一个位移，这里返回的值是1（也就是m3所在的位移）。follower B收到这个响应以后会根据这个位移1来截断日志，它知道了应该遗弃掉m2，从位移1开始同步获取日志。

消费者管理：
------

涉及服务端的**GroupCoordinator 和 消费端的ConsumerCoordinators**

**客户端** **ConsumerCoordinators 是和服务端 GroupCoordinator 是配合使用的，目的是管理消费者**，包括通过管理消费者上下线，给消费者分发要消费的主题分区。不过，我在介绍消费者协调器之前，先解释几个概念。

### offset 管理

每一个consumer客户端内存都会保存它消费的每个主题分区的消费offset，`消费 offset 表示这个 onsumer group 消费到这个主题分区的哪个 offset 了`。

为什么要有消费 offset 呢？当某个消费者上下线时会造成主题分区的消费换到别的消费者，而新的消费者不知道主题分区消费到哪里了，这时就需要服务端保存消费 offset，这样新的消费者才能得到消费的 offset 接着消费，不会造成重复消费和漏掉消费的情况。

也就是说消费者会定期向服务端提交offset，老版本是写入ZooKeeper，显然消费者提交offset是个高频操作。而ZooKeeper是做分布式协调的，轻量级元数据存储，ZooKeeper不适合高并发的请求。

后来的版本放弃 ZooKeeper 做消费offset的保存而改用内部主题`__consumer_offsets`，主题的key是`group.id+topic+分区号`，value就是当前消费的offset值。每隔一段时间`__consumer_offsets`会把key重复的历史的数据删除，也就是说只保留最新的一条key值。内部主题`__consumer_offsets`的分区数是50，这样就能很好地抵挡高并发的请求。

**什么是 GroupCoordinator ？**

每个消费者组都会选择一个broker作为自己的 GroupCoordinator，GroupCoordinator `负责监控`消费者组里各个消费者的心跳，判断是否宕机，然后开启 rebalance 把分区分配给各个消费者。

消费组中的消费者刚启动的时候，就会跟对应 GroupCoordinator 的 broker 建立通信，GroupCoordinator 会分配主题分区给这个消费者消费。GroupCoordinator会尽量`均匀`地分配分区给各个消费者进行消费。

**那如何找到 GroupCoordinator 呢？**

每个消费者都有一个消费者组id，消费者首先对消费者组id进行hash，hash后的值对`__consumer_offsets`的分区数取模得到要发送的分区。消费者向集群中一个节点发送请求，节点根据消费者提供消费组id和节点缓存的元数据得到这个分区的leader所在的broker，这个broker上运行的 **GroupCoordinator** 负责接收和管理对应消费者组提交的消费offset。

### Rebalance 消费者重平衡

ConsumerCoordinators和GroupCoordinator之间最重要的职责就是**负责执行消费者重平衡的操作**。消费者重平衡是指在分区或消费者有变动的时候，需要重新给消费者内的消费者分配要消费的分区。

好，接着我来给你介绍下 ConsumerCoordinator 和 GroupCoordinator 的工作流程。

### 协调者工作流程

如下图所示，Kafka 集群有 3 个节点，同一个消费者组 1 下有 3 个消费者去消费 topic。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b89ed571abd4d25836d6339f3a13188~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1015&h=461&s=45684&e=png&a=1&b=fea641)

如果想详细学习请看 掘金小册[《Kafka 源码精讲》](https://juejin.cn/book/7017291709167960064 "https://juejin.cn/book/7017291709167960064") 你将获得：

*   全面学 Kafka 各个组件，系统掌握 Kafka 的原理；
*   系统学习 Kafak 的轮子组件，如内存缓冲池，基于 NIO 的通信模块；
*   Kafka 高可靠分布式设计；
*   Kafka 底层文件存储设计；