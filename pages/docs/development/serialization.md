---
layout: page
title: Serialization
permalink: serialization.html
---

Any Kafka streams operation that writes to disk requires a [Serde](https://kafka.apache.org/11/javadoc/org/apache/kafka/common/serialization/Serde.html) 
for each type that needs to be serialized.

For each aggregate, a set of Serdes is required, and it the responsibility of the client to create these, 
by providing an implementation of the `AggregateSerdes` interface.

Simple Sourcing provides some utilities to assist with this for both Avro and JSON formats.
We recommend the Avro implementation due to its inbuilt support for schema evolution via the Confluent Schema Registry.

Unfortunately there is still some unavoidable boiler plate code required to set up the Serdes.

The user example project includes several different configurations you can use as a template when setting up
your own project.

## Avro serialization recipe

These are the steps required to create a single Avro Serde, as a summary:

1. Create your domain class definition. This is the type you work with in your Simple Sourcing or KStream code.
1. Create the Avro schema file (`.avdl` files) to represent the Avro schema for the class.
1. Run the Maven goal `generate-sources` to generate Java files for the `.avdl`. Note that these generated classes will be subclasses of Avro `GenericObject`.
  
   Ths step happens automatically when you run `mvn install`.
1. Create a `GenericMapper` that maps your domain object to the auto-generated Avro Java object. 
1. Create an instance of `Serde<GenericRecord>`. The underlying implementation for this Serde is provided by Confluent. We can create it with `AvroGenericUtils.genericAvroSerde`, passing in the necessary
configuration for the Schema Registry.
1. Use the `GenericSerde.of` method to create the desired Serde for your domain class. 

    It works by combining the `Serde<GenericRecord>` with the `GenericMapper`. 
    It uses the `Serde<GenericRecord>` to convert between `byte[]` and `GenericRecord`, 
    and your `GenericMapper` to convert between `GenericRecord` and your domain class.
1. Repeat the above process for the other domain entities that are required for the aggregate.
1. Create a `AggregateSerdes` instance with the Serdes for all the required types for all the Simple Sourcing processing steps.

## Create domain classes in Avro

When using Avro as your serialization format, you need to generate Avro classes for all your domain objects.
We recommend using the Avro IDL to define your objects as it provides a more concise readable format than the `.avsc` JSON format.
To get started, check out the [Avro reference documentation](https://avro.apache.org/docs/1.8.1/idl.html) and
take a look at [user.avdl](/examples/user/src/main/avro/user.avdl) from the included example projects.

You can have as many types as you like for events and commands. You don't need to  using a union type to represent your
individual events and commands. We're using the recent extension to the schema registry for
[multiple schemas registered under the same topic](https://www.confluent.io/blog/put-several-event-types-kafka-topic/)
to support this pattern.

```
record PostCreated {
  string title;
  string body;
  long authorUserId;
}

record PostPublished {}

record PostLiked {
  long userId;
}

record PostUnliked {
  long userId;
}

record PostDeleted {}
```

Add your `.avdl` file to the `src/main/avro` directory in your project and configure the Avro plugin for whichever
build tool you use (Maven, Gradle and sbt all have plugins) to generate code for your domain classes.

You can also explicitly run `mvn generate-sources` to generate the Java serialization classes.

## Schema Naming Strategy

The schema for every Avro serialized type is saved in the Confluent Schema registry. The Schema Registry supports two naming strategies:
* Topic Record name Strategy - This creates a schema from the topic name and the fully qualified name of the Java class generated from the `.avdl` file.
This schema naming strategy must be used for Events and Commands, where different types of events and commands appear in the same topic.
* Topic Name Strategy - This schema strategy uses only the topic name for the schema name. 
It will not work for commands and events, but is fine for aggregates and other `KTable` based projections, which are typically monomorphic. 
It is also the only schema naming stategy supported by some of the plugins for Kafka Connect.

This behaviour is controlled by the `SchemaNameStrategy` in the `AvroGenericUtils.genericAvroSerde` helper method.

## Json Serialization

The `/examples/user` example app demonstrates how to create Serdes for your domain objects using Json serialization,
using the [Gson](https://github.com/google/gson) Json library.

Please take a look at the example app for more details. 
