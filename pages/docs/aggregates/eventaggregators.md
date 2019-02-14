---
layout: page
title: Event aggregators
permalink: eventaggregators.html
---

An event handler, or event aggregator, is a function which takes the current aggregate, 
applies an event to it to produce a new updated aggregate. 
One [`Aggregator`](/apidocs/io/simplesource/api/Aggregator.html) is required for each event.

### Event aggregator DSL

An aggregator [`AggregatorBuilder`](/apidocs/io/simplesource/dsl/AggregatorBuilder.html) builder can be used to define aggregator functions for multiple event types from a set
of single-event aggregators. 

For example:

```java
private static Aggregator<AccountCreated, Optional<Account>> handleAccountCreated() {
    return (currentAggregate, event) ->
            Optional.of(new Account(event.username(), event.initialFunds(), emptyList()));
}

private static Aggregator<AccountUpdated, Optional<Account>> handleAccountUpdated() {
    return (currentAggregate, event) -> currentAggregate.map(a -> a.toBuilder().username(event.username()).build());
}

private static Aggregator<FundsAdded, Optional<Account>> handleFundsAdded() {
    return (currentAggregate, event) -> currentAggregate.map(a -> a.addFunds(event.addedFunds()));
}

public static Aggregator<AccountEvent, Optional<Account>> getAggregator() {
    return AggregatorBuilder.<AccountEvent, Optional<Account>>newBuilder()
            .onEvent(AccountCreated.class, handleAccountCreated())
            .onEvent(AccountUpdated.class, handleAccountUpdated())
            .onEvent(FundsAdded.class, handleFundsAdded())
            .build();
}
```  

### Best practices for aggregators

Some best practices need to be followed to implement the aggregator correctly:

* `Pure function`: the aggregator should not have any side effect, and it should not modify the passed aggregate state.
It should rather do a deep copy of the passed aggregate, apply the event and return this altered aggregate.
* `Non-blocking`: The event aggregator is part of a Kafka transaction which has a timeout. So if the aggregator function 
takes longer than expected, this might cause rebalanced and instability to the whole Simple Sourcing.
Any blocking operation should be implemented as a separate client which reads from the generated Simple Sourcing events topic
