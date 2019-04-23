---
layout: page
title: Architecture Overview
permalink: /simple_sagas_architecture.html
---

Simple Sagas is designed to take advantage of the horizontally scalable deployment model of Kafka Streams.

A typical Simple Sagas implementation will consist of of multiple instances of:
* A [*saga coordinator*](/simple_sagas_saga_coordinator.html) application.
* [*Action processor*](/simple_sagas_action_processors.html) applications.
* [Simple sourcing applications](/simple_sourcing_application_builder.html) (if using [Simple sourcing](/simple_sourcing_docs_home.html) for event sourcing). 

As these are all KStream applications, they can be scaled horizontally by simply starting additional instances. All inter-process communication happens via Kafka.

Sagas are expressible as a directed acyclical graph (DAG) of saga actions. Clients can use the [saga builder DSL](simple_sagas_saga_builder_dsl.html) to build the saga.

Sagas can be executed by: 
* Publishing a [request](/apidocs-sagas/io/simplesource/saga/model/messages/SagaRequest.html) to the saga request topic, or
* Using the [client API](/simple_sagas_client_api.html) from any JVM application.

[Saga actions](/simple_sagas_key_concepts.html#actions-and-commands) must share a common underlying type. See the [serialization page](/simple_sagas_serialization.html#saga-action-command-type) for more details on this.

The deployment requirements and setup are identical to those for [Simple Sourcing](/simple_sourcing_deployment.html).
