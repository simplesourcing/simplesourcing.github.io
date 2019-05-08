---
layout: page
title: Saga Coordinator
permalink: /simple_sagas_saga_coordinator.html
---

The saga coordinator receives a saga request in the saga request topic. This request can be dispatched by any kafka or kafka streams application, 
or via the [Saga Client API](/simple_sagas_client_api.html).

The saga coordinator guides the saga through the execution process, delegating to the [/simple_sagas_action_processors.html](action processors) to execute the actions comprising the saga,
and publishes a response to the saga response topic when the action is complete.

The saga coordinator runs as a [Kafka Streams](https://kafka.apache.org/documentation/streams/) application. 
It can run in the same JVM process as the [action processor app](/simple_sagas_action_processors.html#creating-an-action-processor-app).

To run it on it's own, use the [`SagaApp`](/apidocs-sagas/io/simplesource/saga/saga/app/SagaApp.html) to create an executable with a main method like this:

```java
SagaApp.of(
         sagaSpec,
         actionSpec,
         topicBuilder -> topicBuilder.withDefaultTopicSpec(partitions, replication, retentionInDays)
         propsBuilder -> propsBuilder.withStreamAppConfig("kafka_broker:9092", "my-app-id"))
     .withActions(
         "event_sourcing_user", 
         "event_sourcing_account",
         "event_sourcing_auction",
         "async_payment")
     .withRetryStrategy(RetryStrategy.repeat(3, Duration.ofSeconds(10)))
     .run();
```

The saga coordinator needs to know which actions to support, and how to [serialize](/simple_sourcing_serialization.html) them. This is done with the 
[`withActions`](/apidocs-sagas/io/simplesource/saga/saga/app/SagaApp.html#withAction-java.lang.String-io.simplesource.saga.shared.topics.TopicConfigBuilder.BuildSteps-)
operations. It does not need to know the implementation details of these action processors.

The saga coordinator does not know about any saga graph topologies. It can accept any saga that the client defines via the [saga builder DSL](/simple_sagas_saga_builder_dsl.html), as longs as 
it knows about all the action types that are used in the saga.
