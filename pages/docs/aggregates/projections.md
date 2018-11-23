---
layout: page
title: Query-side projection
permalink: projections.html
---

Generating complex query side projections is out of scope for Simple Sourcing.

Simple Sourcing only takes care of generating a projection for the current aggregate state.

For all other projections such as projections derived by aggregating or transforming the event sequence in different ways, 
or projections involving joins on multiple aggregates, we suggest using Kafka Streams directly.

![Alt text](../../../images/event-sourcing-projections.png?raw=true "Projections")
