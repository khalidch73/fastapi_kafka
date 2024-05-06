# FastAPI Event Driven Development With Kafka, Zookeeper and Docker Compose

In this directory we will set up a single-node Kafka and Zookeeper environment using Docker Compose. We will then produce and consume test messages using the Kafka console producer and consumer.

```bash
mkdir fastapi_kafka
cd fastapi_kafka
```

## Create a Docker Compose file

```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.0.1
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_SYNC_LIMIT: 2
    volumes:
      - ./zk-data:/var/lib/zookeeper/data
      - ./zk-log:/var/lib/zookeeper/log

  kafka:
    image: confluentinc/cp-kafka:7.0.1
    hostname: kafka
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9999
    volumes:
      - ./kafka-data:/var/lib/kafka/data

networks:
  default:
    name: kafka-network
```

## Start the Kafka and Zookeeper containers

```bash
docker-compose up -d
```

## Check the containers

```bash
docker compose ps
```

## Create test.events topic

### Enter the Kafka container’s shell

```bash
docker exec -it kafka bash
```

### Use the kafka-topics command

```bash
$ kafka-topics --create --topic test.events --bootstrap-server kafka:29092 --partitions 4 --replication-factor 1
```

### Lets check the newly created topic

```bash
kafka-topics --describe --topic test.events --bootstrap-server kafka:29092
```

### Produce a test message

```bash
kafka-console-producer --bootstrap-server kafka:29092 --topic test.events
```

### Consume the test messages

```bash
kafka-console-consumer --bootstrap-server kafka:29092 --topic test.events --from-beginning
```
