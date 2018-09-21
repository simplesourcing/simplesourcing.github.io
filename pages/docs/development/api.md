---
layout: page
title: API Overview
permalink: api.html
---

## Java

TODO: link to JavaDocs...

### Command Handlers

[Command handlers](commandhandlers.html) are the mechanism for validating commands, and turning them into events.

### Aggregator

[Events aggregators](eventaggregators.html) are used to updated the aggregate state from the stream of events.

### Command API

[The Command API](runners.html) is the user API exposed by Simple Sourcing for submitting commands, and listening for the result.

### FutureResult

Commands are submitted asynchronously. The result of command or any validation errors are returned in a future.

### AggregatorSpec

An AggregateSpec represents all the details that are required for a single aggregate, including command and event 
handler and serialization information, all in one place.

They can be created a assembled with the [AggregateSetBuilder DSL](runners.html).

### Serde

A Serde is a mechanism for [**ser**ializing and **de**zerialising](serialization.html) your domain objects. 
Serdes are required by any operation that may move data on the Kafka cluster.

## Kotlin

Coming soon...

## Scala

coming soon...
