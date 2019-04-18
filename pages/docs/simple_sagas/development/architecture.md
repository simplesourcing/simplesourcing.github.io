---
layout: page
title: Architecture Overview
permalink: /simple_sagas_architecture.html
---

Simple Sagas is designed to take advantage of the horizontally scalable deployment model of Kafka Streams.

A typical Simple Sagas implementation will consist of of multiple instances of:
* The [*saga coordinator*](/simple_sagas_saga_coordinator.html) app.
* [*Action processor*](/simple_sagas_action_processors.html) apps.
* [Simple sourcing applications](/simple_sourcing_application_builder.html) if using [Simple sourcing](/simple_sourcing_docs_home.html) for event sourcing. 

As these are all KStream applications, they can be scaled horizontally by simply starting additional instances.

All communication between these processes happens via Kafka.

Sagas are expressible as a directed acyclical graph (DAG) of saga actions. 

Clients can use the [saga builder DSL](simple_sagas_saga_builder_dsl.html) to build the saga, 
and then submit a request to execute the saga by publishing a message to the saga request topic (if part of an existing Kafka streams workflow),
 or alternatively use the [client API](/simple_sagas_client_api.html) from any JVM application.

