# Docker Compose commands for starting each DB service

You may update these docker compose files as per your requirement, but **do not** checkin your local changes unless needed by everyone else. 

Note: These containers are meant for local testing only.   

https://github.com/sambos/HowTo/wiki/Docker-Containers


* Change to each service to start
  * e.g.: CD ./mongo

* To Start container
  * docker-compose up -d
  
* To Stop container
  * docker-compose down

### Postgres management console
* http://localhost:5050
* Test your connection:
  * database name: testdb
  * username/password : postgres/postgres
  * hostname : postgresdb (pg container name)

### Rabbit MQ management console
* http://localhost:15672/
  * username/password : guest/guest


### Cassandra management console
* http://localhost:9091/
* select "Working with CQL"
* Create New Connection & Test
  * enter Host Name : cassandradb //as your datastax server container name
  * port : 9042



