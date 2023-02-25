# Apache Kafka Architecture - Delivery Guarantees
![](https://supergloo.com/wp-content/uploads/2018/11/hardertheycome_4350.jpg?ezimgfmt=rs%3Adevice%2Frscb1-1)

Apache Kafka offers message delivery guarantees between producers and consumers.  _For more background or information Kafka mechanics such as producers and consumers on this, please see [Kafka Tutorial](https://supergloo.com/kafka-tutorials/) page. _

Kafka delivery guarantees can be divided into three groups which include “at most once”, “at least once” and “exactly once”.

*   *   _**at most once**_ which can lead to messages being lost, but they cannot be redelivered or duplicated

*   *   _**at least once**_ ensures messages are never lost but may be duplicated
    

*   _**exactly once**_ guarantees each message processing (not delivery) happens once and only once

Which option sounds the most appealing?  The choice is obvious, right?

On the surface, everyone wants “exactly once”.    But, what if their overall equation includes more than just Kafka processing.  For example what if the architecture plan includes consuming from Kafka and writing to a data lake?  Now things become more complicated.

This is the point where I pause, reflect and smile and think of the old Jimmy Cliff song “You can get it if you really want it”.

#### Kafka Delivery Guarantee Considerations

Simply put, there are costs and considerations with each of the processing options.  Considerations include additional performance burdens which increase latency, the additional complexity of your implementation which may make your design more brittle, etc.

When considering, it is important to break things down into two different aspects which include durability guarantees provided for the message producer and the message consumer.

Also, although software stacks may promise to deliver each message once and only once or in other words, exactly once, the devil is in the details. Some delivery semantics of exactly once does not cover cases where a consumer writes to disk or external systems which fail outside of Kafka.  You need to consider you’re entire data flow use case before assuming exactly once.  

### Kafka Architecture – Producer Perspective

When a message has been published in Kafka, it is essentially stored in a log.  Kafka’s has the option to configure replication of messages through replication factor settings but it will not be covered here.

Since Kafka 0.11, Kafka producers includes two options for tuning your desired delivery guarantee: idempotent and transactional producers. The idempotent option will no longer introduce duplicates.  This option essentially changes Kafka’s delivery semantics from \`at least once\` to \`exactly once\`

The transactional producer allows an application to send messages to multiple partitions atomically.  Topics which are included in transactions should be configured for durability via \`replication.factor\` settings.  On the other side, consumers should be configured to read only committed messages.

These strong guarantees are not required by all use cases. When it comes to latency sensitive cases, Kafka allows the producer to set a certain level of durability. Should the producer require a waiting time in order for the message to be committed, this can take on the order of 10 ms. In addition, the [Kafka producer](https://supergloo.com/kafka-tutorials/kafka-producer/) could also specify the send to be completed absolutely asynchronously, or it wants to delay the send until the leader receives the message. The last, however, does not necessarily include the followers.

### Kafka Architecture – Consumer Perspective

Recall Kafka maintains a numerical offset for each record in a partition. This offset denotes the location of the consumer in the partition. For example, a consumer which is at position 5 has consumed records with offsets 0 through 4.  [Kafka Consumers](https://supergloo.com/kafka-tutorials//kafka-consumer/) may choose to store their current offset value in Kafka or outside Kafka through use of the \`enable.auto.commit\` configuration property.

Instead of relying on the consumer to periodically commit consumed offsets back to Kafka automatically, the Kafka Consumer API users can manually control their offsets positions. This is especially useful when the consumption of the messages is coupled with activities outside of Kafka such as writing to a data lake or warehouse.  In these cases, the consumer may choose to update their offset position only after a successful write to their data lake.  The alternative of automatic offset update is easier to implement but doesn’t account for downstream failures.  For example, if a write to the data lake fails but the consumer offset position is updated after reading from the Kafka topic, data loss in the data lake would occur.

Let’s dive into this a bit more because it is critical in your understanding of delivery guarantee semantics.

If we enable auto commit of offsets back to Kafka and fail between consuming from Kafka, it may look similar to the following in pseudo-code

ConsumerRecords<String, String> records = consumer.poll(100);
         for (ConsumerRecord<String, String> record : records)
             insertDataLake(record)

 Again, pseudo-code here for illustrative purposes only.

If a failure occurs in \`insertDataLake\` function above and the consumer is restarted with offset commits set to automatic, there is a chance of data loss at data lake because the offset will be greater than the number of messages that have been inserted to the data lake.  This is an example of \`at-most-once\` delivery.  There are no duplicates in the data lake, but there is the possibility of missing records.

It might be completely fine to deploy with a slight chance of data loss is acceptable.

Conversely, if choosing to manually commit offset location, you can move towards \`at-least-once\` delivery guarantee.  Each record will likely be delivered one time, but in event of a failure in between consumption to write to the data lake and committing back to Kafka, there is the possibility of storing duplicates in the data lake example.

props.put("enable.auto.commit", "false")
KafkaConsumer consumer = new KafkaConsumer(props)
ConsumerRecords<String, String> records = consumer.poll(100);
         for (ConsumerRecord<String, String> record : records)
             insertDataLake(record)
             // what happens if failure happens after this ^ but before the following completes?
             consumer.commitSync()

In this case, the consumer process that took over from failure would consume from last committed offset and would repeat the insert data. Now, Kafka provides “at-least-once” delivery guarantees, as each record will likely be delivered one time but in a failure case, data could be duplicated.

What hasn’t been covered until now is the choice of granularity with regards to committing offset locations.  First, we should know processing in batches of records can be much more performant (less latency) than processing individual records.  This is similar to architectural software predecessors such as using JDBC.  Processing in batches of records is available in Kafka as well.   This means you have the choice of level granularity of consuming and committing offsets in batches or individual records.

As hopefully expected, your choice of batch vs individual has consequences on performance.  There is much more overhead required to process individually rather than batch.  But, if we require “exactly once” delivery to our data lake, we may be forced to choose the individual record approach.  Alternatively, we may explore the possibility of implementing data lake with upsert support and Kafka consumer with “at least-once” processing.

To recap

##### Kafka At-Most-Once

Read the message and save its offset position before it possibly processes the message record in entirety; i.e. save to data lake. In case of failure, the consumer recovery process will resume from the previously saved offset position which is further beyond the last record saved to the data lake. This example demonstrates at-most-once semantics.  Data loss at data lake, for example, is possible.

##### Kafka At-Least-Once

The consumer can also read, process the messages and save to data lake, and afterward save its offset position.  In a failure scenario which occurs after the data lake storage and offset location update, a Kafka consumer addressing the failed process could duplicate data. This forms the “at-least-once” semantics in case of consumer failure.  Duplicates in the data lake could happen.

##### Kafka Exactly Once

When it comes to exactly once semantics with Kafka and external systems, the restriction is not necessarily a feature of the messaging system, but the necessity for coordination of the consumer’s position with what is stored in the destination.  For example, a destination might be in an HDFS or object store based data lake. One of the most common ways to do this is by introducing a two-phase commit between the consumer output’s storage and the consumer position’s storage.  Or in other words, taking control of offset location commits in relation to data lake storage on a per record basis.  

#### Kafka Delivery Conclusion

By default, Kafka provides at-least-once delivery guarantees.  Users may implement at-most-once and exactly once semantics by customizing offset location recording.  In any case, Kafka offers you the option for choosing which delivery semantics you strive to achieve.