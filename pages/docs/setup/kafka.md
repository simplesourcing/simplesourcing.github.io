---
layout: page
title: Kafka Setup
permalink: kafka.html
---

## Local Kafka setup for development

Although you can install and run Kafka locally we suggest using pre-built
docker images as a much more reliable and productive way to develop.

Once you have installed [Docker](https://www.docker.com/), create a `docker-compose.yml` file
to define the docker containers that you want to run. Start with the
following:

```
version: '3'
services:
  zookeeper:
    image: 'confluentinc/cp-zookeeper:5.0.0'
    restart: always
    hostname: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOO_MY_ID: 1
      ZOO_PORT: 2181
      ZOO_SERVERS: server.1=zookeeper:2888:3888
      COMPONENT: zookeeper
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  broker:
    image: 'confluentinc/cp-kafka:5.0.0'
    hostname: broker
    stop_grace_period: 120s
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "8500:8500"
    environment:
      COMPONENT: kafka
      KAFKA_BROKER_ID: 1001
      KAFKA_RESERVED_BROKER_MAX_ID: 10000
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://broker:9092'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
      KAFKA_JMX_OPTS: '-Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=broker -Djava.net.preferIPv4Stack=true -Dcom.sun.management.jmxremote.rmi.port=8500'
```

This defines two containers one running zookeeper and one running a Kafka
broker. This is the minimum required for Simple Sourcing to run.

To run these containers:

```
$ docker-compose up
```

Press Ctrl-C to stop the services. To stop and remove them:

```
$ docker-compose down
```

### Schema Registry

If you are using Avro to serialize the messages on Kafka, you will also
want to setup a [Schema Registry](https://www.confluent.io/confluent-schema-registry/).

Add the following to your `docker-compose.yml`:

```
  schema_registry:
    image: 'confluentinc/cp-schema-registry:5.0.0'
    hostname: schema_registry
    depends_on:
      - zookeeper
      - broker
    ports:
      - "8081:8081"
    environment:
      TZ: Australia/Sydney
      SCHEMA_REGISTRY_HOST_NAME: schema_registry
      SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL: 'zookeeper:2181'
      SCHEMA_REGISTRY_KAFKASTORE_TIMEOUT_MS: 10000
      SCHEMA_REGISTRY_LOG4J_ROOT_LOGLEVEL: 'INFO'
```

