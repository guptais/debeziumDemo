## References
`https://github.com/debezium/debezium-examples/tree/master/tutorial`

## Reference of Kafka image used in debezium

https://hub.docker.com/r/debezium/kafka

## Start Zookeeper
```
docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:0.9
```

## Start Kafka
```
docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:0.9
```

## Start a MySql database with a preconfigured database inventory
```
docker run -it --rm --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:0.9
```

## Start a MySQL command line client
```
docker run -it --rm --name mysqlterm --link mysql --rm mysql:5.7 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
```
 once you get the prompt. Try this
```
mysql> use inventory;

mysql> show tables;

mysql> SELECT * FROM customers;
```

## Start Kafka Connect

```
docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka --link mysql:mysql debezium/connect:0.9
```

## USing the Kafka Connect REST API

```
curl -H "Accept:application/json" localhost:8083/
```

## To list the connectors
```
curl -H "Accept:application/json" localhost:8083/connectors/
```

At this point we are running the Debezium services, a MySQL database server with a sample inventory database, and the MySQL command line client that is connected to our database. 

## Set up the connector for the inventory database 

```
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.whitelist": "inventory", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "dbhistory.inventory" } }'
```

you might need to escape the double quotes if required.

```
curl -H "Accept:application/json" localhost:8083/connectors/
```

should return
```
["inventory-connector"]
```

To see the running connector tasks which are responsible for the doing the work

```
curl -i -X GET -H "Accept:application/json" localhost:8083/connectors/inventory-connector
```


Let’s look at all of the data change events in the `dbserver1.inventory.customers` topic. We’ll use the debezium/kafka Docker image to start a new container that runs one of Kafka’s utilities to watch the topic from the beginning of the topic:
This is a like a watcher and keeps track all the events that occur 

```
docker run -it --name watcher --rm --link zookeeper:zookeeper --link kafka:kafka debezium/kafka:0.9 watch-topic -a -k dbserver1.inventory.customers
```


