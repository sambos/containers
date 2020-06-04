## kafka worked examples
https://github.com/sambos/kafka-samples

### book
https://www.confluent.io/wp-content/uploads/confluent-kafka-definitive-guide-complete.pdf

## Kafka Container setup
### kafka confluent workshop 
https://github.com/confluentinc/kafka-workshop


#### Kafka Docker Connectivity
https://github.com/wurstmeister/kafka-docker/wiki/Connectivity

#### kafka docker info
https://github.com/wurstmeister/kafka-docker
https://docs.confluent.io/3.2.1/installation/docker/docs/quickstart.html#getting-started-with-docker-compose
https://github.com/datastax/kafka-examples/tree/master/connectors/jdbc-source-connector


#### fast-data-dev Docker Container
##### All in One Docker container for Kafka - for developers

https://github.com/lensesio/fast-data-dev

```sh
docker run --rm -p 2181:2181 -p 3030:3030 -p 8081-8083:8081-8083 \
      -p 9581-9585:9581-9585 -p 9092:9092 -e ADV_HOST=127.0.0.1 \
      landoop/fast-data-dev:latest
```

```
TODO: Add Docker compose file
```


## Useful commands:

```console
# check topics
docker-compose exec kafka1 bash -c 'kafka-topics --zookeeper zookeeper:2181 --list'
Default Topics:

# command to check if compression is working fine..
- docker-compose exec kafka1 bash (topic name = new-employees)
- kafka-run-class kafka.tools.DumpLogSegments --files /var/lib/kafka/data/new-employees-0/00000000000000000000.log --print-data-log | grep compresscodec


you should see something like :
baseOffset: 0 lastOffset: 0 count: 1 ... compresscodec: NONE ...
baseOffset: 1 lastOffset: 1 count: 1 ... compresscodec: GZIP ...
baseOffset: 2 lastOffset: 2 count: 1 ... compresscodec: SNAPPY ...
baseOffset: 3 lastOffset: 3 count: 1 ... compresscodec: LZ4 ...
```
#### kafkacat commands:

you may use docker-compose-full.yml to run with these commands

```console, sh
# describe a topic
docker-compose exec kafkacat kafkacat -b kafka1:9092 -L -t new-employees
docker-compose exec kafka1 kafka-topics --zookeeper zookeeper:2181 --describe --topic new-employees

# get earliest offset
docker-compose exec kafka1 kafka-run-class kafka.tools.GetOffsetShell --broker-list kafka1:9092 --topic new-employees --time -2

# latest offset
docker-compose exec kafka1 kafka-run-class kafka.tools.GetOffsetShell --broker-list kafka1:9092 --topic new-employees --time -1 --offsets 1

# view details of consumer group
docker-compose exec kafka1 kafka-consumer-groups --bootstrap-server kafka1:9092 --describe --group demo-group

# kafka offset (run on kafka container) - (same as above command)
kafka-run-class kafka.admin.ConsumerGroupCommand --bootstrap-server kafka1:9092 --group consumer-1 --describe

TOPIC                 PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
sample.producer.topic 0          26              26              0 


# Decode __consumer_offsets topic 
docker-compose exec connect bash -c 'kafka-console-consumer --bootstrap-server kafka1:9092 --topic __consumer_offsets --from-beginning --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter"'

or run command from connect/kafka node

kafka-console-consumer --bootstrap-server kafka1:9092 --topic __consumer_offsets --from-beginning --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter"

format:
[groupId,topicName,partitionNumber]::[OffsetMetadata[OffsetNumber,NO_METADATA],CommitTime 1520613132835,ExpirationTime 1520699532835]

reponse:
[demo-group,postgres-movies,0]::OffsetAndMetadata(offset=30, leaderEpoch=Optional.empty, metadata=, commitTimestamp=1591295989063, expireTimestamp=None)
[test-1,new-employees,0]::OffsetAndMetadata(offset=13, leaderEpoch=Optional.empty, metadata=, commitTimestamp=1591296368280, expireTimestamp=None)
[test-1,new-employees,0]::OffsetAndMetadata(offset=13, leaderEpoch=Optional.empty, metadata=, commitTimestamp=1591296369725, expireTimestamp=None)

# delete topic
docker-compose exec kafka1 kafka-topics --zookeeper zookeeper:2181 --delete --topic movies-raw

#create topic
docker exec connect kafka-topics --create --topic quickstart-data --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181

#create source connector
see examples for file and jdbc connector below

docker-compose exec <kafkacat-container-name> <kafkacat-commands>

# display last 10 messages 
kafkacat -b kafka1:9092 -t <topic-name> -p 0 -o -10 -e

#consume messages from topic
kafkacat -b kafka1:9092 -C -t <topic-name>

# produce messages from file
kafkacat -b kafka1:9092 -P -c 2 -t <topic-name> -l /data/movies-json.js

#meta-data
kafkacat -b kafka1:9092 -L -t <topic_name>

              -               -               -

```
## Kafka Connector examples

#### JDBC connector example

```sh
# setup postgres db with table

# create jdbc source connector for postgresdb
docker-compose exec connect bash -c 'curl -i -X POST -H "Accept:application/json" \
        -H "Content-Type:application/json" -d @/connect/postgres-source.json http://localhost:8083/connectors'
        
or inline:

docker-compose exec connect curl -s -X POST -H "Content-Type: application/json" --data '{"name":"jdbc_source_postgres_movies","config":{"connector.class":"io.confluent.connect.jdbc.JdbcSourceConnector","connection.url":"jdbc:postgresql://database:5432/WORKSHOP?user=postgres&password=postgres","table.whitelist":"movies","mode":"incrementing","incrementing.column.name":"id","validate.non.null":"false","_comment":"The Kafka topic will be made up of this prefix, plus the table name  ","topic.prefix":"postgres-"}}' http://connect:8083/connectors

# check connectors are created 
docker-compose exec connect curl -s -X GET http://connect:8083/connect

response:
["jdbc_source_postgres_movies"]

#check topic is created
docker-compose exec kafka1 bash -c 'kafka-topics --zookeeper zookeeper:2181 --list'

response:
__confluent.support.metrics
__consumer_offsets
_confluent-ksql-kafka_workshop_command_topic
_schemas
docker-connect-configs
docker-connect-offsets
docker-connect-status
postgres-movies

# check that movies avro schema got registered in schemaregistry
docker-compose exec kafkacat curl -s -X GEThttp://schemaregistry:8081/subjects/postgres-movies-value/versions/latest

```

#### FileStream connector example

```sh
# generate some data on connect container
docker-compose exec connect bash -c 'seq 1000 > /tmp/quickstart/file/input.txt'

#create topic
docker exec connect kafka-topics --create --topic quickstart-data --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181

#copy file-source.json, file-sink.json from local to connect container
docker cp ./file-*.json <container-name>:/connect

#create file source/sink connectors
docker-compose exec connect bash -c 'curl -i -X POST -H "Accept:application/json" \
        -H "Content-Type:application/json" -d @/connect/file-source.json http://localhost:8083/connectors'

docker-compose exec connect bash -c 'curl -i -X POST -H "Accept:application/json" \
        -H "Content-Type:application/json" -d @/connect/file-sink.json http://localhost:8083/connectors'
        
or inline:

docker exec connect curl -s -X POST -H "Content-Type: application/json" --data '{"name": "quickstart-file-source", "config": {"connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector", "tasks.max":"1", "topic":"quickstart-data", "file": "/tmp/quickstart/file/input.txt"}}' http://connect:8083/connectors

```


## Schema Registry
##### Multiple schemas, single topic
https://karengryg.io/2018/08/18/multi-schemas-in-one-kafka-topic/


