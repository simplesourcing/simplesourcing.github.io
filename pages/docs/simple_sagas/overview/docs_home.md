---
layout: page
title: Introduction
permalink: /simple_sagas_docs_home.html
sidebar: docs_sidebar
---

A [*saga*](/simple_sagas_key_concepts.html) is a protocol for executing distributed transactions. Simple Sagas is an framework for defining and executing sagas using Kafka.

The framework consists of:
* A [*Saga builder DSL*](/simple_sagas_saga_builder_dsl.html) to define saga requests, and a [*client API*](/simple_sagas_client_api.html) to submit requests and handle the response.
* A [*saga coordinator*](/simple_sagas_saga_coordinator.html) that receives saga requests, guides them through the full execution lifecycle, 
and publishes a response when the saga is complete.
* [*Action processors*](/simple_sagas_action_processors.html) that execute the individual actions that make up the saga.

Kafka is used for all inter-process communication and state persistence.

### Design goals

Simple Sagas is designed to make building saga applications easy:

* Functional interfaces - everything should be defined and created in a functional, declarative style, with no complex ordering dependencies to figure out (e.g. you need to invoke "A" before you can create "B").
* Minimised Kafka exposure - user should not need to be a Kafka and Kafka streams expert to use the framework.
* Sensible defaults - everything works out the box with as little configuration as possible, yet it's still possible to fine tune and override everything.
* Topic management - topic creation is integrated into the framework. Any topic that is required will be created by the platform at the right time.

### Next Steps

Try one of the following:

* Learn about [sagas](/simple_sagas_key_concepts.html), their properties and related concepts.
* Read more about the Simple Sagas [application architecture](/simple_sagas_architecture.html).
* Go to the [quick start](/simple_sagas_quick_start.html) and start using Simple Sagas.
* View the [Javadocs](/apidocs-sagas/)
