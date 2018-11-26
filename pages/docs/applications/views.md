---
layout: page
title: Materialized Views
permalink: views.html
---

Creating read views involves consuming one or more of the projection topics and writing them to a database suitable for querying. 

This is out of scope for Simple Sourcing, and it is up to the client to decide how to do this. 

The recommended option is to use [Kafka Streams](https://kafka.apache.org/documentation/streams/) directly to do the desired transformation and aggregation
 either from the event stream or the aggregate topic to new projection topics. 
 Then use [Kafka Connect](https://www.confluent.io/product/connectors/) to stream the output of
these topics to a database or search/query engine of your choice.

In the example app (`examples/auction`) we have provided an example of writing the projection topics to [MongoDB](https://www.mongodb.com/)
using the [Kafka Connect MongoDB sink](https://www.confluent.io/connector/kafka-connect-mongodb-sink/)

Please take a look at the example for more details.

![Alt text](../../../images/event-sourcing-projections.png?raw=true "Projections")
