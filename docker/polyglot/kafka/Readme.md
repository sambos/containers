
https://github.com/wurstmeister/kafka-docker

https://github.com/datastax/kafka-examples/tree/master/connectors/jdbc-source-connector

#### Kafka Docker Connectivity
https://github.com/wurstmeister/kafka-docker/wiki/Connectivity

#### Kafka Confluent with Docker QuickStart
https://docs.confluent.io/3.2.1/installation/docker/docs/quickstart.html#getting-started-with-docker-compose


#### kafka confluent workshop 
https://github.com/confluentinc/kafka-workshop


#### Useful commands:

```sh
# check topics
docker-compose exec kafka1 bash -c 'kafka-topics --zookeeper zookeeper:2181 --list'

# display last 10 messages 
docker-compose exec worker kafkacat -b kafka1:9092 -t <topic-name> -p 0 -o -10 -e

# delete topic
docker-compose exec kafka1 kafka-topics --zookeeper zookeeper:2181 --delete --topic movies-raw

#consume messages from topic
docker-compose exec worker kafkacat -b kafka1:9092 -C -t movies-raw

# produce messages from file
docker-compose exec worker kafkacat -b kafka1:9092 -P -c 2 -t movies-raw -l /data/movies-json.js



```
