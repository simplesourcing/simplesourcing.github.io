---
layout: page
title: API Overview
permalink: api.html
---

## API Documentation

Javadocs are available [here...](/apidocs)

## Programming Model

### Command API

[The Command API](command_api.html) is the user API exposed by Simple Sourcing for submitting commands, and listening for the result.

### Command Handlers

[Command handlers](commandhandlers.html) are the mechanism for validating commands, and turning them into events.

### Aggregator

[Events aggregators](eventaggregators.html) are used to updated the aggregate state from the stream of events.

### AggregatorSpec

An AggregateSpec represents all the details that are required for a single aggregate, including command and event 
handler and serialization information, all in one place.

They can be created a assembled with the [AggregateBuilder DSL](aggregate_builders.html).

### Serde

A Serde is a mechanism for [**ser**ializing and **de**serializing](serialization.html) your domain objects. 
Serdes are required by any operation that may move data on the Kafka cluster.

## Language choice

Simple Sourcing is witten in Java, and can be used in any JVM language.

## Scala

There is currently no dedicated Scala API avaiable. However it is easy to create a Simple Sourcing application in Scala.
[An example is provided](https://github.com/simplesourcing/simplesagas/blob/master/modules/user/src/main/scala/command/App.scala) in the experimental [Simple Sagas](https://github.com/simplesourcing/simplesagas) project.

This example includes helper functions for generic derivation of Serdes for Json serialization using [Circe](https://circe.github.io/circe/).

A dedicated Scala DSL may be added in the future.

## Kotlin

As with Scala, it is easy and natural to use Simple Sourcing from a Kotlin application.