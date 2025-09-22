# Configure Kafka to delivery exact-one message
Configuring Apache Kafka to support exact-one message delivery can be achieved by using a combination of Kafka's built-in features and some additional strategies. Exact-one message delivery typically means that a message should be delivered to exactly one consumer, and it should not be duplicated or lost. Here's a step-by-step guide on how to configure Kafka for this purpose:

1.  **Topic Configuration**:
    
    *   Create a Kafka topic where you want to achieve exact-one message delivery. You can do this using the `kafka-topics.sh` script or Kafka's Java API.
    
    bash
    
    `bin/kafka-topics.sh --create --topic your_topic_name --partitions 1 --replication-factor 3 --zookeeper your_zookeeper_server:2181` 
    
    In this example, we create a topic with a single partition and a replication factor of 3 for fault tolerance. Adjust the configuration according to your needs.
    
2.  **Producer Configuration**:
    
    *   Configure your Kafka producer to produce messages with a unique key for each message. This key will be used to determine the partition to which the message is sent.
    
    java
    
    `Properties properties = new Properties();
    properties.put("bootstrap.servers", "your_broker_servers");
    properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    
    Producer<String, String> producer = new KafkaProducer<>(properties);` 
    
    When sending a message, specify a key:
    
    java
    
    `producer.send(new ProducerRecord<>("your_topic_name", "message_key", "message_value"));` 
    
    Using a unique key for each message ensures that messages with the same key are always sent to the same partition.
    
3.  **Consumer Configuration**:
    
    *   Configure your Kafka consumers with a unique `group.id` for each consumer group. This ensures that each message is consumed by only one consumer in the group.
    
    java
    
    `Properties properties = new Properties();
    properties.put("bootstrap.servers", "your_broker_servers");
    properties.put("group.id", "your_unique_consumer_group_id");
    properties.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    properties.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    
    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);` 
    
4.  **Consumer Processing Logic**:
    
    *   Implement your consumer logic to process each message once and acknowledge its successful processing.
    
    java
    
    `while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord<String, String> record : records) {
            // Process the message
            System.out.printf("Received message: key=%s, value=%s%n", record.key(), record.value());
            
            // Commit the offset to mark the message as processed
            consumer.commitSync(Collections.singletonMap(record.partition(), new OffsetAndMetadata(record.offset() + 1)));
        }
    }` 
    
    The `commitSync` method commits the offset to Kafka after processing, ensuring that the message won't be consumed again.
    
5.  **Error Handling and Retries**:
    
    *   Implement error handling and retries in your consumer to handle processing failures gracefully. Messages that cannot be successfully processed should not be lost, and you should have mechanisms to handle exceptions and retries.

By following these steps and configuring your Kafka producers and consumers as described, you can achieve exact-one message delivery, where each message is delivered to exactly one consumer and is not duplicated or lost. Proper error handling and retries are essential to ensure the reliability of your Kafka-based system.