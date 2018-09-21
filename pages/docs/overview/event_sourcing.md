---
layout: page
title: Introduction to Event Sourcing and CQRS
permalink: event_sourcing.html
---

#### Event Sourcing

Event Sourcing is a design pattern in which one treats the events that happen in the domain 
as the fundamental source of truth about the domain's state.

As an example, a double-entry accounting ledger is an event sourced system. 
The ledger entries are the events, and are the primary source of truth.
The balance sheet represents the current state of the system, and is derived by applying the ledger entries in sequence.

#### CQRS

CQRS stands for Command Query Responsibility Segregation. It is a distributed design pattern in which the responsibility 
for writing to a system is separated from the responsibility for reading from it.

It is frequently used to improve performance by relieving the write subsystem of all querying responsibilities,
and separately optimising the read and write subsystems for their respective workloads. 
Reading and writing are often performed by different services.

With CQRS the following principles hold true:
* A query does not modify the system state.
* A command to update system state is only either accepted or rejected - it does not reveal system state.

#### Event Sourcing and CQRS

Although event sourcing and CQRS are distinct concepts, they are complementary.

Though it is possible to do event sourcing without CQRS, separating command and query responsibilities results in a 
cleaner and simpler event sourcing implementation.

Although CQRS also can be done without event sourcing, 
a good way to decouple the read from the write subsystems is for 
the write subsystem to publish changes to an event log,
and the query subsystem to materialize the views it requires by applying these changes.

#### Further reading
   * [Concepts](concepts.md) - glossary of key concepts
   
{% include_relative further_reading.md %}
 
