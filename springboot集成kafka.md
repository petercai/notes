# springboot集成kafka
安装kafka
-------

启动Kafka本地环境需Java 8+以上

Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者在网站中的所有动作流数据。

Kafka启动方式有Zookeeper和Kraft，两种方式只能选择其中一种启动，不能同时使用。

Kafka下载[downloads.apache.org/kafka/3.7.0…](https://link.juejin.cn/?target=https%3A%2F%2Fdownloads.apache.org%2Fkafka%2F3.7.0%2Fkafka%255C_2.13-3.7.0.tgz "https://downloads.apache.org/kafka/3.7.0/kafka%5C_2.13-3.7.0.tgz")

![](https://bbs-img.huaweicloud.com/blogs/img/image1_521.png)

解压tar -xzf kafka_2.13-3.7.0.tgz

![](https://bbs-img.huaweicloud.com/blogs/img/image2_528.png)

**一、Zookeeper启动Kafka（kafka内置zookeeper）**

Kafka依赖Zookeeper

1、启动Zookeeper 2、启动Kafka

使用kafka自带Zookeeper启动

./zookeeper-server-start.sh ../config/zookeeper.properties &

./zookeeper-server-stop.sh ../config/zookeeper.properties

![](https://bbs-img.huaweicloud.com/blogs/img/image3_515.png)

./kafka-server-start.sh ../config/server.properties &

![](https://bbs-img.huaweicloud.com/blogs/img/image4_473.png)

./kafka-server-stop.sh ../config/server.properties

**二、Zookeeper服务器启动Kafka**

Zookeeper服务器安装

[zookeeper.apache.org/](https://link.juejin.cn/?target=https%3A%2F%2Fzookeeper.apache.org%2F "https://zookeeper.apache.org/")

![](https://bbs-img.huaweicloud.com/blogs/img/image5_446.png)

[dlcdn.apache.org/zookeeper/z…](https://link.juejin.cn/?target=https%3A%2F%2Fdlcdn.apache.org%2Fzookeeper%2Fzookeeper-3.9.2%2Fapache-zookeeper-3.9.2-bin.tar.gz "https://dlcdn.apache.org/zookeeper/zookeeper-3.9.2/apache-zookeeper-3.9.2-bin.tar.gz")

tar zxvf apache-zookeeper-3.9.2-bin.tar.gz

![](https://bbs-img.huaweicloud.com/blogs/img/image6_403.png)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75e2de4c9dfc4bf0b10805d665d3cb49~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=681&h=76&s=8666&e=png&b=000000)

配置Zookeeper服务器

cp zoo_sample.cfg zoo.cfg

![](https://bbs-img.huaweicloud.com/blogs/img/image8_347.png)

启动Zookeeper服务器

![](https://bbs-img.huaweicloud.com/blogs/img/image9_286.png)

./zkServer.sh start

修改Zookeeper端口

![](https://bbs-img.huaweicloud.com/blogs/img/image10_263.png)

Zoo.cfg添加内容

admin.serverPort=8099

apache-zookeeper-3.9.2-bin/bin目录下重启Zookeeper

![](https://bbs-img.huaweicloud.com/blogs/img/image11_233.png)

Zookeeper服务器启动kafka

/opt/kafka_2.13-3.7.0/bin目录下

./kafka-server-start.sh ../config/server.properties &

![](https://bbs-img.huaweicloud.com/blogs/img/image12_221.png)

Kafka配置文件server.properties

![](https://bbs-img.huaweicloud.com/blogs/img/image13_196.png)

**三、使用KRaft启动Kafka**

![](https://bbs-img.huaweicloud.com/blogs/img/image14_189.png)

UUID通用唯一识别码（Universally Unique Identifier）

1、生成Cluster UUID(集群UUID)：./kafka-storage.sh random-uuid

![](https://bbs-img.huaweicloud.com/blogs/img/image15_176.png)

2.格式化kafka日志目录：./kafka-storage.sh format -t 3pMJGNJcT0uLIBsZhbucjQ -c ../config/kraft/server.properties

![](https://bbs-img.huaweicloud.com/blogs/img/image16_164.png)

3.启动kafka：./kafka-server-start.sh ../config/kraft/server.properties &

![](https://bbs-img.huaweicloud.com/blogs/img/image17_148.png)

springboot集成kafka
-----------------

![](https://bbs-img.huaweicloud.com/blogs/img/image1_581.png)

创建topic时，若不指定topic的分区(partition)数量使，则默认为1个分区(partition)

![](https://bbs-img.huaweicloud.com/blogs/img/image2_582.png)

### 修改server.properties文件

vim server.properties

listeners=PLAINTEXT://0.0.0.0:9092

![](https://bbs-img.huaweicloud.com/blogs/img/image3_578.png)

advertised.listeners=PLAINTEXT://192.168.68.133:9092

![](https://bbs-img.huaweicloud.com/blogs/img/image4_531.png)

![](https://bbs-img.huaweicloud.com/blogs/img/image5_497.png)

![](https://bbs-img.huaweicloud.com/blogs/img/image6_444.png)

### springboot加入依赖kafka

```xml
<dependency><groupId>org.springframework.kafka</groupId><artifactId>spring-kafka</artifactId></dependency>

```

加入spring-kafka依赖后，springboot自动装配好kafkaTemplate的Bean

![](https://bbs-img.huaweicloud.com/blogs/img/image7_412.png)

application.yml配置连接kafka

```makefile
spring:kafka:bootstrap-servers: 192.168.68.133:9092

```

### 生产者

发送消息

![](https://bbs-img.huaweicloud.com/blogs/img/image8_380.png)

```typescript
@Resourceprivate KafkaTemplate<String,String> kafkaTemplate;@Testvoid kafkaSendTest(){kafkaTemplate.send("kafkamsg01","hello kafka");}

```

![](https://bbs-img.huaweicloud.com/blogs/img/image9_315.png)

### 消费者

接收消息

![](https://bbs-img.huaweicloud.com/blogs/img/image10_290.png)

```typescript
@Componentpublic class KafkaConsumer {@KafkaListener(topics = {"kafkamsg01","test"},groupId = "123")public void consume(String message){System.out.println("接收到消息："+message);}}

```

若没有配置groupid

Failed to start bean 'org.springframework.kafka.config.internalKafkaListenerEndpointRegistry'; nested exception is java.lang.IllegalStateException: No group.id found in consumer config, container properties, or @KafkaListener annotation; a group.id is required when group management is used.

![](https://bbs-img.huaweicloud.com/blogs/img/image11_258.png)

![](https://bbs-img.huaweicloud.com/blogs/img/image12_245.png)

```typescript
@Componentpublic class KafkaConsumer {@KafkaListener(topics = {"kafkamsg01","test"},groupId = "123")public void consume(String message){System.out.println("接收到消息："+message);}}

```

![](https://bbs-img.huaweicloud.com/blogs/img/image13_216.png)
utm_campaign=other&utm_content=content")