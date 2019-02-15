---
layout: page
title: Aggregate API Overview
permalink: aggregate_api.html
---

The Simple Sourcing programming model is aggregate based. The most important design decision is determining what these aggregates are, and the commands and events that related to them. As events are immutable, any events that are generated must be supported throughout the life of the application. 

For each aggregate we need to implement command handlers, event handlers, and Serdes for serialization.

### Command handlers

[Command handlers](commandhandlers.html) are the mechanism for validating commands, and turning them into events.

### Event handlers

[Events handlers](eventaggregators.html) are used to update the aggregate state from the stream of events.

### Serde

A Serde is a mechanism for [**ser**ializing and **de**serializing](serialization.html) your domain objects. 
Serdes are required by any operation that may move data on the Kafka cluster.
