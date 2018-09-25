# KSQL - Anomaly Detection - Data Streaming Workshop

### Into to KSQL

KSQL is a streaming SQL engine, is uses some foundations from SQL but it's not compatible, because SQL is designed to work static data, KSQL is designed to work with streaming.

### Running KSQL server and connecting to KSQL console

```
$ ksql

                  ===========================================
                  =        _  __ _____  ____  _             =
                  =       | |/ // ____|/ __ \| |            =
                  =       | ' /| (___ | |  | | |            =
                  =       |  <  \___ \| |  | | |            =
                  =       | . \ ____) | |__| | |____        =
                  =       |_|\_\_____/ \___\_\______|       =
                  =                                         =
                  =  Streaming SQL Engine for Apache Kafka® =
                  ===========================================

Copyright 2017-2018 Confluent Inc.

CLI v5.0.0, Server v5.0.0 located at http://localhost:8088
```

To be able to read data from beginning (it will be useful in our workshop), set `auto.offset.reset` to `earliest`:

```
ksql> SET 'auto.offset.reset' = 'earliest';

Successfully changed local property 'auto.offset.reset' from 'null' to 'earliest'
```

### Showing topics, streams, tables

```
ksql> SHOW TOPICS;


 Kafka Topic      | Registered | Partitions | Partition Replicas | Consumers | ConsumerGroups
----------------------------------------------------------------------------------------------
 _schemas         | false      | 1          | 1                  | 0         | 0
 connect-configs  | false      | 1          | 1                  | 0         | 0
 connect-offsets  | false      | 25         | 1                  | 0         | 0
 connect-statuses | false      | 5          | 1                  | 0         | 0
 temp             | false      | 1          | 1                  | 1         | 1
 test             | false      | 4          | 1                  | 0         | 0
 twitter          | true       | 1          | 1                  | 0         | 0
----------------------------------------------------------------------------------------------
```

```
ksql> SHOW STREAMS;

 Stream Name | Kafka Topic | Format
------------------------------------
------------------------------------
```

```
ksql> SHOW TABLES;

 Table Name | Kafka Topic | Format | Windowed
----------------------------------------------
----------------------------------------------
```

### Printing topic contents

```
ksql> PRINT 'temp' FROM BEGINNING;

Format:STRING
9/21/18 10:18:10 PM CEST , NULL , Room1,20
9/21/18 10:18:10 PM CEST , NULL , Room2,21
9/21/18 10:18:10 PM CEST , NULL , Room3,25
9/21/18 10:21:13 PM CEST , NULL , Room4,40
^CTopic printing ceased
```

### Creating streams from topic

```
ksql> CREATE STREAM temp (room STRING, temp INT) with (KAFKA_TOPIC='temp',VALUE_FORMAT='DELIMITED');

 Message
----------------
 Stream created
----------------
```

### Display created stream:

```
ksql> SHOW STREAMS;

 Stream Name | Kafka Topic | Format
---------------------------------------
 TEMP        | temp        | DELIMITED
```

### Describe stream

```
ksql> DESCRIBE TEMP;

Name                 : TEMP
 Field   | Type
-------------------------------------
 ROWTIME | BIGINT           (system)
 ROWKEY  | VARCHAR(STRING)  (system)
 ROOM    | VARCHAR(STRING)
 TEMP    | INTEGER
-------------------------------------
For runtime statistics and query details run: DESCRIBE EXTENDED <Stream,Table>;
```

### Select data from stream

```
ksql> SELECT * FROM temp;
1537561090735 | null | Room1 | 20
1537561090736 | null | Room2 | 21
1537561090736 | null | Room3 | 25
```

This is continous query, you have to stop it (or limit).


### Create table (aggregated stream) as a select

```
ksql> CREATE TABLE avgtemp WITH (PARTITIONS=1) AS SELECT room, SUM(temp)/COUNT(temp) AS avgtemp FROM temp GROUP BY room;

 Message
---------------------------
 Table created and running
---------------------------
```

We can now query table like normal table in SQL. Difference here is this query never stops.
```
ksql> SELECT * FROM avgtemp;
1537566224286 | Room7 | Room7 | 25
1537565944500 | Room3 | Room3 | 20
1537565944501 | Room5 | Room5 | 20
1537565944499 | Room1 | Room1 | 20
```

Once new data are produced, averages are calculated and we see new averages. But if you re-run query you will see only latest values. This is how table in KSQL works, it represents state, not log of events.

### Create stream of alerts (using LEFT JOIN)

Then we want to build some alerting if temperature is different than acutal calculated average for a room. We can use LEFT JOIN for stream and table. LEFT join will work as table lookup - we enrich stream data with some table lookup data. Table data are also generated dynamically, in this case it's generated from the same stream (by calculating total avegarage per room).

```
ksql> CREATE STREAM anomaly AS SELECT temp.room, avgtemp.avgtemp, temp.temp, 'ALERT' FROM temp LEFT JOIN avgtemp ON temp.room = avgtemp.room WHERE ABS(temp.temp-avgtemp.avgtemp) > 10;

 Message
----------------------------
 Stream created and running
----------------------------
```


We have now:
- TEMP - input stream with temperatures from sensors (imported into Kafka with Kafka Connect)
- AVGTEMP - table with average temperatures per room computes continously
- ANOMALY - output stream with alerts where messages are emitted only when difference between average and current temperature is more than 10.
Each of construct has a Topic and can be read outside of KSQL, using standard Consumer API

We can start 3 KSQL session and run queries:

```
ksql> SELECT * FROM temp;
ksql> SELECT * FROM avgtemp;
ksql> SELECT * FROM anomaly;
```

And produce some data from sensors and see how anomalies are handles and alerted...

We can also read data from topics outside of KSQL, beacuse our tables and streams are running within KSQL server engine, but it can be consumer by any custom client (for example trigger some reaction on alerts). In this case, we can use `kafka-console-consumer`.

```
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic temp --from-beginning
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic AVGTEMP --from-beginning
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic ANOMALY --from-beginning
```

