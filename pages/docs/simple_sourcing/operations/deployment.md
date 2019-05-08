---
layout: page
title: Kafka Deployment
permalink: /simple_sourcing_deployment.html
---

## Deploying to Kafka

Simple Sourcing is a library that makes use of Kafka Streams. It does not need to be deployed separately - 
it runs in process as part of your application. 

The minimum viable Simple Sourcing deployment consists of the following:
* A Zookeeper instance
* 3 or more Kafka broker instances
* If using Avro serialization, an instance of the Confluent Schema registry

See the [auction example app](https://github.com/simplesourcing/simplesource-examples/tree/master/examples/auction) for an example docker configuration.

## Inspecting data in Kafka

It can be useful to see the data contained in the topics to understand the data structures used and to validate correctness.

To see a list of available topics
```
docker-compose exec broker kafka-topics --zookeeper zookeeper:2181 --list
```

To see what data is available in a given topic
```bash
docker-compose exec broker kafka-run-class kafka.tools.GetOffsetShell \
  --offsets 10 --broker-list broker:9092 \
  --topic simple-events
```

This outputs one line per topic partition in the following format

```
<TOPICNAME> : <PARTITION> : Highest offset in latest log file [ highest offset in rest of log files ] [ , earliest offset held of oldest log file ]
```

To see the contents of the event source topic from the beginning if using Avro
```bash
docker-compose exec schema_registry kafka-avro-console-consumer \
  --bootstrap-server broker:9092 \
  --property schema.registry.url=http://schema_registry:8081 \
  --property print.key=true \
  --from-beginning \
  --topic simple-events
```

If using text-based encodings like JSON you can use
```bash
docker-compose exec broker kafka-console-consumer \
  --bootstrap-server broker:9092 \
  --property print.key=true \
  --from-beginning \
  --topic simple-events
```

The [Simple Sourcing example repo](https://github.com/simplesourcing/simplesource-examples) also consists of [scripts](https://github.com/simplesourcing/simplesource-examples/tree/master/scripts) to perform these tasks on a running docker stack.

