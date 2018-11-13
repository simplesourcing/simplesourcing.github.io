---
layout: page
title: Aggregate Sets
permalink: aggregate_sets.html
---

The entry point into a Simple Sourcing application is the `AggregateSetBuilder` DSL. With this you can:
* Add aggregates, and their associated command and event handlers
* Set the Kafka for configuration to specifiy how to inteact with the Kafka cluster
* Call the `build` method which generates a `CommandAPISet` instance

For example:

```java
public static void main(final String[] args) {
    // start the application
    final CommandAPISet aggregateSet = new AggregateSetBuilder()
        .withKafkaConfig(builder ->
            builder
                .withKafkaApplicationId("userMappedAvroApp1")
                .withKafkaBootstrap("localhost:9092")
                .withApplicationServer("localhost:1234")
                .build())
        .addAggregate(userAggregateSpec)
        .addAggregate(accountAggregateSpec)
        .addAggregate(auctionAggregateSpec)
        .build();

    final CommandAPI<UserKey, UserCommand> api =
        aggregateSet.getCommandAPI("user");
    ...    
}
```

The [command API](command_api.html) can then be used to publish commands.