### Start the topology as defined in http://debezium.io/docs/tutorial/

```
export DEBEZIUM_VERSION=0.9
docker-compose -f docker-compose-mysql.yaml up
```
### Start MySQL connector

```
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-mysql.json
```

### Consume messages from a Debezium topic
```
docker-compose -f docker-compose-mysql.yaml exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbserver1.inventory.customers
```

### Modify records in the database via MySQL client
```
docker-compose -f docker-compose-mysql.yaml exec mysql bash -c 'mysql -u $MYSQL_USER -p$MYSQL_PASSWORD inventory'
```

### Shut down the cluster
```
docker-compose -f docker-compose-mysql.yaml down
```
