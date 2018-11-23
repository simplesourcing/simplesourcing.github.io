---
layout: page
title: Starting the Application
permalink: application_builder.html
---

The entry point into a Simple Sourcing application is the `EventSourcedApp` DSL. With this you can:
* Set the Kafka for configuration to specifiy how to inteact with the Kafka cluster
* Add one or more aggregates, and their associated command and event handlers
* Call the `start` method which starts the application

For example:

```java
public static void main(final String[] args) {
    final AggregateSerdes<~> avroAggregateSerdes = ...;

    new EventSourcedApp()
        .withKafkaConfig(configBuilder -> configBuilder
                .withKafkaApplicationId("userMappedAvroApp1")
                .withKafkaBootstrap("localhost:9092")
                .build())
        .<~>addAggregate(aggregateBuilder -> aggregateBuilder
                .withName("user")
                .withSerdes(avroAggregateSerdes)
                .withResourceNamingStrategy(new PrefixResourceNamingStrategy("user_avro_"))
                .withInitialValue((k) -> Optional.empty())
                .withAggregator(UserEvent.getAggregator())
                .withCommandHandler(UserCommand.getCommandHandler()))
        .<~>addAggregate(aggregateBuilder -> ... /* another aggregate */ )
        .start();
}
```

The application is now ready to receive commands published using the [command API](command_api.html).
