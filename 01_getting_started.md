# Getting started - Data Streaming Workshop

## Pre-requisites

1. Download Confluent Open Source 5.0 from https://www.confluent.io/download/. You will need Java 1.8 installed, Kafka and rest of components are running on the top of JVM platform.

2. Create Twitter account and apply for developer account, go to https://apps.twitter.com/ and click "Apply for a developer account". You need to add valid email, phone number and provide some description of your application.


3. Some additional useful tools: 
   - **kafkacat** - simple tool for listing topics, reading and writing data to kafka. This is non-JVM Kafka client, it's faster than stardard Kafka distribution tools
   - **jq** - JSON parser (we will use it for pretty print)
   - **curl** - standard console http client

## Running Confluent Platform locally with CLI

We will run Confluent Platform locally using CLI, this is recommended only for development, for running Kafka in production we have to start multiple Zookeeper instances and Kafka broker instances.

For ease of use I have adjusted my `PATH` env variable. Additionally I set `CONFLUENT_CURRENT` to persist data across system restart.
Below is configuration I have in my `.bash_profile`.


```
export PATH="$PATH:~/Dev/kafka/confluent-5.0.0/bin"
export CONFLUENT_CURRENT="/Users/krzwalko/Dev/kafka/confluent-5.0.0/data"
```

To check status of platform

```
$ confluent status

This CLI is intended for development only, not for production
https://docs.confluent.io/current/cli/index.html

ksql-server is [DOWN]
connect is [DOWN]
kafka-rest is [DOWN]
schema-registry is [DOWN]
kafka is [DOWN]
zookeeper is [DOWN]
```

Start platform (all components)

```
$ confluent start

This CLI is intended for development only, not for production
https://docs.confluent.io/current/cli/index.html

Using CONFLUENT_CURRENT: /Users/krzwalko/Dev/kafka/confluent-5.0.0/data/confluent.GBFm8bYt
Starting zookeeper
zookeeper is [UP]
Starting kafka
kafka is [UP]
Starting schema-registry
schema-registry is [UP]
Starting kafka-rest
kafka-rest is [UP]
Starting connect
connect is [UP]
Starting ksql-server
ksql-server is [UP]
```

To check after start

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

To check log files on individual component (in this example show only last 10 lines and follow output)

```
$ confluent log kafka -f
```

or to follow only last 200 lines:

```
$ confluent log kafka -n 200 -f
```



