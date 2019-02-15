---
layout: page
title: Users Example in Detail
permalink: users.html
---

Here we will explore a very simple example of creating and updating a user
with a first and last name. The complete code can be found under
`examples/user`.

## Aggregates

Firstly we decide on the aggregates for the domain. In this case we have
a single aggregate called `User` representing the current state of the
system.

```java
@Data
@Builder(toBuilder = true)
@AllArgsConstructor
public class User {
    private final String firstName;
    private final String lastName;
    private final Integer yearOfBirth;
}
```

An aggregate is modeled as a POJO and here we use Lombok annotations to
avoid writing boilerplate code for getters/setters/constructors etc.

## Events

Now that we have an aggregate we events to describe the changes to the
aggregate over time. These events will be stored (in Kafka) to record
the sequence of changes to each User.

Once events are stored they are an immutable record that the aggregate
has been changed. As such they are named with a past tense. They often
will follow a pattern of xxxUpdated, xxxDeleted etc.

For Users we have the following:

```java
public interface UserEvent {

    @Data
    class UserInserted implements UserEvent {
        private final String firstName;
        private final String lastName;
    }

    @Data
    class FirstNameUpdated implements UserEvent {
        private final String firstName;
    }

    @Data
    class LastNameUpdated implements UserEvent {
        private final String lastName;
    }

    ...
}
```

Once again we use the Lombok `@Data` annotation to reduce boilerplate.


### Avro mapping

As it is the events which are persisted to Kafka, each event must be
serializable. If you are using Json, then no extra code may be
necessary, however for Avro, an IDL and mapping code will be required.

For each event add an equivalent record to the Avro IDL. For example:

```java
  record UserInserted {
    string firstName;
    string lastName;
  }

  record FirstNameUpdated {
    string firstName;
  }

  record LastNameUpdated {
    string lastName;
  }

  record YearOfBirthUpdated {
    union {null, int} yearOfBirth;
  }
```

The `DomainMapper` interface defines functions to convert between domain
objects and Avro objects. They are implemented in `UserAvroMappers` and
simply return objects in the correct form.

Converting from an Event (the Domain) to an Avro object:

```java
if (value instanceof UserEvent.UserInserted) {
    final UserEvent.UserInserted event = (UserEvent.UserInserted)value;
    return new UserInserted(event.firstName(), event.lastName());
}
```

Converting from an Avro object to a User domain object is equally simple:

```java
final GenericRecord specificRecord = mapper.fromGeneric(serialized);
if (specificRecord instanceof UserInserted) {
    final UserInserted event = (UserInserted)specificRecord;
    return new UserEvent.UserInserted(event.getFirstName(), event.getLastName());
}
```

## Aggregators

An `Aggregator` is a function which takes the current
aggregate, applies an event to it to produce a new updated aggregate.
One `Aggregator` is required for each event and we implement these
alongside the events.

```java
static Aggregator<YearOfBirthUpdated, Optional<User>> handleYearOfBirthUpdated() {
    return (currentAggregate, event) ->
            currentAggregate.map(user -> user.toBuilder()
                    .yearOfBirth(event.yearOfBirth())
                    .build());
}
```

`Aggregator` is a Functional Interface in Java, so the `handleXyz` methods
return a lambda function which does the update to the current aggregate.

The `currentAggregate` is defined as `Optional<User>` so `map` ensures
the update is only applied if the value exists.

### Commands

Commands are requests to create an event which indicates the aggregate
was changed. There should be one command for each event. For Users we
have the following commands:

```java
public interface UserCommand {

    @Data
    class InsertUser implements UserCommand {
        private final String firstName;
        private final String lastName;
    }

    @Data
    class UpdateName implements UserCommand {
        private final String firstName;
        private final String lastName;
    }

    @Data
    class UpdateYearOfBirth implements UserCommand {
        private final Integer yearOfBirth;
    }

    ...
 }
 ```

Commands are also stored in Kafka and as such are required to be
serializable. If using Avro then Avro IDL, and domain mappers will be
required.

## Command handlers

When the system reads commands it must decide whether or not the
command can be processed, and if it can create the appropriate event.
This is done by implementing `CommandHandlers`. For example:

```java
static CommandHandler<UserKey, UpdateYearOfBirth, UserEvent, Optional<User>> doUpdateYearOfBirth() {
    return CommandHandler.ifSeq(
            (userId, currentAggregate, command) -> currentAggregate
                    .map(d -> success(
                            new UserEvent.YearOfBirthUpdated(command.yearOfBirth())))
                    .orElse(failure("Attempted to update non-existent user: " + userId.id())));
}
```

Here you see we use a helper function `CommandHandler.ifSeq`. This function
checks that the current sequence number is the one we expect and creates
an optimistic lock on changes to the user. In other words, if we think
the user is called John and want to update his name to Jonny, this would
fail if another update has already changed his name to Jon.

Once again, within the update, `currentAggregate` is `Optional` so we use
`map` to only update it if it exists. In the case the `currentAggregate`
doesn't exist we return a `failure` with an appropriate error message.

## Wiring it all together

We now have all the basic classes and functions for our domain and we
need to hook everything together.

### Commands and events

For events and commands we need to map the event objects to `Aggregators`
and the command objects to `CommandHandlers`. Both are achieved in a very
similar way:

```java
static Aggregator<UserEvent, Optional<User>> getAggregator() {
    return AggregatorBuilder.<UserEvent, Optional<User>> newBuilder()
            .onEvent(UserInserted.class, handleUserInserted())
            .onEvent(FirstNameUpdated.class, handleFirstNameUpdated())
            .onEvent(LastNameUpdated.class, handleLastNameUpdated())
            .onEvent(YearOfBirthUpdated.class, handleYearOfBirthUpdated())
            .onEvent(UserDeleted.class, handleUserDeleted())
            .onEvent(BuggyEvent.class, handleBuggyEvent())
            .build();
```

and

```java
    static CommandHandler<UserKey, UserCommand, UserEvent, Optional<User>> getCommandHandler() {
        return CommandHandlerBuilder.<UserKey, UserCommand, UserEvent, Optional<User>>newBuilder()
                // Command handling
                .onCommand(InsertUser.class, doInsertUser())
                .onCommand(UpdateName.class, doUpdateName())
                .onCommand(UpdateYearOfBirth.class, doUpdateYearOfBirth())
                .onCommand(DeleteUser.class, doDeleteUser())
                .onCommand(BuggyCommand.class, doBuggyCommand())
                .build();
    }
```

Here you can see the builders maintain a mapping between the class types
and the functions which handle them.

### AggregatorSpecs

An `AggregatorSpec` defines the settings required for Simple Sourcing to
run. In particular it calls `getAggregator()` and `getCommandHandler()`.

```java
static public <S> AggregateSpec<UserKey, UserCommand, UserEvent, S, Optional<User>> createSpec(
        final String name,
        final DomainSerializer<UserKey, UserCommand, UserEvent, S, Optional<User>> aggregateSerdes,
        final ResourceNamingStrategy resourceNamingStrategy,
        final InitialValue<UserKey, Optional<User>> initialValue
) {
    return AggregateBuilder.<UserKey, UserCommand, UserEvent, S, Optional<User>>newBuilder()
            .withName(name)
            .withDomainSerializer(aggregateSerdes)
            .withResourceNamingStrategy(resourceNamingStrategy)
            .withInitialValue(initialValue)
            .withAggregator(UserEvent.getAggregator())
            .withCommandHandler(UserCommand.getCommandHandler())
            .build();
}
```

## Business logic

With all the bits in place we can now write some business logic to create
and update users.

```java
public static FutureResult<CommandError, NonEmptyList<Sequence>> submitCommands(
        final CommandAPI<UserKey, UserCommand> commandAPI
) {
    final UserKey key = new UserKey("user" + System.currentTimeMillis());
    final String firstName = "Sarah";
    final String lastName = "Dubois";

    return commandAPI
        .publishAndQueryCommand(new CommandAPI.Request<>(
            key,
            Sequence.first(),
            UUID.randomUUID(),
            new UserCommand.InsertUser(firstName, lastName)),
            Duration.ofMinutes(2L)
        )
        .flatMap(sequences -> {


            logger.info("Received result {} new sequences", sequences);
            return commandAPI.publishAndQueryCommand(new CommandAPI.Request<>(
                key,
                sequences.last(),
                UUID.randomUUID(),
                new UserCommand.UpdateName("Sarah Jones", lastName)),
                Duration.ofMinutes(2L)
            );
        });
}
```

There is a bit to this so we'll go through it:

* The return type `FutureResult<CommandError, NonEmptyList<Sequence>>` is
  a wrapper around Java's `Future`. This can be read as a result of
  either a `CommandError` or a `NonEmptyList` of `Sequence`s at some point
  in the future. It is important to understand that when `submitCommands`
  is called, nothing is actually executed. The result is just some commands
  that may be executed in the future.
* `Sequence` is a monotonically increasing number representing the event
  that was written successfully to the queue.
* There is one parameter, `CommandAPI<UserKey, UserCommand>` which is the
  interface into Simple Sourcing and allows us to create `FutureResult`s.
* `commandAPI.publishAndQueryCommand` allows you to send a command and
  query the result of that command. It is necessary to query the result
  because sending a command is an asynchronous operation. The parameters
  are as follows:
  * `key` - a unique identifier for this user that we have generated
  * `Sequence.first()` - the state we expect the user to be in. In
  this, case as the user hasn't been created yet, it will be the `0`, or
  the `first()` number in a sequence.
  * `UUID.randomUUID()` - An identifier for this command so it can be
  queried later. As we call `publishAndQueryCommand` there is no need for
  this to be retained for later use.
  * `new UserCommand.InsertUser(firstName, lastName))` - the actual
  command to submit.
  * `Duration.ofMinutes(2L)` - a timeout to wait when querying the result
  of the command.
* As discussed above, the result of the command is a `FutureResult` which
  has not executed anything. At this point we want to run a second
  operation to amend the user so we append a second operation with
  `.flatMap(sequences -> {`. This second operation has the same syntax as
  the first and the result will again be a `FutureResult`

## The main method

We can finally put everything together. We need to get hold of a
`commandAPI` so it can be passed into the business logic. The following
builds one with the relevant Kafka settings.

```java
final CommandAPISet aggregateSet = new EventSourcedApp()
    .withKafkaConfig(builder ->
        builder
            .withKafkaApplicationId("userAvroApp1")
            .withKafkaBootstrap("localhost:9092")
            .withApplicationServer("localhost:1234")
            .build())
    .addAggregate(UserAvroAggregate.createSpec(
        aggregateName,
        avroAggregateSerdes,
        new PrefixResourceNamingStrategy("user_avro_"),
            k -> null
    ))
    .start()
    .getCommandAPISet();
final CommandAPI<UserId, GenericRecord> api =
    aggregateSet.getCommandAPI(aggregateName);
```

Finally we can call:
```java
submitCommands(api).unsafePerform(e -> CommandError.InternalError);
```

`unsafePerform` is the action which will actually run the business logic
and either fail or succeed. Up until that is called no commands have
actually been submitted.
