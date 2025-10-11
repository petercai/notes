# springboot kafka消费者配置详解
背景：众所周知kafka是一个很强大的消息中间件，能支持大数量的消息订阅转发、性能极强。对于kafka消费者来说，经常会遇到消息积压、消费慢的情况。这其中很可能是因为消费者的配置没有配置合适导致的。

首先说一下使用的技术框架 spring-boot 2.6.4 + kafka

消费端简易代码

```java
@Component
@Slf4j
public class MessageReceiveListener {

    @KafkaListener(topics = "ifaas-test", containerFactory = "ifaasContainerFactory")
    public void receiveMessage1(List<ConsumerRecord> consumerRecords, Acknowledgment ack) {

        try {
            log.info("receiveMessage1 接收的kafka消息："+consumerRecords.size());
            ack.acknowledge();
        } catch (Exception e) {
            log.error("kafka失败消息:{}", JSON.toJSONString(consumerRecords));
        }
    }
}

```

这里kafka消费者采用的是@KafkaListener的形式。

其中构建containerFactory的代码如下

```java
@Bean("ifaasContainerFactory")
public ConcurrentKafkaListenerContainerFactory ifaasContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
    Map<String, Object> props = staticConsumerProps(ifaasServers, groupId);
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
    factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(props));
    factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);
    
    factory.setBatchListener(true);
    
    factory.setConcurrency(1);
    return factory;
}

private Map<String, Object> staticConsumerProps(String servers, String groupId) {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
    props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
    
    props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 300000);
    props.put(ConsumerConfig.REQUEST_TIMEOUT_MS_CONFIG, 350000);
    props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 3000);
    props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 400000);
    props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

    
    
    props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, pollRecords);
    
    props.put(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, 1024 * 100); 
    
    
    props.put(ConsumerConfig.FETCH_MAX_WAIT_MS_CONFIG, 30000);
}

```

配置详解
----

比较关键的就是staticConsumerProps下的配置，如果配置不当，还会导致启动服务报错，下面将会对每一个配置进行详细解读。 总的来说，涉及到的几个时间相关的参数很重要，参数之间的有相互制约的关系：

*   request.timeout.ms：请求超期时间。应该要大于或者等于session.timeout.ms
*   session.timeout.ms：会话超期是啊金。应该要小于或者等于max.poll.interval.ms
*   heartbeat.interval.ms：发送心跳的时间。应该要小于或者等于session.timeout.ms的1/3
*   max.poll.interval.ms：最大轮训间隔时间，确保消费者有足够时间处理消息。

影响每次拉取消息的数量和间隔多久拉取数据的配置 `fetch.min.bytes`（一次fetch操作最小的字节数） 和 `fetch.max.wait.ms`（一次fetch操作的最大等待时间） 哪个先满足就会立即返回给消费者。这两者可以根据消费者的吞吐量和数据流量合理计算出最高性能的值。

### 1、BOOTSTRAP\_SERVERS\_CONFIG

kafka服务器的地址，一般是部署kafka服务器的ip+端口，如：192.168.1.1:9092

### 2、ENABLE\_AUTO\_COMMIT_CONFIG

*   **目的**：自动提交偏移量可以简化消费者的实现，使得消费者不需要手动管理偏移量的提交。
    
*   **默认值**：默认值是 `true`，意味着自动提交偏移量是启用的。
    
*   **使用场景**：
    
    *   **启用自动提交**（`true`）：适用于简单的消费场景，消费者不需要对消息处理的精确控制，也不需要保证消息处理的完全准确性和可靠性。
    *   **禁用自动提交**（`false`）：适用于需要精确控制和高可靠性要求的消费场景，消费者需要在确保消息处理成功后才手动提交偏移量
*   **与 auto.commit.interval.ms 的关系**： `auto.commit.interval.ms`：控制自动提交偏移量的时间间隔，默认值是 5000 毫秒（5 秒）。只有在 `enable.auto.commit` 设置为 `true` 时，这个参数才有效。
    
*   **优缺点**
    

1.  **自动提交的优点**：
    
    *   简化代码实现，不需要手动管理偏移量。
    *   适用于对消息处理准确性要求不高的应用场景。
2.  **自动提交的缺点**：
    
    *   可能导致消息重复消费或消息丢失的情况。例如，如果在自动提交之前消费者崩溃，可能会导致已经处理的消息被再次消费。
    *   不能确保消息处理的事务性和一致性。
3.  **手动提交的优点**：
    
    *   更精确地控制偏移量的提交，确保只有在消息成功处理后才提交偏移量。
    *   适用于对消息处理准确性和可靠性要求高的应用场景，如金融交易系统、订单处理系统等。
4.  **手动提交的缺点**：
    
    *   需要额外的代码来管理偏移量的提交，增加了实现的复杂性。
    *   需要确保在正确的时间点提交偏移量，避免消息重复消费或丢失。

### 3、SESSION\_TIMEOUT\_MS_CONFIG

*   **目的**：`SESSION_TIMEOUT_MS_CONFIG` 控制了消费者在与 Kafka 代理失去联系之前的最大时间。消费者在这段时间内必须向代理发送心跳，以表明自己仍然活跃。
    
*   **默认值**：通常默认值是 10000 毫秒（10 秒）。
    
*   **与 heartbeat.interval.ms 的关系**： `heartbeat.interval.ms`（心跳间隔）必须小于 `session.timeout.ms`，以确保消费者在会话超时之前能够发送足够的心跳消息。
    

### 4、REQUEST\_TIMEOUT\_MS_CONFIG

*   **目的**：`REQUEST_TIMEOUT_MS_CONFIG` 控制消费者向代理发送请求后等待响应的最长时间。如果在这段时间内没有收到响应，请求将超时并抛出异常。
    
*   **默认值**：通常默认值是 30000 毫秒（30 秒）。
    
*   **与其他参数的关系**： `session.timeout.ms` 应小于 `request.timeout.ms`，因为消费者会话超时应该在请求超时之前发生，以便及时检测到消费者失效。
    

### 5、HEARTBEAT\_INTERVAL\_MS_CONFIG

*   **目的**：该参数设置消费者发送心跳消息的频率，以便 Kafka 代理能够检测消费者的活跃状态，并在需要时进行重新平衡。
    
*   **默认值**：通常默认值为 3000 毫秒（3 秒）。
    
*   **与 session.timeout.ms 的关系**：
    
    *   `heartbeat.interval.ms` 必须小于 `session.timeout.ms`。例如，如果 `session.timeout.ms` 设置为 10 秒，`heartbeat.interval.ms` 应设置为小于 10 秒的值，例如 3 秒。
*   **与 max.poll.interval.ms 的关系**：
    
    *   `max.poll.interval.ms` 是消费者调用 `poll` 方法的最大间隔时间。虽然 `heartbeat.interval.ms` 主要与会话管理相关，但确保 `poll` 调用频率可以间接影响心跳的发送。

### 6、MAX\_POLL\_INTERVAL\_MS\_CONFIG

*   **目的**：该参数设置了消费者在调用 `poll` 方法之间的最大允许间隔时间，以确保消费者定期从 Kafka 代理获取消息。
    
*   **默认值**：通常默认值为 300000 毫秒（5 分钟）。
    
*   **与其他参数的关系**：
    

**`session.timeout.ms`**：session.timeout.ms 控制消费者会话的超时时间，应小于 max.poll.interval.ms。消费者必须在会话超时之前发送心跳消息以维持会话状态。

**`heartbeat.interval.ms`**：heartbeat.interval.ms 控制心跳消息发送的间隔，应设置为小于 session.timeout.ms 的值。

### 7、GROUP\_ID\_CONFIG

配置kafka消费者的消费组名称

### 8、KEY\_DESERIALIZER\_CLASS\_CONFIG 和 VALUE\_DESERIALIZER\_CLASS\_CONFIG

配置key和value的数据类型

### 9、MAX\_POLL\_RECORDS_CONFIG

*   **目的**：该参数设置每次调用 `poll` 方法时从 Kafka 代理返回的最大消息数，以便消费者可以控制每次处理的消息批次大小。
    
*   **默认值**：通常默认值为 500 条消息。
    
*   **与其他参数的关系**：
    

**`max.poll.interval.ms`**：设置 max.poll.records 时，要确保消费者能在 max.poll.interval.ms 时间内处理完所有获取的消息。

**`fetch.min.bytes` 和 `fetch.max.bytes`**：这些参数控制从 Kafka 代理获取数据的大小，与 max.poll.records 一起决定了每次获取的消息量。

### 10、FETCH\_MIN\_BYTES_CONFIG

*   **目的**：该参数设置消费者每次调用 `poll` 方法时，从 Kafka 代理获取的最小数据量。如果可用数据量小于这个值，消费者会等待更多数据（或直到超时）才会返回数据。
    
*   **默认值**：通常默认值为 1 字节，意味着消费者会尽可能快地返回数据。
    
*   **与其他参数的关系**：
    

**`fetch.max.wait.ms`**：控制消费者等待 fetch.min.bytes 数据量的最大时间。如果在这个时间内没有足够的数据，代理会返回当前可用的数据。

**`fetch.max.bytes`**：控制每次从代理获取的最大数据量。fetch.min.bytes 应该小于或等于 fetch.max.bytes。

### 11、FETCH\_MAX\_WAIT\_MS\_CONFIG

*   **目的**：该参数设置消费者等待从 Kafka 代理获取数据的最大时间。如果在指定时间内没有足够的数据可用，消费者将返回当前可用的数据，而不会等待更长的时间。
    
*   **默认值**：通常默认值为 500 毫秒。
    
*   **使用场景**：
    
    **低延迟需求**：如果消费者需要尽快处理消息，可以将 `fetch.max.wait.ms` 设置得更低，以便更快地获取消息。
    
    **网络延迟较大**：如果网络延迟较大，可以适当增加 `fetch.max.wait.ms` 的值，以确保消费者在网络较差的情况下仍能获取足够的数据。
    
*   **与其他参数的关系**：
    

**`fetch.min.bytes`**：控制每次从代理获取的最小数据量。消费者会等待直到达到 `fetch.min.bytes` 或者超过 `fetch.max.wait.ms`。

**`fetch.max.bytes`**：控制每次从代理获取的最大数据量。确保 `fetch.max.wait.ms` 时间内可以获取到的数据不会超过 `fetch.max.bytes`。