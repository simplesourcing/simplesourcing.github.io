---
layout: page
title: Testing
permalink: testing.html
---

The `simplesource-command-testutils` module includes a couple of classes to help in testing your application.

`SimpleSourcingAPITestDriver` implements both the `CommandAPI` and `QueryAPI` interfaces. It uses the Kafka Streams
[TopologyTestDriver](https://cwiki.apache.org/confluence/display/KAFKA/KIP-247%3A+Add+public+test+utils+for+Kafka+Streams)
test tool to run the same topology used by the Simple Sourcing production code in your tests, mocking out
both the Schema Registry and Kafka.

`SimpleSourcingAPITestHelper` provides a DSL built on top of `SimpleSourcingAPITestDriver` for chaining together
publication of commands and making assertions about the inner state of the event sourcing library
after each command. It also provides a simple mechanism to validate the correct events and projection changes
occurred as a result of each command.

Below we insert a new user then update their name. In each case we provide the expected events and projection state.
The second command will automatically get sent with the same key and sequence number of the last event
from the previous command.

```java
testHelper.publishCommand(
    id,
    Sequence.first(),
    new UserCommand(new InsertUser(firstName, lastName)))
    .expecting(
        NonEmptyList.of(new UserEvent(new UserInserted(firstName, lastName))),
        Optional.of(new User(id, firstName, lastName, null))
    )
    .thenPublish(
        new UserCommand(new UpdateName(updatedFirstName, updatedLastName)))
    .expecting(
        NonEmptyList.of(
            new UserEvent(new FirstNameUpdated(updatedFirstName)),
            new UserEvent(new LastNameUpdated(updatedLastName))),
        Optional.of(new User(id, updatedFirstName, updatedLastName, null))
    )
```

There is also a variation of the `thenPublish` method where you provide a function
that takes the last projection value and sequence and produces a new command with your own provided sequence id.
This is useful when you're testing error scenarios. Below we're creating a new user then sending an update command
with an invalid sequence id. We indicate we expect an error providing the expected failure reason.

```java
testHelper.publishCommand(
    id,
    Sequence.first(),
    new UserCommand(new InsertUser(firstName, lastName)))
    .expecting(
        NonEmptyList.of(new UserEvent(new UserInserted(firstName, lastName))),
        Optional.of(new User(id, firstName, lastName, null))
    )
    .thenPublish(update ->
        new ValueWithSequence<>(new UserCommand(new UpdateName(updatedFirstName, updatedLastName)), Sequence.first()))
    .expectingFailure(NonEmptyList.of(InvalidReadSequence))
```
