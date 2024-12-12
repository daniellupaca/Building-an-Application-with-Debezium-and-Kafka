# Building an Application with Debezium and Kafka

### Course: TÓPICOS DE BASE DE DATOS A
### Teacher: Mag. Patrick Cuadros Quiroga

### Introduction

In modern application development, real-time synchronization and updates of data are constant challenges. Change Data Capture (CDC) tools like Debezium allow monitoring and transmitting changes made to a database to consuming systems, enabling more scalable and reactive architectures. This article presents how to build an application using Debezium and Kafka to capture and process changes in a relational database.

### Why Choose Debezium and Kafka?

1. **Debezium**: Provides an efficient and reliable way to capture changes in databases like MySQL, PostgreSQL, SQL Server, among others. It is easy to configure and allows capturing insert, update, and delete events.
2. **Kafka**: Acts as the message broker that transports captured events to consumers, ensuring scalability and efficient real-time data handling.

### Methodology

#### 1. Environment Setup

- Use Docker to deploy containers for Kafka, Zookeeper, Debezium, and a MySQL database.
- Configure Debezium connectors to monitor a database.

#### 2. Application Development

- Configure a consumer in Python using the `confluent-kafka` library to process change events.
- Design application logic to react to the processed events.

#### 3. Testing and Deployment

- Test the application with CRUD operations on the database and verify the real-time processing of captured events.
- Deploy the application using a local or cloud environment.

### Implementation

#### 1. Environment Setup with Docker

Create a `docker-compose.yml` file to deploy Kafka, Zookeeper, MySQL, and Debezium:

```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:latest
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: test_db

  debezium:
    image: debezium/connect:latest
    ports:
      - "8083:8083"
    environment:
      BOOTSTRAP_SERVERS: kafka:9092
      GROUP_ID: debezium-connect
```
# 2. Configure Debezium Connector
# Send the connector configuration to the Debezium endpoint
```yaml
curl -X POST -H "Content-Type: application/json" \
     --data '{
         "name": "mysql-connector",
         "config": {
             "connector.class": "io.debezium.connector.mysql.MySqlConnector",
             "tasks.max": "1",
             "database.hostname": "mysql",
             "database.port": "3306",
             "database.user": "root",
             "database.password": "root",
             "database.server.id": "1",
             "database.server.name": "dbserver1",
             "database.include.list": "test_db",
             "database.history.kafka.bootstrap.servers": "kafka:9092",
             "database.history.kafka.topic": "schema-changes.test_db"
         }
     }' \
     http://localhost:8083/connectors
```
# 3. Develop the Consumer in Python
# Create a consumer using the confluent-kafka library
```yaml
from confluent_kafka import Consumer, KafkaException

consumer_config = {
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'python-consumer',
    'auto.offset.reset': 'earliest'
}

consumer = Consumer(consumer_config)
consumer.subscribe(['dbserver1.test_db.test_table'])

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        if msg.error():
            raise KafkaException(msg.error())
        print(f"Received message: {msg.value().decode('utf-8')}")
finally:
    consumer.close()
```

## Results

The application captures real-time change events in the MySQL database, processes them using Kafka and Debezium, and consumes them via the Python client. Tests showed that operations like inserts, updates, and deletions in the database are instantly reflected in the processed messages.

## Conclusion

Using tools like Debezium and Kafka simplifies the implementation of reactive and scalable data architectures. This project demonstrates how to efficiently capture and process database changes, enabling real-time applications.

## GitHub Repository

The source code for the application is available at the following GitHub link:  
[Project Repository](https://github.com/daniellupaca/Building-an-Application-with-Debezium-and-Kafka)

## References

1. Debezium Documentation. [https://debezium.io/](https://debezium.io/)  
2. Kafka Documentation. [https://kafka.apache.org/](https://kafka.apache.org/)  
3. Docker Documentation. [https://docs.docker.com/](https://docs.docker.com/)
