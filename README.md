# MongoDB Change Event Stream via Debezium Kafka Connector with a .NET 5 Client

Read [the related article](https://vkontech.com/mongodb-change-data-capture-via-debezium-kafka-connector-with-a-net-5-client/) for a detailed overview.

## Required Software ##
- Docker
- Windows 10
- MSVS 2019 or Rider

## Running the Example ##

1. Install Zookeeper

`docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:1.6`

2. Install Kafka

`docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:1.6`

3. Install MongoDB

`docker run -p 27017:27017 --name mongo1 mongo mongod --replSet my-mongo-set`

4. Configure Mongo as a Replica Set

`docker exec -it mongo1 mongo`
`config = {"_id":"my-mongo-set","members":[{"_id":0,"host":"mongo1:27017"}]}`
`rs.initiate(config)`

6. Install Kafka Connect

`docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka --link mongo1:mongo1 debezium/connect:1.6`

7. Create the "users-connector"

`curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{"name":"users-connector","config":{"connector.class":"io.debezium.connector.mongodb.MongoDbConnector","mongodb.hosts":"my-mongo-set/mongo1:27017","mongodb.name":"mongo1","collection.include.list": "testdb.users"}}'`

8. Run the simple .NET consumer from this repo and create a new Mongo entity:

`db = (new Mongo('localhost:27017')).getDB('testdb')`
`db.users.insert({name : 'John'})`

9. Make sure the inserted entity gets printed to the Console
