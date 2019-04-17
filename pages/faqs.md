---
layout: page
title:  "FAQs"
permalink: /simple_sourcing_faqs.html
---

# Frequently Asked Questions

## Why do you require the commands to be Avro objects?

In our original design, commands were processed locally by the API so there was no need for them to be Avro objects.
The problem with this approach was in guaranteeing ordering of events pushed into the event store topic. Since Kafka
doesn't currently support conditional writes, we couldn't easily prevent two clients adding an event with the same
sequence number at the same time for the same aggregate. Some form of coordination between the clients was required.

Kafka uses consumer groups to solve this problem when multiple consumers are reading from a topic. Rather than building
our own complex coordination system, we wanted to piggyback on Kafka's solution to this problem. In order to do this,
we required routing the commands through Kafka and hence mandating the use of Avro. Once we'd done this, since we're
using the aggregate key as the key to this topic, we could guarantee a single consumer would process all commands
for a given aggregate in order.

## What is the the process for recovery of a projection?

Projections are technically not stored, they should be conceptually viewed
as at point in time cache of the event journal.

That is a bit simplified since projections can join multiple journals etc.
But conceptually your recovery is replay from start.

In reality for performance reasons etc often point in time checkpoints are
used to speed it up.

In Simple Sourcing we use a point in time optimisation using the KTable,
but its data can be reproduced completely from just the event topic.

In our implementation Kafka Streams does checkpoint the KTable so recovery
will be from the checkpoint (but that is really an implementation optimisation).

