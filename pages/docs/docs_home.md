---
layout: page
title: Getting Started
permalink: docs_home.html
sidebar: docs_sidebar
---

Simple Sourcing is an API for building event sourcing systems where Kafka is used as the primary data store and System of record.

Simple Sourcing has specifically kept its footprint small and looks to solve a very focused problem. It is not meant to
be a full blown digital platform, for this there a a number of existing platforms that meet those needs well. Instead
the focus of this project is to provide the core primitives to build a CQRS system that leverages an event journal as
its primary data store (commonly referred to as Event Sourcing). We have decided to focus most of our effort on the
Command processing and Event generation side of the CQRS system with the view that the Kafka Streams technology provides a very
rich and expressive API for reading and transforming event streams and generating view projections from them. We may investigate some options to providing a
lightweight read side API in the future but for now that is out of scope of the core project.

Our key focus has been simplicity and ease of use. We have still not fully realised that goal and anticipate a number of
improvements over the next couple of months.

This documentation assumes basic knowledge of [Event Sourcing and CQRS](overview/event_sourcing.md) principles and terminology. If you're new
to either of these, there's some great introductory articles and talks listed in the *Background reading*
section below to get you started.


Simple Sourcing provides the following key benefits

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


## Getting Started

If you want to jump in straight away and see an example system running locally see the [Quick Start](quickstart.md)

For more information on the design of the system see the [Design Overview](overview/design.md)

## Project Source

Core repo: [https://github.com/simplesourcing/simplesource](https://github.com/simplesourcing/simplesource)

Examples: [https://github.com/simplesourcing/simplesource-examples](https://github.com/simplesourcing/simplesource-examples)


## Background reading

   * [Event Sourcing and CQRS](overview/event_sourcing.md) - a brief description of these concepts
{% include_relative overview/further_reading.md %}
