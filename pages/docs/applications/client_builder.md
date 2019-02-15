---
layout: page
title: Building the Client
permalink: client_builder.html
---

The entry point into a Simple Sourcing client is the `EventSourcedClient` DSL. With this you can:
* Set the Kafka for configuration to specify how to interact with the Kafka cluster
* Add a set of commands for one or more aggregates
* Call the `build` method which returns a `CommandAPISet`. A `CommandAPISet` provides access to a  
[`CommandAPI`](command_api.html) for each aggregate

For example:

```java
public static void main(final String[] args) {
    final AggregateSerdes<~> avroAggregateSerdes = ...;

    final CommandAPISet commandApiSet =
            new EventSourcedClient()
                    .<UserKey, UserCommand>addCommands(builder -> builder
                            .withClientId("client-1")
                            .withName("user")
                            .withSerdes(avroCommandSerdes)
                            .withResourceNamingStrategy(namingStrategy)
                            .build())
                    .withKafkaConfig(builder -> builder
                            .withKafkaBootstrap(bootstrapServers)
                            .build())
                    .build();

    return commandApiSet.getCommandAPI("user");
}
```

The client can now publish commands using the [command API](command_api.html).

The `addCommands` method on the `EventSourcedClient` works similarly to the aggregate builder. However the client only needs to know about the commands, and provide serdes  for serialization. It does not need to know about the command and event handlers.

The Client ID, specified in `withClientId`, should be different for each client instance, or at least for each host machine the client is running on. If you have many clients, and don't create separate client IDs for each client, it will result in unncessary network traffic.

## Combined server and client

Although it is generally encouraged to separate the client application from the streaming server application it is possible to create a client from an `EventSourcedApp` directly. 
Calling the `getCommandAPISet` on the `EventSourcedApp` instance returns a `CommandAPISet`.
Note that this method must be called after starting the application with the `start` method.

