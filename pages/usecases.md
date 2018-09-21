---
layout: page
title:  "Use Cases"
section: "section_home"
permalink: usecases.html
---

## Use cases

Event Sourcing provides many benefits that make it a good solution for
data. It is ideal for use when developing micro-services so that each
service can see all the events in the system and create a materialized
view of the events specifically suited to that service.

* Different Views - The events may be projected into many different read
  views depending on how they need to be consumed. For example they could
  be stored in document form in MongoDb or Elastic Search and at the same
  time in a relational form in Postgres.
* Reliable Auditing - As the system stores changes to the domain model
  it automatically has a full audit of every change in the system.
* Temporal Queries - By replaying events up until a point in time the
  entire state of the system can easily be re-created at that point in time.
* Scalablity - Write performance is only limited by Kafka which is already
  massively scalable and materialized views are readonly so they can be
  easily scaled horizontally.

## When is Event Sourcing not a suitable solution?

By it's nature, micro-services and materialized views consuming the event
queue will only ever be eventually consistent. This is fine for many
applications but not all. If the solution requires a stronger consistency
then event sourcing may not be the right solution.
