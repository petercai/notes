# 用好kafka，你不得不知的那些工具
前言
--

工欲善其事，必先利其器。本文主要分享一下消息中间件 kafka 安装部署的过程，以及我平时在工作中针对 kafka 用的一些客户端工具和监控工具。

kafka 部署架构
----------

![](_assets/870e1606bd4bf3dc9e2b2d4568bd28f3.png)

一个 kafka 集群由多个`kafka broker`组成，每个`broker`将自己的元数据信息注册到`zookeeper`中，通过`zookeeper`关联形成一个集群。

prettyZoo 客户端
-------------

既然`kafka`依赖`zookeeper`，我难免就需要看看`zookeeper`中究竟存储了`kafka`的哪些数据，这边介绍一款高颜值的客户端工具`prettyZoo`。`PrettyZoo`是一款基于`Apache Curator` 和 `JavaFX` 实现的 `Zookeeper` 图形化管理客户端，使用非常简单。

**下载地址：**  `https://github.com/vran-dev/PrettyZoo`  

*   连接
    

![](_assets/ffa758ee1f81296920cf1e277cfc716d.png)

*   界面化操作`zookeeper`  
    

![](_assets/6f5028b7a084e49952c5de43c1c9be48.png)

**小 tips:** `kafka`部署时配置文件中配置`zookeeper`地址的时候，可以采用如下的方式，带上目录，比如`xxxx:2181/kafka`或者`xxxx:2181/kafka1`,可以避免冲突。

```
#配置连接 Zookeeper 集群地址（在 zk 根目录下创建/kafka，方便管理）
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/ka
fka
```

kafka Tool 客户端
--------------

`Kafka Tool`是一个用于管理和使用`Apache Kafka`集群的`GUI`应用程序。 `Kafka Tool`提供了一个较为直观的 UI 可让用户快速查看`Kafka`集群中的对象以及存储在`topic`中的消息，提供了一些专门面向开发人员和管理员的功能。

**下载地址：**  `https://www.kafkatool.com/index.html`  

![](_assets/1e76fdcfdf99415ed9f0c8fcdf0fc1ff.png)

kafka 监控工具
----------

`kafka` 自身并没有继承监控管理系统，因此对 `kafka` 的监控管理比较不便，好在有大量的第三方监控管理系统来使用，这里介绍一款优秀的监控工具`Kafka Eagle`，可以用监控 `Kafka` 集群的整体运行情况。

**下载地址**：`https://www.kafka-eagle.org/`，部署也很简单，根据官方文档一步一步来即可。

注意，kafka 需要开启 JMX 端口，即修改`kafka`的启动命令文件`kafka-server-start.sh`，如下图：

![](_assets/9ecf20224ec44b1c78fecd24f7c79c41.png)

![](_assets/e80c18f66cd8999631467e113514212c.png)

![](_assets/14d5d312832fb4b4ba45d71c9d8910a2.png)

kafka 集群部署
----------

### 一、zookeeper 集群部署

1.  上传安装包
    
2.  移动到指定文件夹
    

```
mv zookeeper-3.4.6.tar.gz /opt/apps/
```

3.  解压
    

```
tar -zxvf zookeeper-3.4.6.tar.gz
```

4.  修改配置文件
    

*   进入配置文件目录
    

```
cd /opt/apps/zookeeper-3.4.6/conf
```

*   修改配置文件名称
    

```
mv zoo_sample.cfg zoo.cfg
```

*   编辑配置文件 `vi zoo.cfg`  
    

```
## zk数据保存位置
dataDir=/opt/apps/data/zkdata
## 集群配置, hadoop1、hadoop2、hadoop3是主机名，后面是端口，没有被占用即可
server.1=hadoop1:2888:3888 
server.2=hadoop2:2888:3888 
server.3=hadoop3:2888:3888
```

5.  创建数据目录
    

```
mkdir -p /opt/apps/data/zkdata
```

6.  生成一个 `myid` 文件，内容为它的`id`, 表示是哪个节点。
    

```
echo 1 > /opt/apps/data/zkdata/myid
```

7.  配置环境变量
    

```
vi /etc/profile 

#ZOOKEEPER_HOME 
export ZOOKEEPER_HOME=/opt/apps/zookeeper-3.4.6 
export PATH=$PATH:$ZOOKEEPER_HOME/bin 

source /etc/profile
```

8.  在其他几个节点，即`hadoop2`, `hadoop3`上重复上面的步骤，但是 myid 文件的内容有所区别，分别是对应的 id。
    

```
echo 2 > /opt/apps/data/zkdata/myid
echo 3 > /opt/apps/data/zkdata/myid
```

9.  启停集群
    

```
bin/zkServer.sh start zk 服务启动 
bin/zkServer.sh status zk 查看服务状态
bin/zkServer.sh stop zk 停止服务
```

### 二、kafka 集群部署

1.  官方下载地址：`http://kafka.apache.org/downloads.html`  
    
2.  上传安装包, 移动到指定文件夹
    

```
mv kafka_2.11-2.2.2.tgz /opt/apps/
```

3.  解压
    

```
tar -zxvf kafka_2.11-2.2.2.tgz
```

4.  修改配置文件
    

*   进入配置文件目录
    

```
cd /opt/apps/kafka_2.11-2.2.2/config
```

*   编辑配置文件`vi server.properties`  
    

```
#为依次增长的：0、1、2、3、4，集群中唯一 id 
broker.id=0 
#数据存储的⽬录 
log.dirs=/opt/apps/data/kafkadata 
#指定 zk 集群地址,注意这里加了一个目录 
zookeeper.connect=hadoop1:2181,hadoop2:2181,hadoop3:2181/kafka
```

其他的配置内容说明如下：

```
#broker 的全局唯一编号，不能重复，只能是数字。
broker.id=0
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘 IO 的线程数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka 运行日志(数据)存放的路径，路径不需要提前创建，kafka 自动帮你创建，可以
配置多个磁盘路径，路径与路径之间可以用"，"分隔
log.dirs=/opt/module/kafka/datas
#topic 在当前 broker 上的分区个数
num.partitions=1
#用来恢复和清理 data 下数据的线程数量
num.recovery.threads.per.data.dir=1
# 每个 topic 创建时的副本数，默认时 1 个副本
offsets.topic.replication.factor=1
#segment 文件保留的最长时间，超时将被删除
log.retention.hours=168
#每个 segment 文件的大小，默认最大 1G
log.segment.bytes=1073741824
# 检查过期数据的时间，默认 5 分钟检查一次是否数据过期
log.retention.check.interval.ms=300000
#配置连接 Zookeeper 集群地址（在 zk 根目录下创建/kafka，方便管理）
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/ka
fka
```

5.  配置环境变量
    

```
vi /etc/profile 

export KAFKA_HOME=/opt/apps/kafka_2.11-2.2.2 
export PATH=$PATH:$KAFKA_HOME/bin 

source /etc/profile
```

6.  在不同的节点上重复上面的步骤，但是需要修改配置文件`server.properties`中的`broker.id`。
    

```
# broker.id标记是哪个kafka节点，不能重复
broker.id=1 
broker.id=2
```

7.  启停集群
    

```
# 启动集群
bin/kafka-server-start.sh -daemon /opt/apps/kafka_2.11-2.2.2/config/server.properties 
# 停止集群 
bin/kafka-server-stop.sh stop
```

kafka 命令行工具
-----------

### 1\. 主题命令行操作

*   查看操作主题命令参数`kafka-topics.sh`  
    

![](_assets/abfb015da24af172b301aca5949452cc.png)

*   查看当前服务器中的所有 `topic`  
    

```
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --list
```

*   创建 `first topic`  
    

```
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --create --partitions 1 --replication-factor 3 --topic first
```

选项说明：

--topic 定义 topic 名

--replication-factor 定义副本数

--partitions 定义分区数

*   查看 `first` 主题的详情
    

```
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --describe --topic first
```

*   修改分区数（注意：**分区数只能增加，不能减少**）
    

```
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --alter --topic first --partitions 3
```

*   删除 topic
    

```
bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --delete --topic first
```

### 2\. 生产者命令行操作

*   查看操作生产者命令参数`kafka-console-producer.sh`  
    

![](_assets/e7ed394fb28c7559805952f4f4826d46.png)

*   发送消息
    

```
bin/kafka-console-producer.sh --bootstrap-server hadoop102:9092 --topic first
>hello world
>xuyang hello
```

### 3\. 消费者命令行操作

*   查看操作消费者命令参数`kafka-console-consumer.sh`  
    

![](_assets/6dc4d91a61502e2c336e778253d75898.png)

![](_assets/550a264dfcb6c9a18214cca80bc7e673.png)

*   消费消息
    

```
bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic first
```

*   把主题中所有的数据都读取出来（包括历史数据）。
    

```
bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --from-beginning --topic first
```

总结
--

本文分享了平时我在工作使用 kafka 以及 zookeeper 常用的一些工具，同时分享了 kafka 集群的部署，值得一提的是 kafka 部署配置 zookeeper 地址的时候，我们可以添加一个路径，比如`hadoop:2181/kafka`这种方式，那么 kafka 的元数据信息都会放到`/kafka`这个目录下，以防混淆。

