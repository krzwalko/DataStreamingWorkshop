# Kafka Connect - Social Media Data - Data Streaming Workshop

### Pre-requisite

We will need a Twitter account with developer profile. We need generate OAuth credentials for our application.

Additionally we will need an additional Kafka Connect Connector. We will use jcustenborder/kafka-connect-twitter available here: https://github.com/jcustenborder/kafka-connect-twitter. We have to put JAR files into `share/java/kafka-connect-twitter` directory and restart Kafka Connect.


### Create Connect configuration file

As in previous example connect conf file is a JSON file saved as `twitter.json`. Example is here:

```json
{
  "name": "twitter",
  "config": {
    "connector.class": "com.github.jcustenborder.kafka.connect.twitter.TwitterSourceConnector",
    "tasks.max": "1",
    "errors.log.include.messages": "true",
    "twitter.oauth.accessToken": "<access token>",
    "twitter.oauth.accessTokenSecret": "<access token secret>",
    "twitter.oauth.consumerKey": "<consumer key>",
    "twitter.oauth.consumerSecret": "<consumer secret>",
    "twitter.debug": "true",
    "process.deletes": "false",
    "filter.keywords": "kafka",
    "kafka.status.topic": "twitter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false",
    "key.converter.schemas.enable": "false",
    "errors.log.enable": "true"
  }
}
```

### Create connector using REST API

To create and start connector use curl REST API as in previous example:

```
$ curl -XPOST localhost:8083/connectors -H 'Content-Type: application/json' -d @twitter.json
```

Check status if connector is running, eventually check connect logs:

```
$ curl localhost:8083/connectors/twitter/status | jq .

{
  "name": "twitter",
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

Something like below is correct, seems that connector is started properly and started to fetch some tweets

```
$ confluent log connect -n 200 -f

[2018-09-22 00:11:49,902] INFO TwitterSourceConnectorConfig values:
	filter.keywords = [kafka]
	kafka.status.topic = twitter
	process.deletes = false
	twitter.debug = true
	twitter.oauth.accessToken = [hidden]
	twitter.oauth.accessTokenSecret = [hidden]
	twitter.oauth.consumerKey = [hidden]
	twitter.oauth.consumerSecret = [hidden]
 (com.github.jcustenborder.kafka.connect.twitter.TwitterSourceConnectorConfig:279)
[2018-09-22 00:11:49,950] INFO Setting up filters. Keywords = kafka (com.github.jcustenborder.kafka.connect.twitter.TwitterSourceTask:61)
[2018-09-22 00:11:49,951] INFO Starting the twitter stream. (com.github.jcustenborder.kafka.connect.twitter.TwitterSourceTask:68)
[2018-09-22 00:11:49,953] INFO WorkerSourceTask{id=twitter-0} Source task finished initialization and start (org.apache.kafka.connect.runtime.WorkerSourceTask:199)
[2018-09-22 00:11:49,953] INFO Establishing connection. (twitter4j.TwitterStreamImpl:62)
[2018-09-22 00:12:06,927] INFO Connection established. (twitter4j.TwitterStreamImpl:62)
[2018-09-22 00:12:06,927] INFO Receiving status stream. (twitter4j.TwitterStreamImpl:62)
```

Let's acually see if Tweets are really imported into Kafka.

```
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic twitter --from-beginning
```

Then when we Tweet something with keyword, it should be visible in our topic.

```
{"CreatedAt":1537568145000,"Id":1043262532956377089,"Text":"KWK Kafka kopie na poziomie 1000!","Source":"<a href=\"http://twitter.com\" rel=\"nofollow\">Twitter Web Client</a>","Truncated":false,"InReplyToStatusId":-1,"InReplyToUserId":-1,"InReplyToScreenName":null,"GeoLocation":null,"Place":null,"Favorited":false,"Retweeted":false,"FavoriteCount":0,"User":{"Id":1015179610508681216,"Name":"ukasz","ScreenName":"ukasz96771829","Location":null,"Description":"I am what I am","ContributorsEnabled":false,"ProfileImageURL":"http://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png","BiggerProfileImageURL":"http://abs.twimg.com/sticky/default_profile_images/default_profile_bigger.png","MiniProfileImageURL":"http://abs.twimg.com/sticky/default_profile_images/default_profile_mini.png","OriginalProfileImageURL":"http://abs.twimg.com/sticky/default_profile_images/default_profile.png","ProfileImageURLHttps":"https://abs.twimg.com/sticky/default_profile_images/default_profile_normal.png","BiggerProfileImageURLHttps":"https://abs.twimg.com/sticky/default_profile_images/default_profile_bigger.png","MiniProfileImageURLHttps":"https://abs.twimg.com/sticky/default_profile_images/default_profile_mini.png","OriginalProfileImageURLHttps":"https://abs.twimg.com/sticky/default_profile_images/default_profile.png","DefaultProfileImage":false,"URL":null,"Protected":false,"FollowersCount":0,"ProfileBackgroundColor":"F5F8FA","ProfileTextColor":"333333","ProfileLinkColor":"1DA1F2","ProfileSidebarFillColor":"DDEEF6","ProfileSidebarBorderColor":"C0DEED","ProfileUseBackgroundImage":true,"DefaultProfile":true,"ShowAllInlineMedia":false,"FriendsCount":1,"CreatedAt":1530872655000,"FavouritesCount":0,"UtcOffset":-1,"TimeZone":null,"ProfileBackgroundImageURL":"","ProfileBackgroundImageUrlHttps":"","ProfileBannerURL":null,"ProfileBannerRetinaURL":null,"ProfileBannerIPadURL":null,"ProfileBannerIPadRetinaURL":null,"ProfileBannerMobileURL":null,"ProfileBannerMobileRetinaURL":null,"ProfileBackgroundTiled":false,"Lang":"en","StatusesCount":21,"GeoEnabled":false,"Verified":false,"Translator":false,"ListedCount":0,"FollowRequestSent":false,"WithheldInCountries":[]},"Retweet":false,"Contributors":[],"RetweetCount":0,"RetweetedByMe":false,"CurrentUserRetweetId":-1,"PossiblySensitive":false,"Lang":"in","WithheldInCountries":[],"HashtagEntities":[],"UserMentionEntities":[],"MediaEntities":[],"SymbolEntities":[],"URLEntities":[]}
```

