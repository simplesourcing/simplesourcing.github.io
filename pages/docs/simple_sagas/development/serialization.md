---
layout: page
title: Serialization
permalink: /simple_sagas_serialization.html
---

In all Kafka and Kafka Streams programs, 
many operation involves shuffling data on the network, or storing data in Kafka topics.
The data involved must be serializable and for this we need to provide 
[Serdes](https://kafka.apache.org/21/javadoc/org/apache/kafka/common/serialization/Serde.html).

Simple Sagas provides some help to reduce the amount of boilerplate code involved for creating Avro and Json serdes.

### Saga Action Command Type

Sagas consist of one free type, the type `A` in [`SagaApp<A>`](/apidocs-sagas/io/simplesource/saga/saga/app/SagaApp.html).

This is a common representation of an [action command](/simple_sagas_key_concepts.html#actions-and-commands), required by the action processor to execute the action.

Due to limitations of the Java type system, it is not possible (or at least not practical) to represent a saga in a serializable format in which each action command has its own specific type.

A compromise needed to be made, and this involves choosing a serializable type that can be conveniently converted to any of the specific types that the
[action processors](/simple_sagas_action_processors.html) are implemented in terms of.

### Avro

In Simple Sagas, [Avro](https://avro.apache.org/) is the recommended serialization format. 
For Avro serialization an appropriate choice for `A` 
is [`SpecificRecord`](https://avro.apache.org/docs/1.8.2/api/java/org/apache/avro/specific/SpecificRecord.html) 
or [`GenericRecord`](https://avro.apache.org/docs/1.8.2/api/java/org/apache/avro/generic/GenericRecord.html).

Helpers are provided to minimise the effort involves with working with these types. 

When using Avro serialization, Simple Sagas takes advantage of the Confluent Schema registry
to ensure that data conforms to a schema. At runtime, an instance of the Schema registry must be available.

For Java applications, it makes sense to define the domain types in your application as [AVDL](https://avro.apache.org/docs/1.8.2/idl.html) 
types, and use the Avro Maven plugin to generate Java classes for these types. These classes implement the 
[`SpecificRecord`](https://avro.apache.org/docs/1.8.2/api/java/org/apache/avro/specific/SpecificRecord.html) 
interface, are serializable, and can be used in Sagas easily:

The [`AvroSerdes.Specific`](/apidocs-sagas/io/simplesource/saga/serialization/avro/AvroSerdes.Specific.html) 
interface provides factory methods for Serdes for any type that implements [`SpecificRecord`](https://avro.apache.org/docs/1.8.2/api/java/org/apache/avro/specific/SpecificRecord.html).

If it is not possible or desired to represent the domain types with AVDL types, one can use the [`AvroSerdes`](/apidocs-sagas/io/simplesource/saga/serialization/avro/AvroSerdes.html) 
methods to create the required Serdes. In this case the user must provide an Serde implementation for the type `A`.

### Json
