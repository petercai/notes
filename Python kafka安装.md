# Python kafka安装
什么是kafka?
---------

Kafka是由Apache软件基金会开发的开源流式数据平台。它最初由LinkedIn开发，用于处理大规模的实时数据流。Kafka被设计为高可靠、高吞吐量的分布式消息系统，用于发布、订阅流式数据。它具有持久性、水平扩展性和容错能力，能够处理数以千计的消息并支持多个消费者。

Kafka的核心概念包括以下几个部分：

1.  **Producer（生产者）** ：负责将数据发布到Kafka的主题（topic）。
2.  **Broker（代理）** ：Kafka集群中的服务器节点，负责存储数据和处理数据流。
3.  **Topic（主题）** ：数据流的类别，数据被发布到主题并由消费者订阅。
4.  **Consumer（消费者）** ：从Kafka主题中读取消息的应用程序。
5.  **ZooKeeper**：Kafka使用ZooKeeper来管理和协调Kafka broker。

Kafka被广泛应用于日志聚合、数据管道、事件驱动架构等场景，许多大型互联网公司和企业都在生产环境中使用Kafka来处理实时数据流

kafka发送数据的多方式
-------------

Kafka生产者发送数据到Kafka集群有多种方式，其中常见的包括：

1.  **同步发送**：在同步发送中，生产者发送消息并等待直到消息被成功写入至少一个副本后才继续。这种方式可以确保消息不会丢失，但会增加延迟，因为生产者需要等待确认。
2.  **异步发送**：在异步发送中，生产者发送消息后继续执行而不等待确认。这种方式可以提高性能，但可能会导致消息丢失，因为生产者在确认消息是否成功写入之前不会知道。
3.  **批量发送**：生产者可以将多个消息打包成批次进行发送，以减少网络开销和提高吞吐量。批量发送可以通过配置生产者的参数来实现。
4.  **带回调函数的发送**：生产者可以通过指定回调函数来处理发送结果。当消息成功发送或发送失败时，回调函数会被调用，可以用于处理成功发送的消息或处理发送失败的消息。
5.  **分区器自定义**：生产者可以通过自定义分区器来控制消息发送到哪个分区。通过自定义分区器，可以实现特定的消息路由逻辑，以满足业务需求。

这些方式可以根据具体的业务需求和性能要求来选择和组合使用，以实现高效可靠的数据发送到Kafka集群。

安装MySql
-------

手动下载[MySql](https://link.juejin.cn/?target=https%3A%2F%2Fwww.mysql.com%2Fcn%2Fdownloads%2F "https://www.mysql.com/cn/downloads/")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4502a050c86545e7810281fd101dcf29~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2788&h=1904&s=603663&e=png&a=1&b=fdfdfd)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6ed2b7c004841d1bf16200c8d3450eb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2790&h=1520&s=326969&e=png&a=1&b=fbfafa)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69c78845ff52496c9d797e0405145a59~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2790&h=1878&s=404644&e=png&a=1&b=f8f7f7)

自动安装mysql

```null
brew install mysql

```

配置mysql环境变量（.bash_profile）

```ruby
export PATH=$PATH:/usr/locol/mysql/bin

export PATH=$PATH:/usr/local/mysql/support-files

```

查看mysql的版本，打开终端，输入

```css
mysql --version 

```

初始化 MySQL 数据库

```null
mysql_secure_installation

```

启动 MySQL 服务

```sql
brew services start mysql

```

登录到MySQL服务器

```css
mysql -u root -p

```

修改数据库用户密码

```css
mysqladmin -u 这里输入用户名 -p password 这里输入数据库密码

```

安装Navicat Premium
-----------------

Navicat Premium 是一款功能强大的数据库管理工具，它提供了一个集成化的环境，用于在多种数据库系统中进行数据库开发、管理和维护。Navicat Premium 支持主流的数据库系统，包括 MySQL、MariaDB、Oracle、SQL Server、PostgreSQL 等。

例如，创建一个名为“user\_database”的数据库，并创建了两张表，分别是“contact\_table”，“login_table”

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/491d40a52e684e0290989493979445a6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=3028&h=1080&s=473282&e=png&a=1&b=fbfbfb)

* * *

**Python建立MySql数据库连接**

在 Python 环境中安装名为 "mysql-connector-python" 的软件包。这个软件包是用于在 Python 中连接和操作 MySQL 数据库的官方驱动程序

```null
pip install mysql-connector-python

```

pymysql 是一个用于在 Python 中连接和操作 MySQL 数据库的库。它是一个纯 Python 实现的 MySQL 客户端，提供了简单而强大的接口，使得在 Python 中进行数据库操作变得更加方便

```null
pip install pymysql

```

还需要安装一下 cryptography包，因为 python连接数据库是的加密方式需要 cryptography包。如果你不安装cryptography，python连接数据库会报错

```null
pip install cryptography

```

执行python代码

```ini
import pymysql

try:
    
    connection = pymysql.connect(
        host='localhost',  
        port=3306,  
        user='root',  
        password='这里填入先前自己设定的密码',  
        database='user_database',  
        charset='utf8mb4'  
    )
    
    insert_query = """
    INSERT INTO contact_table
    (agent_name, mobile, sex, email, whats_app, hide_mobile, agent_num, company_name, company_num, company_address, contact_check, add_time)
    VALUES
    ('John Doe', '1234567890', 'Male', 'johndoe@example.com', '1234567890', 0, 123, 'ABC Company', 456, '123 Main St', 'check', NOW())
    """
    
    cursor = connection.cursor()
    cursor.execute(insert_query)
    
    connection.commit()
    print("数据插入成功")
except Exception as e:
    print("连接MySQL数据库失败：", e)

```

插入成功后，可以通过数据库查看到插入的数据

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17e9631bfd134c44adaf07f78ad4a686~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=3028&h=1010&s=442535&e=png&a=1&b=f7f7f7)

安装 zookeeper
------------

安装kafka需要安装[zookeeper](https://link.juejin.cn/?target=https%3A%2F%2Fzookeeper.apache.org%2Freleases.html "https://zookeeper.apache.org/releases.html")，因为Kafka依赖于Zookeeper来管理其分布式节点，并存储元数据以确保一致性

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/291e0e75700c4841a754cc7c16bb00c7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2270&h=1862&s=493248&e=png&a=1&b=fefdfd)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23e864b6b52a4ead948bf8a5d4d93dec~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2528&h=1702&s=459415&e=png&a=1&b=fdfdfd)

然后进入 Zookeeper 的配置目录，一般是 conf 文件夹，复制 zoo_sample.cfg 并重命名为 zoo.cfg

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de60ca2af5bf458fb42126b3c7bf309e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1886&h=908&s=153988&e=png&a=1&b=fcfcfc)

其中，zoo.cfg配置文件中各配置项的含义如下所示：

```ini

tickTime=2000


initLimit=10


syncLimit=5


dataDir=/tmp/zookeeper


clientPort=2181


maxClientCnxns=60


autopurge.snapRetainCount=3


autopurge.purgeInterval=1








extendedTypesEnabled=true


admin.serverPort=8888

```

配置好zoo.conf配置文件后，执行以下命令启动ZooKeeper

```bash
cd /Users/xxx/Desktop/apache-zookeeper-3.9.2-bin/bin

./zkServer.sh start

```

启动CLI

```bash
./zkCli.sh

```

停止Zookeeper服务器

```arduino
./zkServer.sh stop

```

安装 kafka
--------

*   首先，你需要从 Apache Kafka 的官方网站[下载 Kafka](https://link.juejin.cn/?target=https%3A%2F%2Fkafka.apache.org%2Fdownloads "https://kafka.apache.org/downloads") 的压缩包

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f145d05b0eb34ab19834e5641118b3c8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2544&h=1356&s=293546&e=png&a=1&b=fefefe)

*   解压下载的压缩包到本地路径，然后进入config目录下，编辑server.properties配置文件

| 属性 | 默认值 | 描述 |
| --- | --- | --- |
| broker.id | 0 | 每个broker都可以用一个唯一的非负整数id进行标识；这个id可以作为broker的名字，你可以选择任意数字作为id，但是一定要保证唯一性； |
| log.dirs | /tmp/kafka-logs | kafka存放数据的路径。这个路径并不是唯一的，可以是多个，路径之间使用逗号分隔；每当创建新的partition时，都会选择在包含最少partitions的路径下进行； |
| listeners | PLAINTEXT://192.168.65.60:9092 | server接受客户端连接的端口，ip配置kafka本机ip即可 |
| zookeeper.connect | localhost:2181 | zookeeper连接字符串的格式为：hostname:port，此处分别对应zk集群中的节点；连接方式为:hostname1:port1,hostname2:port2,hostname3:port3 |
| log.retention.hours | 168 | 每个日志文件删除之前保存的时间。默认数据保存时间对所有topic都一样 |
| num.partitions | 1 | 创建topic的默认分区数 |
| default.replication.factor | 1 | 自动创建topic的默认副本数量，建立设置为大于等于2 |
| min.insync.replicas | 1 | 当producer设置acks=-1时，min.insync.replicas指定的最小数目（必须确认每一个repica的写数据都是成功的），如果这个数目没有达到，producer发送消息会产生异常 |
| delete.topic.enable | false | 是否允许删除主题 |

在以上的配置项中，我们最主要需要关注如下几个配置内容

【broker的序号】broker.id=0  
【当前kafka的监听地址】listeners=PLAINTEXT://localhost:9092  
【日志的存储路径】log.dirs=/Users/muse/kafka_2.13-3.0.0/kafka-logs  
【zookeeper的服务地址】zookeeper.connect=localhost:2181

*   找到项目根目录，运行以下命令，启动 Kafka Broker

```bash
cd /Users/xxx/Desktop/kafka-3.7.0-src/bin
  
./kafka-server-start.sh ../config/server.properties

```

停止kafak服务器

```vbscript
./kafka-server-stop.sh  ../config/server.properties

```

安装EFAK（原名 Kafka Eagle）
----------------------

[EAFK](https://link.juejin.cn/?target=https%3A%2F%2Fwww.kafka-eagle.org%2F "https://www.kafka-eagle.org/") 是一个开源的 Kafka 集群管理和监控工具，旨在帮助用户更好地管理和监控其 Kafka 集群。

*   **实时监控：**  实时监控 Kafka 集群的状态、健康状况以及性能指标。
*   **消费者组管理：**  查看和管理消费者组、消费者、消费者偏移等信息。
*   **Topic 管理：**  创建、修改、删除 Kafka Topic，并查看 Topic 详细信息。
*   **告警系统：**  支持配置告警规则，及时发现集群问题并通知管理员。
*   **图表和报表：**  提供可视化的图表和报表，帮助用户更好地理解集群情况。
*   **用户权限：**  支持多用户和角色权限管理，确保安全性。
*   **易于部署：**  提供简单的安装和部署流程，适用于各种规模的 Kafka 集群。
*   **跨平台：**  支持 Linux、Windows、Mac OS X 等多种平台。
*   **跨版本：**  支持 Kafka KRaft 模式。

创建mysql数据库ke 用来储存元数据

修改EFAK的conf目录下配置文件——system-config.properties

```ini
efak.username=root
efak.password=填写mysql的密码

```

配置环境变量

```bash
export KE_HOME=/Users/xxx/Desktop/kafka-eagle-bin-3.0.1/efak-web-3.0.1
export PATH=$PATH:$KE_HOME/bin

```

单机版启动EFAK

```bash
cd /Users/xxx/Desktop/kafka-eagle-bin-3.0.1/efak-web-3.0.1/bin

./ke.sh start

```

集群方式启动

```bash
./ke.sh cluster start
./ke.sh cluster restart

```

启动成功界面，如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a458fcf78ac84f589039a57602a6f713~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1042&h=608&s=102281&e=png&a=1&b=fdfdfd)

在浏览器中输入 [http://127.0.0.1:8048](https://link.juejin.cn/?target=http%3A%2F%2F127.0.0.1%3A8048 "http://127.0.0.1:8048")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07a72e1cb9954b9d8458dc8581f83139~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2012&h=1358&s=146318&e=png&a=1&b=1a1a1a)

然后输入账号密码：admin / 123456，进行登陆

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21de74f0cdec46d68c1e6c14ba78d05d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=3850&h=1816&s=388699&e=png&a=1&b=181e2e)

EFAK常用命令

| 命令 | 描述 |
| --- | --- |
| ke.sh start | 启动EFAK服务器 |
| ke.sh status | 查看EFAK运行状态 |
| ke.sh stop | 停止EFAK服务器 |
| ke.sh restart | 重新启动EFAK服务器 |
| ke.sh stats | 查看linux操作系统中的EFAK句柄数 |
| ke.sh cluster start | 查看EFAK集群分布式启动 |

用python写一个kafka的示例
------------------

```python
from kafka import KafkaProducer, KafkaConsumer
from kafka.errors import KafkaError


topic = 'test_topic'

producer = KafkaProducer(bootstrap_servers=['xxx.xxx.x.xx:xxxx'])
for i in range(20):
    
    message = f'消息内容 i = {i}'
    future = producer.send(topic, bytes(message, 'utf-8'))
    try:
        
        record_metadata = future.get(timeout=10)
        print(f'{message} 发送到分区 {record_metadata.partition} 偏移量 {record_metadata.offset}')
    except KafkaError as e:
        print('发送消息失败， {}: {}'.format(message, e))

consumer = KafkaConsumer(topic, bootstrap_servers=['xxx.xxx.x.xx:xxxx'], auto_offset_reset='earliest',
                         enable_auto_commit=True, group_id='my-group', max_poll_records=10)

while True:
    
    messages = consumer.poll(timeout_ms=1000)
    if not messages:
        continue
    for topic_partition, records in messages.items():
        for record in records:
            print(f"从Kafka集群中拉取消息 = {record.value.decode('utf-8')}")



```

