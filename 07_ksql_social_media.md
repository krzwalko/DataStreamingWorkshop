# KSQL - Social Media Data - Data Streaming Workshop

We have now Tweets in topic - in raw form, let's acually work with them to make some analysis and react in near-time to some trends in social media.

First, let's start KSQL and create a stream from a topic. We have data in JSON, but we are interested only in some basic fields, not whole content of Tweet.

```
ksql> CREATE STREAM twitter (id BIGINT, lang STRING, text STRING) WITH (KAFKA_TOPIC='twitter', VALUE_FORMAT='JSON');

 Message
----------------
 Stream created
----------------
```

Check if Tweets are visible as stream

```
ksql> SELECT * FROM twitter;

1537567927215 | {"Id":1043261615292993537} | 1043261615292993537 | tr | RT @muthispsikoloji: Franz Kafka'nın Başından Geçen Müthiş Bir Hikâye. https://t.co/Tsf9AA7BxL
1537567947363 | {"Id":1043261700601012229} | 1043261700601012229 | en | @alokranj @magistrabeck Can't argue with "free," but Stanley's written half a dozen books on Kafka since that oldie-but-goodie.
```

Let's do some filtering and extract phrase as separate field. First we create a stream. Stream will create also a topic.

```
ksql> CREATE STREAM twitter_phrase AS SELECT 'KWK' AS phrase, * FROM twitter WHERE text LIKE '%KWK%';

 Message
----------------------------
 Stream created and running
----------------------------
```

When topic is created, we can create insert queries to track new phrase and extract it as separate field. This is nice concept of aggregating different streams into one topic.
```
ksql> INSERT INTO twitter_phrase SELECT 'Franz' AS phrase, * FROM twitter WHERE text LIKE '%Franz%';

 Message
-------------------------------
 Insert Into query is running.
-------------------------------
```

Add next phrase:
```
ksql> INSERT INTO twitter_phrase SELECT 'Apache' AS phrase, * FROM twitter WHERE text LIKE '%Apache%';

 Message
-------------------------------
 Insert Into query is running.
-------------------------------
```

After all we have tweets with phrases, so we can make aggregations:
```
ksql> SELECT phrase, COUNT(*) FROM twitter_phrase GROUP BY phrase;
Apache | 2
Franz | 10
KWK | 2
```

We now want to react on Tweets on particular keyword or phrase, but only when number of Tweets in some period of time increased, we will use windowed tables:

```
ksql> CREATE TABLE twitter_alerts AS SELECT phrase, COUNT(*) AS count FROM twitter_phrase WINDOW TUMBLING (SIZE 10 MINUTES) GROUP BY phrase HAVING COUNT(*) >= 3;

 Message
---------------------------
 Table created and running
---------------------------
```

```
ksql> SELECT timestamptostring(rowtime, 'yyyy-MM-dd HH:mm:ss.SSS') AS ts, rowkey, phrase, count from twitter_alerts;
2018-09-22 00:45:38.303 | Apache : Window{start=1537569600000 end=-} | Apache | 7
2018-09-22 00:33:33.341 | Franz : Window{start=1537569000000 end=-} | Franz | 4
2018-09-22 00:49:00.378 | Franz : Window{start=1537569600000 end=-} | Franz | 4
```

