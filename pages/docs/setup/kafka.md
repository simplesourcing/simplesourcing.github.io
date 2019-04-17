---
layout: page
title: Kafka Setup
permalink: /simple_sourcing_kafka.html
---

## Local Kafka development setup

Although you can install and run Kafka locally we suggest using pre-built
[Docker](https://www.docker.com/) images as a much more reliable and productive way to develop.

The simplest way to start is to use one of the `docker-compose.yml` files provided in the [example project](https://github.com/simplesourcing/simplesource-examples/) as a basis.

A typical Kafka Docker Compose stack consists of the following container instances:
* A Zookeeper instance
* One or more instances of a Kafka broker
* An instance of the [Confluent Schema Registry](https://www.confluent.io/confluent-schema-registry/) if you are using Avro to serialize the messages on Kafka

In addtion you may also want to include a [Kafka Connect]() instance if your development includes streaming to a projection database or search eangine. 

Once you have installed Docker, create a `docker-compose.yml` file
to define the docker containers that you want to run. 

To run these containers:

```bash
$ docker-compose up
```

Press Ctrl-C to stop the services. To stop and remove them:

```bash
$ docker-compose down
```

See the [Docker Compose docs](https://docs.docker.com/compose) for more information.

## Examples

Example `docker-compose.yml` files are given below:

* [Kafka, Zookeeper and Schema Registry](https://github.com/simplesourcing/simplesource-examples/blob/master/examples/user/docker-compose.yml)
* [Kafka, Kafka Connect and Mongo](https://github.com/simplesourcing/simplesource-examples/blob/master/examples/auction/docker-compose.yml)
