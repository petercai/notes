# 消息的顺序性:Message Key 和 Message Queue Selector

`一般为了高性能，会配置多个Partition，比如8个Partition，16个Partition，为了保证有序消息，应该如何设计消息的投递`

### 高可用、高性能下的有序消息设计 🚀

在实现高可用和高性能的消息系统时，通常会配置多个 Partition。要在这种情况下保证消息的顺序性，我们需要合理设计消息的投递策略。下面分别介绍 RocketMQ 和 Kafka 的实现原理和具体设计方法。

`在投递的环节，通过使用消息键（Message Key）来对消息进行分类，确保同一个业务ID的消息进入同一个队列或分区，从而保证消息的顺序性。`

### 原理详解 🚀

#### 消息投递和分类

在 Kafka 和 RocketMQ 中，消息投递和分类的过程如下：

1.  **消息键（Message Key）**：
    
    *   生产者在发送消息时，会为每条消息指定一个消息键（如订单ID、用户ID等）。
    *   消息键用于标识消息的业务ID，确保相同业务ID的消息被分类到同一个队列或分区。
2.  **分区选择**：
    
    *   根据消息键进行分区选择。Kafka 和 RocketMQ 都会对消息键进行哈希计算，并根据分区数量取模，确定消息应该发送到哪个分区。
    *   这样可以确保具有相同消息键的消息总是被发送到同一个分区，从而保证这些消息的顺序性。

#### RocketMQ 实现原理和设计方法 🌟

##### 1\. 消息投递到特定的 Queue（Partition）

在 RocketMQ 中，每个 Topic 可以有多个 Queue（Partition）。为了保证消息的顺序性，我们可以使用 **Message Queue Selector** 来将消息投递到特定的 Queue。

**生产者端实现：** 

生产者可以使用 `MessageQueueSelector` 接口，根据消息的业务ID（如订单ID）选择 Queue（Partition）。这样，`具有相同业务ID的消息将始终发送到同一个 Queue，从而保证顺序性`。

示例代码：

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.List;

public class OrderProducer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("OrderProducerGroup");
        producer.start();

        String topic = "OrderTopic";
        String[] tags = new String[] {"TagA", "TagB", "TagC"};
        for (int i = 0; i < 100; i++) {
            int orderId = i % 10; 
            Message msg = new Message(topic, tags[i % tags.length], "KEY" + i,
                ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
            SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                @Override
                public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                    Integer id = (Integer) arg;
                    int index = id % mqs.size(); 
                    return mqs.get(index);
                }
            }, orderId); 
            System.out.printf("%s%n", sendResult);
        }
        producer.shutdown();
    }
}

```

**消费者端实现：** 

在消费者端，使用顺序消息监听器 `MessageListenerOrderly` 来消费消息，确保同一时间只有一个线程在处理同一个 Queue 中的消息。

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerOrderly;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class OrderConsumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("OrderConsumerGroup");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.subscribe("OrderTopic", "TagA || TagB || TagC");

        consumer.registerMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
                for (MessageExt msg : msgs) {
                    
                    System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), new String(msg.getBody()));
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });

        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}

```

#### Kafka 实现原理和设计方法 🌟

##### 1\. 消息投递到特定的 Partition

在 Kafka 中，每个 Topic 也可以有多个 Partition。为了保证消息的顺序性，我们可以使用消息键（Message Key）来决定消息投递到哪个 Partition。这样，`具有相同消息键的消息将始终发送到同一个 Partition，从而保证顺序性`。

**生产者端实现：** 

生产者在发送消息时，可以指定消息键（如订单ID）。Kafka 使用消息键进行分区选择，确保具有相同消息键的消息总是被发送到同一个 Partition。

示例代码：

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.Future;

public class OrderProducer {
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        String topic = "OrderTopic";
        for (int i = 0; i < 100; i++) {
            String orderId = "order" + (i % 10); 
            String value = "Order data " + i;
            ProducerRecord<String, String> record = new ProducerRecord<>(topic, orderId, value);
            Future<RecordMetadata> future = producer.send(record);
            RecordMetadata metadata = future.get();
            System.out.printf("Sent message with key %s to partition %d with offset %d%n", orderId, metadata.partition(), metadata.offset());
        }

        producer.close();
    }
}

```

**消费者端实现：** 

消费者端按 Partition 顺序消费消息，确保同一时间只有一个线程在处理同一个 Partition 中的消息。

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class OrderConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "OrderConsumerGroup");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("OrderTopic"));

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("Consumed message with key %s from partition %d with offset %d: %s%n", record.key(), record.partition(), record.offset(), record.value());
            }
        }
    }
}

```

### 总结 🌟

**RocketMQ** 和 **Kafka** 都通过消息键（业务ID）来保证消息的顺序性：

*   **RocketMQ** 使用 `MessageQueueSelector` 接口将消息发送到特定的 Queue（Partition）。
*   **Kafka** 使用消息键来决定消息发送到特定的 Partition。

这两种方法都能在高可用、高性能的环境中保证消息的顺序性，同时充分利用分区（Partition）带来的并行处理能力。希望这些内容对你理解 RocketMQ 和 Kafka 的顺序消费特性有所帮助！如果有任何问题，欢迎随时交流！😊

扩容
--

`mqs.size()` 是动态获取的。在 RocketMQ 和 Kafka 中，`mqs.size()` 动态获取队列（Queue）或分区（Partition）的数量，这确保了当你增加或减少队列或分区数量时，代码可以自动适应新的配置，而无需修改。

### 动态获取队列/分区数量的实现 🚀

#### RocketMQ 中的实现

在 RocketMQ 中，`mqs.size()` 动态获取的是当前 Topic 下所有队列的数量。示例代码如下：

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.common.RemotingHelper;

import java.util.List;

public class OrderProducer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("OrderProducerGroup");
        producer.start();

        String topic = "OrderTopic";
        String[] tags = new String[] {"TagA", "TagB", "TagC"};
        for (int i = 0; i < 100; i++) {
            int orderId = i % 10; 
            Message msg = new Message(topic, tags[i % tags.length], "KEY" + i,
                ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
            SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                @Override
                public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                    Integer id = (Integer) arg;
                    int index = id.hashCode() % mqs.size(); 
                    return mqs.get(index);
                }
            }, orderId); 
            System.out.printf("%s%n", sendResult);
        }
        producer.shutdown();
    }
}

```

在这个示例中，`mqs` 是 `MessageQueue` 的列表，`mqs.size()` 返回的是当前 Topic 下的队列数量。

#### Kafka 中的实现

在 Kafka 中，分区数量也是动态获取的。示例代码如下：

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.Future;

public class OrderProducer {
    public static void main(String[] args) throws Exception {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        String topic = "OrderTopic";
        for (int i = 0; i < 100; i++) {
            String orderId = "order" + (i % 10); 
            String value = "Order data " + i;
            ProducerRecord<String, String> record = new ProducerRecord<>(topic, orderId, value);
            Future<RecordMetadata> future = producer.send(record);
            RecordMetadata metadata = future.get();
            System.out.printf("Sent message with key %s to partition %d with offset %d%n", orderId, metadata.partition(), metadata.offset());
        }

        producer.close();
    }
}

```

在 Kafka 中，分区的数量是由 Kafka 集群动态管理的，生产者在发送消息时会根据消息键来确定消息的分区。

### 动态获取队列/分区数量的优点 🌟

*   **扩展性**：无需修改代码即可扩容或缩减队列/分区数量。
*   **灵活性**：自动适应新的队列/分区数量，简化运维。
*   **可靠性**：确保消息按照业务ID的哈希值均匀分布到不同的队列/分区中，从而保证高性能和有序性。

