---
layout: page
title: API Overview
permalink: programming_model.html
---

## Programming model

Building an event sourcing application using Simple Sourcing involves the following steps:

1. Define the aggregates the application will use
2. Implement the [aggregate command and event handlers](aggregate_api.html)
3. Create the [event sourcing streaming application](application_builder.html)
4. Create Simple Sourcing [client instances](client_builder.html) where required

## Language choice

Simple Sourcing is written in Java. Simple Sourcing applications can be implemented in and used from any JVM language.

### Scala

There is currently no dedicated Scala API available. However it is easy to create a Simple Sourcing application in Scala.
[An example is provided](https://github.com/simplesourcing/simplesagas/blob/master/modules/user/src/main/scala/command/App.scala) in the experimental [Simple Sagas](https://github.com/simplesourcing/simplesagas) project.

This example includes helper functions for generic derivation of Serdes for Json serialization using [Circe](https://circe.github.io/circe/).

A dedicated Scala DSL may be added in the future.

### Kotlin

As with Scala, it is easy and natural to use Simple Sourcing from a Kotlin application.

## API reference documentation

Javadocs are available [here...](/apidocs)
