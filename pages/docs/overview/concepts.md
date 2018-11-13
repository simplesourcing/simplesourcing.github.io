---
layout: page
title: Concepts
permalink: concepts.html
---

## Aggregates

An *aggregate* is an entity or a group of entities in the domain that are
treated as a single unit.

## Events

*Events* are things that happen in the domain. They are immutable facts 
that record how the domain changes over time.

## Commands

A *command* is a request to change the system. When a command is executed, it is validated,
and if accepted, one or more events are created.

## Projections

A *projection* is a view of the current state of an aggregate or a sub-domain. 

Projections are realized by applying the events from the beginning of time to get to the current state.

The aggregate state is itself a projection - this is generated and maintained by Simple Sourcing. For generating
all other projections and aggregations, we recommend using Kafka Streams or KSQL. 

#### Further reading
   * [Event Sourcing and CQRS](event_sourcing.md) - a brief description of these concepts
{% include_relative further_reading.md %}
 
