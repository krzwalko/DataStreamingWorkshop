# Kafka Connect - Replication - Data Streaming Workshop

Another way to get Tweets into your Kafka cluster is to get it from another Kafka cluster using replication. We will use Kafka Connect framework to replicate topic from one cluster to another. 

There is official Confluent connector available at https://www.confluent.io/connector/confluent-kafka-replicator/. After downloading and unpacking JAR files from lib directory should be placed into `share/java/kafka-connect-replicator` (create this directory) and Kafka Connect should be restarted to read new connector plugins. 

To check if connector plugin is installed run query:

```
$ curl localhost:8083/connector-plugins | jq .
```

```json
[
  {
    "class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "type": "sink",
    "version": "5.0.0"
  },
  {
    "class": "io.confluent.connect.hdfs.HdfsSinkConnector",
    "type": "sink",
    "version": "5.0.0"
  },
  {
    "class": "io.confluent.connect.hdfs.tools.SchemaSourceConnector",
    "type": "source",
    "version": "2.0.0-cp1"
  },
  {
    "class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "type": "sink",
    "version": "5.0.0"
  },
  {
    "class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "type": "source",
    "version": "5.0.0"
  },
  {
    "class": "io.confluent.connect.replicator.ReplicatorSourceConnector",
    "type": "source",
    "version": "5.0.0"
  },
  {
    "class": "io.confluent.connect.s3.S3SinkConnector",
    "type": "sink",
    "version": "5.0.0"
  },
  {
    "class": "io.confluent.connect.storage.tools.SchemaSourceConnector",
    "type": "source",
    "version": "2.0.0-cp1"
  },
  {
    "class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
    "type": "sink",
    "version": "2.0.0-cp1"
  },
  {
    "class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
    "type": "source",
    "version": "2.0.0-cp1"
  }
]
```

Once we have connector plugin available, we can create configuration file like below (saved as replicator.json). `src.kafka.bootstrap.servers` should be changed to proper IP address.

```json
{
    "name": "replicator",
    "config": {
        "connector.class": "io.confluent.connect.replicator.ReplicatorSourceConnector",
        "key.converter": "io.confluent.connect.replicator.util.ByteArrayConverter",
        "value.converter": "io.confluent.connect.replicator.util.ByteArrayConverter",
		"confluent.topic.replication.factor": "1",
        "src.kafka.bootstrap.servers": "192.168.8.118:19092",
        "dest.kafka.bootstrap.servers": "localhost:9092",
        "topic.whitelist": "twitter"
    }
}
```

And run the connector:

```
$ curl -XPOST localhost:8083/connectors -H 'Content-Type: application/json' -d @replicator.json
```

Topic twitter should be now replicated from source Kafka cluster.



### Note on Source Kafka

On source Kafka proper configuration should be added to add `EXTERNAL` listener. This islocated in `etc/kafka/server.properties` file:

```
listeners=PLAINTEXT://localhost:9092,EXTERNAL://0.0.0.0:19092
listener.security.protocol.map=PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
advertised.listeners=PLAINTEXT://localhost:9092,EXTERNAL://192.168.8.118:19092
```

