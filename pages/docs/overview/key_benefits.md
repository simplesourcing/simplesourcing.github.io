---
layout: page
title: Key Benefits
permalink: key_benefits.html
sidebar: docs_sidebar
---

Simple Sourcing provides the following key benefits:

 * **Developer productivity** - Building a scalable resilient event sourcing system is a considerable amount of work.
   Spend your development time modeling your business domain, building up useful projections of your data rather than
   stressing about the mechanisms of setting up an event sourcing system.
 * **Ensures consistency of event store** - Simple Sourcing uses optimistic locking to guarantee that only commands
   that have seen the very latest event are accepted. Additionally for a given aggregate the command handler is guaranteed to process
   commands one at a time updating the event store in an atomic operation.
 * **Exactly-once semantics** - We use the exactly-once delivery guarantees provided by Kafka to ensure each command
 is processed once, and each event appears in order exactly once in the event store.
 * **Horizontal scalability** - You can run more than one instance of your application and behind the scenes,
   the command handler and projectors will transparently share the work amongst all the running instances.
 * **Manages low level interaction with Kafka** - Simple Sourcing can manage all reads and writes to Kafka,
   creating topics with the appropriate configuration and managing all state used by the command handler, event store and projectors.
 * **Supports evolution of business domain objects** - When using Avro for serialization we use
   the [Confluent Schema Registry](https://docs.confluent.io/current/schema-registry/docs/index.html)
   to provide support for schema evolution for your events, commands and projections.
 * **Support for many types of aggregates** - All Kafka topics, schemas and state stores are specific to each aggregation
   type so they can safely co-exist on the same Kafka cluster and can even be managed from the same Java application.
 * **Low level access** - The event store and projections are persisted to Kafka topics. As such they
   can be read by any process with access to Kafka, including [Kafka Connect sink connectors](https://www.confluent.io/product/connectors/).
   It is perfectly feasible, for instance, to generate projections off the event store using [KSQL](https://www.confluent.io/product/ksql/),
   pushing the output into ElasticSearch using Kafka Connect and build your own Query API based on that datastore.
