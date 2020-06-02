
### kafka confluent workshop 
https://github.com/confluentinc/kafka-workshop


#### Kafka Docker Connectivity
https://github.com/wurstmeister/kafka-docker/wiki/Connectivity

#### kafka docker info
https://github.com/wurstmeister/kafka-docker
https://docs.confluent.io/3.2.1/installation/docker/docs/quickstart.html#getting-started-with-docker-compose
https://github.com/datastax/kafka-examples/tree/master/connectors/jdbc-source-connector



#### Useful commands:

```sh
# check topics
docker-compose exec kafka1 bash -c 'kafka-topics --zookeeper zookeeper:2181 --list'
Default Topics:

```
#### kafkacat commands:

you may use docker-compose-full.yml to run with these commands

```sh

# delete topic
docker-compose exec kafka1 kafka-topics --zookeeper zookeeper:2181 --delete --topic movies-raw

#create topic
docker exec connect kafka-topics --create --topic quickstart-data --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181

#create source connector


docker-compose exec <kafkacat-container-name> <kafkacat-commands>

# display last 10 messages 
kafkacat -b kafka1:9092 -t <topic-name> -p 0 -o -10 -e

#consume messages from topic
kafkacat -b kafka1:9092 -C -t <topic-name>

# produce messages from file
kafkacat -b kafka1:9092 -P -c 2 -t <topic-name> -l /data/movies-json.js

#meta-data
kafkacat -b kafka1:9092 -L -t <topic_name>

# kafka offset (run on kafka container)
kafka-run-class kafka.admin.ConsumerGroupCommand --group consumer-1 --bootstrap-server kafka1:9092 --describe

TOPIC                 PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
sample.producer.topic 0          26              26              0               -               -               -

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

#### Schema Registry
##### Multiple schemas, single topic
https://karengryg.io/2018/08/18/multi-schemas-in-one-kafka-topic/

