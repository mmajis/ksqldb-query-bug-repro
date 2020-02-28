**Describe the bug**
Running a push query in ksqldb v0.7.0 errors out with `Error parsing host address http://<docker network host address>:8088/. Expected format host:port.`

It looks like `StreamsConfig.APPLICATION_SERVER_CONFIG` has wrong format when executing the query.

**To Reproduce**
Steps to reproduce the behavior, include:
0. Clone this repo: 
1. Put your confluent cloud api key and secret in the `.env` file inside the repo
1. `docker-compose up -d`
2. `docker-compose exec ksqldb-cli bash`
3. `LOG_DIR=. ksqldb http://ksql-server:8088`
4. Create a stream:
```
  CREATE STREAM my_stream (my_value VARCHAR) WITH (
  VALUE_FORMAT = 'JSON',
  KAFKA_TOPIC = 'my_stream_topic', 
  PARTITIONS=1, 
  REPLICAS=3);
  ```
5. `SELECT * FROM my_stream EMIT CHANGES;`

**Expected behavior**
Expected the query `SELECT * FROM my_stream EMIT CHANGES;` to start running.

**Actual behaviour**
The ksqlDB-cli outputs this error: `Error parsing host address http://<docker-compose_ksql-server_hostname>:8088/. Expected format host:port.`

Stack trace in ksqldb-cli log: 
```
[2020-02-27 18:53:05,739] ERROR Error parsing host address http://1130da4e15fb:8088/. Expected format host:port.
org.apache.kafka.streams.KafkaStreams.parseHostInfo(KafkaStreams.java:818)
org.apache.kafka.streams.KafkaStreams.<init>(KafkaStreams.java:707)
org.apache.kafka.streams.KafkaStreams.<init>(KafkaStreams.java:584)
io.confluent.ksql.query.KafkaStreamsBuilderImpl.buildKafkaStreams(KafkaStreamsBuilderImpl.java:43)
io.confluent.ksql.query.QueryExecutor.buildTransientQuery(QueryExecutor.java:165)
io.confluent.ksql.engine.EngineExecutor.executeQuery(EngineExecutor.java:129)
io.confluent.ksql.engine.KsqlEngine.executeQuery(KsqlEngine.java:215)
io.confluent.ksql.rest.server.resources.streaming.StreamedQueryResource.handlePushQuery(StreamedQueryResource.java:269)
io.confluent.ksql.rest.server.resources.streaming.StreamedQueryResource.handleStatement(StreamedQueryResource.java:208)
io.confluent.ksql.rest.server.resources.streaming.StreamedQueryResource.streamQuery(StreamedQueryResource.java:161)
sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
java.lang.reflect.Method.invoke(Method.java:498)
org.glassfish.jersey.server.model.internal.ResourceMethodInvocationHandlerFactory.lambda$static$0(ResourceMethodInvocationHandlerFactory.java:52)
...omitted tens of glassfish and jetty lines...
```

**Additional context**
This used to work with v0.6.0.

There's a separate branch `local_kafka` where the querying works. So the issue seems to be related to the configuration necessary to use Confluent Cloud.
