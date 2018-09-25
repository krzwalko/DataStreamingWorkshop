# Managing Kafka topics, consumers and producers - Data Streaming Workshop

Data in Kafka is organized in topics. Topic can be treated as a queue with events. Once events are written to topic, they are immutable, there is no way to change already written data to Kafka topic. Each topic is divided into one or partitions. Each partition can be located at different broker (Kafka server). For better redundancy each partition can be also replicated to avoid data loss. Order delivery is guaranteed only within partitions, not within the topics!

### Managing topics

When platform is running, to list topics, run `kafka-topics` tool:
```
$ kafka-topics --zookeeper localhost:2181 --list

__confluent.support.metrics
__consumer_offsets
_confluent-ksql-default__command_topic
_schemas
connect-configs
connect-offsets
connect-statuses
```

To describe more information about topics, use `--describe` option:

```
$ kafka-topics --zookeeper localhost:2181 --describe

Topic:connect-statuses	PartitionCount:5	ReplicationFactor:1    Configs:cleanup.policy=compact
	Topic: connect-statuses	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: connect-statuses	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: connect-statuses	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: connect-statuses	Partition: 3	Leader: 0	Replicas: 0	Isr: 0
	Topic: connect-statuses	Partition: 4	Leader: 0	Replicas: 0	Isr: 0
Topic:twitter	PartitionCount:1	ReplicationFactor:1	Configs:
	Topic: twitter	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

or using `kafkacat` tool:

```
$ kafkacat -b localhost -L

Metadata for all topics (from broker 0: localhost:9092/0):
 1 brokers:
  broker 0 at localhost:9092
 8 topics:
  topic "_schemas" with 1 partitions:
    partition 0, leader 0, replicas: 0, isrs: 0
  topic "connect-statuses" with 5 partitions:
    partition 0, leader 0, replicas: 0, isrs: 0
    partition 1, leader 0, replicas: 0, isrs: 0
    partition 2, leader 0, replicas: 0, isrs: 0
    partition 3, leader 0, replicas: 0, isrs: 0
    partition 4, leader 0, replicas: 0, isrs: 0
  topic "connect-configs" with 1 partitions:
```

### Creating topics

By default automatic topic creation is enabled in Kafka, but you can have more control how topics are created, use command `kafka-topics` to create a topic:

```
$ kafka-topics --zookeeper localhost:2181 --create --topic test --partitions 4 --replication-factor 1

Created topic "test".
```

To check newly created topic:

```
$ kafka-topics --zookeeper localhost:2181 --describe --topic test
Topic:test	PartitionCount:4	ReplicationFactor:1	Configs:
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 3	Leader: 0	Replicas: 0	Isr: 0
```

### Consuming data

Two main APIs for Kafka are Consumer API and Producer API. There is no surprise that is used for writing data to cluster and second one is for reading. There is simple consumer program which can produce data from terminal console. We can use also kafkacat tool.

Use `kafka-console-consumer` to read data from a topic:

```
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning
```

Using `kafkacat`:

```
$ kafkacat -b localhost -C -t test -o beginning

% Auto-selecting Consumer mode (use -P or -C to override)
% Reached end of topic test [0] at offset 0
% Reached end of topic test [1] at offset 0
% Reached end of topic test [2] at offset 0
% Reached end of topic test [3] at offset 0
```

Do you see any data? It waits, because there are no data written to topics, so let's write something.

### Producing data

Again, we can use `kafka-console-producer` tool (default) or `kafkacat`. To produce some data from a console to a topic using `kafka-console-producer`:

```
$ kafka-console-producer --broker-list localhost:9092 --topic test

>Hello world!
>Hello again
```

You should now see at previously running consumers that some data are actually read from a topic.



### Using consumer groups

Kafka is able to process a lot of data in near-real time and be highly scalable and fault tolerant. This is achieved by concept of partitions and consumer groups. Cosnumers can be dynamically rebalanced when belogs to the same consumer group. To demonstrate this, let's do the following:


Run two consumers using particular consumer group, by using `kafka-console-consumer` (on separate terminal consoles):

```
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning --group group1
```

or `kafkacat`:

```
$ kafkacat -b localhost -G group1 test

% Group group1 rebalanced (memberid rdkafka-b9640eed-eea3-4042-82ef-30d35a75f3e2): assigned: test [0], test [1], test [2], test [3]
% Reached end of topic test [0] at offset 0
% Reached end of topic test [1] at offset 0
% Reached end of topic test [2] at offset 0
% Reached end of topic test [3] at offset 0
```

List current consumer groups:
```
$ kafka-consumer-groups --bootstrap-server localhost:9092 --list
```

Describe our consumer group:

```
$ kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group group1

TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                     HOST            CLIENT-ID
test            2          4               4               0               consumer-1-8c9535b8-9af0-4d98-8cab-f8f8d319d011 /127.0.0.1      consumer-1
test            3          4               4               0               consumer-1-8c9535b8-9af0-4d98-8cab-f8f8d319d011 /127.0.0.1      consumer-1
test            0          4               4               0               consumer-1-71c052d3-e1de-4529-9d3a-61fe09f56513 /127.0.0.1      consumer-1
test            1          5               5               0               consumer-1-71c052d3-e1de-4529-9d3a-61fe09f56513 /127.0.0.1      consumer-1
```

Show members of consumer group:

```
$ kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group group1 --members

CONSUMER-ID                                     HOST            CLIENT-ID       #PARTITIONS
consumer-1-94370965-8090-436a-bef0-daafc2e2a1bd /127.0.0.1      consumer-1      2
consumer-1-aeca0785-fa5b-4cbd-bb5f-11fbb77dd997 /127.0.0.1      consumer-1      2
```

Then produce some messages when both consumers are active. You should see that messages are distributed by two consumers. If you stop one, then partitions will be rebalances and second will handle the traffic. If you stop both consumers, produce some data, they will be waiting in topic to be consumed, during that time you will see lag in output of `kafka-consumer-groups`.

When consumers are stopped, but some messages are written to topic partitions, we can observe lag:

```
$ kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group group1

Consumer group 'group1' has no active members.

TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
test            1          5               6               1               -               -               -
test            3          4               6               2               -               -               -
test            0          4               6               2               -               -               -
test            2          4               5               1               -               -               -
```

### Deleting topic

First, stop consumers, because topic will be recreated, additionally you will see errors on existing consumer console.

```
$ kafka-topics --zookeeper localhost:2181 --delete --topic test

Topic test is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
```

