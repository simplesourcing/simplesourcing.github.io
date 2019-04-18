---
layout: page
title: Design Overview
permalink: /simple_sourcing_design.html
---

With Simple Sourcing you are responsible for defining the domain model for your aggregates, the associated commands and events, 
and the functions for generating these entities. 

Simple Sourcing provides an [API](/simple_sourcing_api.html) for submitting these commands, 
and behind the scenes manages the persistence of the event store and any other state to Kafka.

Simple Sourcing also provides the guarantee that if a command is accepted, it will process it, 
generate one or more events and write them to the event log, and update the aggregate state
in a single atomic operation.
It also ensures that all events are written in the same order as the commands are received.

![Event Sourcing with CQRS](/images/simple-sourcing-diagram.png)

In the CQRS model implemented by Simple Sourcing, clients are able to submit **commands** to perform an action
against **aggregates** each identified by a unique key.

The **Command Handler** is responsible for accepting these commands and validating them against a **projection**
of the aggregate state. If it accepts the command, it generates one or more **events** to represent the changes
made to the aggregate. These events are pushed into an **Event Store**, an append-only record of all the events
applied to each aggregate.

One or more **Projectors** listen on the stream of events and build up a materialized view or **projection**
of the aggregation.

Simple Sourcing is an opinionated API. We have made a couple of design decisions you need to be aware of:

   * A single topic is used for the event store for each _type_ of aggregate. The unique identifier of the aggregate is
     used as the key for the messages pushed into Kafka guaranteeing that inserting order will be preserved for all updates
     on a given aggregate.
   * Publishing a command is an asynchronous operation. To guarantee ordering of updates into the event store
     we push commands through a topic in Kafka, allowing us to process all commands for a given key serially while
     still providing a highly performant, horizontally scalable solution.
