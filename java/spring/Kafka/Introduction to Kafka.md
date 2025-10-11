# Introduction to Kafka
Imagine you built an e-commerce application using a microservice design pattern. The Communication between the different services of your application might look like this.

![](_assets/1!BqbaTHfPELtUbDw8uDFmVA.png)

There will be lots of connections between the services. Some services will send and receive different data formats. Managing this type of communication can be very hard. This can lead to errors and loss of data.

To solve this problem, your application services can communicate with each other using a publisher-subscriber messaging system.

**Kafka** is a publisher-subscriber messaging system. It enables the exchange of data between different services, servers, and applications. This article explains the basic components of Kafka.

![](_assets/1!QEN2aNpv3-P8ez-7Nled8A.png)

In a publisher-subscriber messaging system like Kafka, services would focus on either sending (publishing) or receiving (consuming) messages. This would make it easier to manage the connection.

![](_assets/1!LR-O9aqtHYQuS1Tqno-aHA.png)

Instead of your services communicating directly with each other, services will communicate via a message broker. The producers send messages to the Broker. While the consumers pull messages from the **Brokers**.

Kafka Broker
------------

A Broker is a server on which Kafka runs. A broker serves as a middleman between the producers and the consumers. A collection of Kafka brokers is called a **Kafka cluster**.

Having many brokers in your system, increase the fault tolerance and scalability of your system.

Messages sent to a broker by producers are stored in a **Kafka topic.**

![](_assets/1!bb45Cx3RD36XN-LxOhn0VQ.png)

**Kafka Topic**
---------------

> You can think of a **Kafka Topic** as a folder in a file system or a table in a database. —

A topic holds a series of related messages/data. Using the example above, you can create a topic for orders, payments, new registrations, customer complaints e.t.c

So for example, if a customer places a new order, the order producer sends a new message to the broker. The broker then stores the message in the order topic.

You can create as many topics as you want.

A topic is also divided into different parts. The different parts hold different types of messages. For example, in the order topic, there will be separate parts for _new orders, canceled orders, delivered orders e.t.c_

These different parts of a topic are called **Kafka** **partitions**.

![](_assets/1!KA84Vn8VdWIILHXU0hU3ag.png)

Kafka Partition
---------------

A Kafka partition is an ordered sequence of messages. It has a queue structure. This means that messages in a partition are stored in the sequence in which they come. New messages are added to the back of the queue.

The messages stored in a partition are called **records**. Each record has a unique key called **offset.** _Think of it as a record ID._

A _partition ID_ is attached to the messages sent to a broker. The partition ID tells the broker which partition to store the message.

The partitions of a topic are distributed across different brokers. One broker doesn’t hold all the partitions of a topic. This enables Kafka to replicate the partitions across many brokers. So if one broker fails, the message is still available on other brokers. This helps to eliminate data loss.

Among the replicas of a partition, one of them acts as a leader while the rest are followers. The messages are sent to the leader. Then is replicated among the followers.

Conclusion
----------

Managing point-to-point communication between services in a large system can be very hard. Splitting services into producers and consumers helps.

Kafka comes in as a middleman between the producers and consumers. It helps to manage the messages between them in a reliable manner.
