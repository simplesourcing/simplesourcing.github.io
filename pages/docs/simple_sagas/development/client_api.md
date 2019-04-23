---
layout: page
title: Client API
permalink: /simple_sagas_client_api.html
---

A saga can be executed using the saga client API. The client has to be able to access the same cluster 
that the saga coordinator is running on.

It is the responsibility of the client to define the [saga graph](simple_sagas_key_concepts.html#saga-graph).
This can be done using the [Saga builder DSL](/simple_sagas_saga_builder_dsl.html). 
The client then uses the client API to execute the saga.

The [saga API](/apidocs-sagas/io/simplesource/saga/model/api/SagaAPI.html) has two methods:
* `submitSaga` - submits a request to execute a saga.
* `getSagaResponse` - asynchronously queries the result of saga execution. It returns a [`FutureResult`](/apidocs/io/simplesource/data/FutureResult.html) that asynchronously completes with the result of the saga when the saga completes.

The Kafka implementation of this API publishes a saga request to the saga request topic in Kafka when `submitSaga` is called, 
and waits for a result to appear in the saga result topic to complete `getSagaResponse`.

To create an instance of the saga API, a [saga client builder](/apidocs-sagas/io/simplesource/saga/client/api/SagaClientBuilder.html) is available.

For example:

```java
SagaAPI<A> sagaApi = SagaClientBuilder
     .create(propsBuilder -> propsBuilder.withBootstrapServers("kafka_broker:9092"))
     .withSerdes(serdes)
     .withClientId(clientId)
     .withScheduler(scheduler)
     .build();
```

Some notes:

1. `withSerdes` is to specify a [SagaClientSerdes](/apidocs-sagas/io/simplesource/saga/model/serdes/SagaClientSerdes.html) used to serialise the saga request and saga responses.
2. `withClientId` is used to create a private topic to stream saga results to. This enables the client to listen to only the it made. Each client instance should have a unique client ID.





