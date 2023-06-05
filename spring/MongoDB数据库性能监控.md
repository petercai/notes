# MongoDB数据库性能监控
![](https://s6.51cto.com/oss/202302/16/b23637312f8a4d49c43598c2e3df8140f5c0ac.png)

> 最近项目在使用MongoDB作为图片和文档的存储数据库，为啥不直接存MySQL里，还要搭个MongoDB集群，麻不麻烦？
> 
> 让我们一起，一探究竟，继续学习MongoDB数据库性能监控，实现快速入门，丰富个人简历，提高面试level，给自己增加一点谈资，秒变面试小达人，BAT不是梦。

一、MongoDB启动慢​
-------------

#### 1、启动日常卡住

根本不用为了截屏而快速操作，MongoDB启动真的超级慢。

![](https://s3.51cto.com/oss/202302/16/723ca49994003500662735b3f284a389b25657.png)

![](https://s9.51cto.com/oss/202302/16/782cb0583e0d7d2b629568096168347adf03bd.png)

#### 2、启动MongoDB配置服务器，间歇性失败

![](https://s6.51cto.com/oss/202302/16/f9a9ed984b12e826ba97298c8486ae2a4746c8.jpg)

![](https://s4.51cto.com/oss/202302/16/43a48cc76efd7912a219923045b0dbfbafc0bb.jpg)

#### 3、查看MongoDB日志，分析“MongoDB启动慢”的原因

![](https://s7.51cto.com/oss/202302/16/513483269453f82f07b519d5be063c26a1f380.png)

#### 4、耗时“一小时”，MongoDB启动成功！

![](https://s7.51cto.com/oss/202302/16/e14423c41fee5f733b9432470851f79008862a.jpg)

二、原因分析
------

在MongoDB关闭之前，有较大的索引建立的操作没有完成，MongoDB就直接shutdown了，等MongoDB再次启动的时候，MongoDB默认会将这个index重建好，重建期间处于startup状态。

由于不清楚重建索引需要多久，因此可以通过重启mongod时加上–noIndexBuildRetry参数来跳过索引重建。等启动完成后，再创建这个索引。

> 下面从几方面，监控一下MongoDB的性能问题。

三、MongoDB内存使用
-------------

![](https://s3.51cto.com/oss/202302/16/2635bec938005dfca33521639a85d8856467ca.png)

*   **常驻内存：** 常驻内存是MongoDB在RAM中显式拥有的内存。如果查询一个集合数据，MongoDB会将其放入常驻内存中，MongoDB会获得其地址，这个地址不是RAM中数据的真实地址，而是一个虚拟地址。MongoDB可以将它传递给内核，内核会查找出数据的真实位置。如果内核需要从内存中清理缓存，MongoDB仍然可以通过该地址对其进行访问。MongoDB会向内核请求内存，然后内核会查看数据缓存，如果发现数据不存在，就会产生缺页错误并将数据复制到内存中，最后再返给MongoDB。
*   **虚拟内存：** 操作系统提供的一种抽象，它对软件进程隐藏了物理存储的细节。每个进程都可以看到一个连续的内存地址空间。在Ops Manager中，MongoDB的虚拟内存是映射内存的两倍。
*   **映射内存：** 包含MongoDB曾经访问过的所有数据。

四、MongoDB磁盘空间​
--------------

当磁盘空间不足时，可以进行如下操作：

1.  可以添加一个分片；
2.  删除未使用的索引；
3.  可以执行压缩操作；
4.  关闭副本集成员，将其数据复制到更大的磁盘中挂载;
5.  用较大驱动器的成员替换副本集中的成员；

五、常用命令
------

#### 1、MongoDB获取系统信息

![](https://s5.51cto.com/oss/202302/16/e1c872780a23d67c8695995edb0623775a77bc.png)

#### 2、MongoDB获取系统内存情况

![](https://s9.51cto.com/oss/202302/16/f2b49e620547a98609b9662b730d3143788bf7.png)

#### 3、MongoDB获取连接数信息

```
`db.serverStatus().connections`

*   1.


```

![](https://s7.51cto.com/oss/202302/16/421eab174bc24140b27810747b5def4a7642ac.png)

#### 4、MongoDB获取全局锁信息

```
`db.serverStatus().globalLock`

*   1.


```

![](https://s9.51cto.com/oss/202302/16/7711945594925cb1019089aa61be03fbe0f980.png)

#### 5、MongoDB获取操作统计计数器

```
`db.serverStatus().opcounters`

*   1.


```

![](https://s9.51cto.com/oss/202302/16/58fac1a32bbd281fe4d33476c4daf30fc2e660.png)

#### 6、MongoDB获取数据库状态信息

![](https://s4.51cto.com/oss/202302/16/26bb05708f7895c289e756cbaa61ae57ace4a7.png)

以上是MongoDB的重要指标，通过这些指标我们可以了解到MongoDB的运行状态，评估数据库的健康程度，并快速确定实际项目中遇到的性能瓶颈。

比如项目中遇到的Timeout异常：

```
`com.mongodb.MongoSocketReadTimeoutException: Timeout while receiving message
  at com.mongodb.connection.InternalStreamConnection.translateReadException(InternalStreamConnection.java:475)
  at com.mongodb.connection.InternalStreamConnection.receiveMessage(InternalStreamConnection.java:226)
  at com.mongodb.connection.UsageTrackingInternalConnection.receiveMessage(UsageTrackingInternalConnection.java:105)
  at com.mongodb.connection.DefaultConnectionPool$PooledConnection.receiveMessage(DefaultConnectionPool.java:438)
  at com.mongodb.connection.CommandProtocol.execute(CommandProtocol.java:112)
  at com.mongodb.connection.DefaultServer$DefaultServerProtocolExecutor.execute(DefaultServer.java:168)
  at com.mongodb.connection.DefaultServerConnection.executeProtocol(DefaultServerConnection.java:289)
  at com.mongodb.connection.DefaultServerConnection.command(DefaultServerConnection.java:176)
  at com.mongodb.operation.CommandOperationHelper.executeWrappedCommandProtocol(CommandOperationHelper.java:216)
  at com.mongodb.operation.CommandOperationHelper.executeWrappedCommandProtocol(CommandOperationHelper.java:207)
  at com.mongodb.operation.CommandOperationHelper.executeWrappedCommandProtocol(CommandOperationHelper.java:113)
  at com.mongodb.operation.FindOperation$1.call(FindOperation.java:488)
  at com.mongodb.operation.FindOperation$1.call(FindOperation.java:1)
  at com.mongodb.operation.OperationHelper.withConnectionSource(OperationHelper.java:241)
  at com.mongodb.operation.OperationHelper.withConnection(OperationHelper.java:214)
  at com.mongodb.operation.FindOperation.execute(FindOperation.java:483)
  at com.mongodb.operation.FindOperation.execute(FindOperation.java:1)
  at com.mongodb.Mongo.execute(Mongo.java:818)`

*   1.
*   2.
*   3.
*   4.
*   5.
*   6.
*   7.
*   8.
*   9.
*   10.
*   11.
*   12.
*   13.
*   14.
*   15.
*   16.
*   17.
*   18.
*   19.


```

六、MongoDB持久性​
-------------

#### 1、复制延迟

复制延迟是指从节点无法跟上主节点的速度。

从节点一个操作的时间减去主节点此操作的时间，就是复制延迟。延迟应该尽可能的接近0，并且通常是毫秒级的。

#### 2、备份

备份操作通常会将所有数据读入内存，因此，备份操作通常应该在副本集从节点而不是主节点进行，如果是单机MongoDB，则应该在空间时间进行备份，比如深夜凌晨。

#### 3、持久性

持久性是数据库必备的一种特性，想象一下，如果数据库不具备持久性，如果数据库重启，数据全部丢失，太可怕了，不敢想。

为了在服务器发生故障时提供持久性，MongoDB使用预写式日志机制，英文简称 WAL。WAL是数据库系统中一种常见的持久性技术。在数据存入数据库之前，将这些更改操作写到磁盘上。

从MongoDB4.0开始，执行写操作时，MongoDB会使用与oplog相同的格式创建日志。oplog语句具有幂等性，不管执行多少次，结果都是一样的。

MongoDB还维护了日志和数据库数据文件的内存视图。默认情况，每50毫秒会将日志条目刷新到磁盘上，每60秒会将数据库文件刷新到磁盘上。刷新数据的时间60秒间隔被称为检查点。日志用于将上一个检查点之后的数据提供持久性。MongoDB的持久性就是在发生故障时，重启之后，将日志中的语句重新执行一遍，以保证在关闭前丢失的数据重新刷新到MongoDB中。

MongoDB会在data目录下创建一个journal的子目录，WiredTiger日志文件的名称为WiredTigerLog.<sequence>。sequence是一个从0 000 000 001开始的数字。

![](https://s8.51cto.com/oss/202302/16/755a9302676c8015bbe7312d28a2c7f5cfadac.png)

MongoDB会对写入的日志进行压缩，日志文件限制的最大大小为100MB。如果大于100MB，MongoDB就会自动创建一个新的日志文件，由于日志文件只需在上次检查点之后恢复数据，因此在新的检查点写入完成时，旧的日志文件就会被删除。

