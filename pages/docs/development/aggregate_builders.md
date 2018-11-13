---
layout: page
title: Aggregate Builders
permalink: aggregate_builders.html
---

Having defined the aggregate model, and the associate aggregate command and event handlers, we use the `AggregateBuilder` builder 
to wire these all together and produce a `AggregateSpec`. 

An `AggregateSpec` represents everything Simple Sourcing needs to know about a single aggregate.

```java
AgregateSpec userSpec = AggregateBuilder.<UserKey, UserCommand, UserEvent, Optional<User>>newBuilder()
                .withName("user")
                .withSerdes(avroAggregateSerdes)
                .withResourceNamingStrategy(new PrefixResourceNamingStrategy("application_avro_"))
                .withInitialValue((k) -> Optional.empty())
                .withAggregator(UserEvent.getAggregator())
                .withCommandHandler(UserCommand.getCommandHandler())
                .build();
```

We can define multiple aggregate types, each with their own `AggregateSpec`,
then assemble them all with the [AggregateSetBuilder DSL](aggregate_sets.html), and start the application.
