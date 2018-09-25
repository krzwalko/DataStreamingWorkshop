# Kafka Connect - Data Streaming Workshop

Using console producers and consumers is good way to start play with Kafka, but for real applications we will not use a terminal console. We have to put data into Kafka in other way. A special framework for data synchronization called Kafka Connect will effectively load (and eventually transform) data from/to Kafka.

We have running connect in distributed mode (recommended), other way is connect standalone, but this is not fault tolerant and scalable, can run only on single node.

```
$ confluent status

This CLI is intended for development only, not for production
https://docs.confluent.io/current/cli/index.html

ksql-server is [UP]
connect is [UP]
kafka-rest is [UP]
schema-registry is [UP]
kafka is [UP]
zookeeper is [UP]
```

To communicate with Kafka Connect distributed cluster (in our case cluster is one-node), we use REST API. We will start with simple file source connector.

### Managing connectors

To list of all connectors on cluster:
```
$ curl localhost:8083/connectors | jq .
[]
```

We create a text file and we want to synchronize content to Kafka. File will be named `temp.txt` with example content (delimited by commas):

```
Room1,20
Room2,20
Room3,20
Room4,20
Room5,20
```

Then create JSON file which will be applied to Connect cluster and save it under `file.json` filename.

```json
{
    "name": "file",
    "config": {
        "connector.class": "FileStreamSource",
        "tasks.max": 1,
        "file": "/Users/krzwalko/Dev/DataStreamingWorkshop/temp.txt",
        "topic": "temp",
        "key.converter": "org.apache.kafka.connect.storage.StringConverter",
        "value.converter": "org.apache.kafka.connect.storage.StringConverter"
    }
}
```

To create connector using configuration file

```
$ curl -XPOST localhost:8083/connectors -H 'Content-Type: application/json' -d @file.json
```

Check if configuration was applied:

```
$ curl localhost:8083/connectors/file | jq .

{
  "name": "file",
  "config": {
    "connector.class": "FileStreamSource",
    "file": "/Users/krzwalko/Dev/DataStreamingWorkshop/temp.txt",
    "tasks.max": "1",
    "name": "file",
    "topic": "temp",
    "value.converter": "org.apache.kafka.connect.storage.StringConverter",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter"
  },
  "tasks": [
    {
      "connector": "file",
      "task": 0
    }
  ],
  "type": "source"
}
```

To check status of connector

```
$ curl localhost:8083/connectors/file/status | jq .

{
  "name": "file",
  "connector": {
    "state": "RUNNING",
    "worker_id": "127.0.0.1:8083"
  },
  "tasks": [
    {
      "state": "RUNNING",
      "id": 0,
      "worker_id": "127.0.0.1:8083"
    }
  ],
  "type": "source"
}
```

When connection is running and we have data in file, topic should be created and content from file should be copied into Kafka.

```
$ kafka-topics --zookeeper localhost:2181 --describe --topic temp

Topic:temp	PartitionCount:1	ReplicationFactor:1	Configs:
	Topic: temp	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

```
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic temp --from-beginning

Room1,20
Room2,20
Room3,20
Room4,20
Room5,20
```

Once you write new lines into file, it will be continously added to topic. Beware that offset is tracked just by number of byte in file.

We can pause and resume connector:

```
$ curl -XPUT localhost:8083/connectors/file/pause
$ curl -XPUT localhost:8083/connectors/file/resume
```


Finally to delete connector

```
$ curl -XDELETE localhost:8083/connectors/file
```
