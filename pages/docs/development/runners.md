---
layout: page
title: Running the application
permalink: runners.html
---

The entry point into a Simple Sourcing application is the `AggregateSetBuilder` DSL. With this you can:
* Add aggregates, and their associated command and event handlers
* Set the Kafka for configuration to specifiy how to inteact with the Kafka cluster
* Call the `build` method which generates a `CommandAPISet` instance

For example:

```java
final CommandAPISet aggregateSet = new AggregateSetBuilder()
    .withKafkaConfig(builder ->
        builder
            .withKafkaApplicationId("userMappedAvroApp1")
            .withKafkaBootstrap("localhost:9092")
            .withApplicationServer("localhost:1234")
            .build())
    .addAggregate(UserAggregate.createSpec(
        "user",
            avroAggregateSerdes,
        new PrefixResourceNamingStrategy("user_avro_"),
        (k) -> Optional.empty()
    ))
    .build();

final CommandAPI<UserKey, UserCommand> api =
    aggregateSet.getCommandAPI("user");
```

A `CommandAPISet` provides a `CommandAPI` for each of the aggregates added. The `CommandAPI` has methods to publish commands,
and wait for the result to these commands.

Note that result of the command only will only be whether the command succeeded or failed:
* If successful, the command sequence number
* If failure, the reason(s) for failure

You'll also want to provide a mechanism to generate commands to push into the system using the provided
`CommandAPI` interface.  In a production system commands would typically be generated off the back of user actions,
or off of an existing data feed.

A Simple Sourcing application will often be deployed as a service that exposes endpoints for the commands that will be executed.
These endpoints interact with the `CommandAPI` interface. 
