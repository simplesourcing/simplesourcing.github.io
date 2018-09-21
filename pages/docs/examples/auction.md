---
layout: page
title: Auction
permalink: auction.html
---

# Auctions

This example is a full application including a web tier and materialized
view for reading. We walk through all the important concepts.
The complete code can be found under
`examples/auction`.
The following diagram shows an example of the full cycle to create an account

![Projections](/images/auction-example-app.png?raw=true "Projections")

## Aggregates

Firstly we decide on the aggregates for the domain. In this case we have
2 aggregates called `Account` and `Reservation` representing the current state of the system. 

### `Account`
It is a representation of user account. Only draft reservations are stored in the account aggregate

```java
@Data
@Builder(toBuilder = true)
@AllArgsConstructor
public final class Account {
    @NonNull
    private final String username;
    @NonNull
    private Money funds;
    @Builder.Default
    private List<Reservation> fundReservations = Collections.emptyList();
}
```

### `Reservation`
It represents fund reservations(transactions) for user account, like for example reserve, confirm, and cancel funds

```java
@Data
@Builder(toBuilder = true)
@AllArgsConstructor
public class Reservation {
    @NonNull
    private ReservationId reservationId;
    @NonNull
    private String description;
    @NonNull
    private Money amount;
    private Status status;

    public enum Status {
        DRAFT,
        CANCELLED,
        CONFIRMED
    }
}
```

## Events

Now that we have an aggregate, we define events to describe the changes to the
aggregate over time. These events will be stored (in kafka) to record
the sequence of changes to each Account.

Once events are stored they are an immutable record that the aggregate
has been changed. As such they are named with a past tense. They often
will follow a pattern of xxxUpdated, xxxDeleted etc.

For Accounts we have the following:

```java
public final class AccountEvents {
    @Value
    public static final class AccountCreated implements AccountEvent {
        final String username;
        final Money initialFunds;
    }

    @Value
    public static final class AccountUpdated implements AccountEvent {
        final String username;
    }

    @Value
    public static final class FundsAdded implements AccountEvent {
        final Money addedFunds;
    }

    @Value
    public static final class FundsReserved implements AccountEvent, AccountTransactionEvent {
        final ReservationId reservationId;
        final String description;
        final Money amount;

        @Override
        public ReservationId getReservationId() {
            return reservationId;
        }
    }

    @Value
    public static final class FundsReservationCancelled implements AccountEvent, AccountTransactionEvent {
        final ReservationId reservationId;

        @Override
        public ReservationId getReservationId() {
            return reservationId;
        }
    }

    @Value
    public static final class ReservationConfirmed implements AccountEvent, AccountTransactionEvent {
        final ReservationId reservationId;
        final Money amount;

        @Override
        public ReservationId getReservationId() {
            return reservationId;
        }
    }

    public interface AccountEvent {
    }

    public interface AccountTransactionEvent {
        ReservationId getReservationId();
    }
}
```

Once again we use the Lombok `@Data`, `@Value` annotations to reduce boilerplate.

### Avro mapping

As it is the events which are persisted to Kafka, each event must be
serializable. If you are using Json, then no extra code may be
necessary, however for Avro, an idl and mapping code will be required.

For each event add an equivalent record to the avro idl. For example:

```
  record AccountCreated {
    string username;
    decimal(12,4) initialAmoount;
  }

  record AccountUpdated {
    string username;
  }

  record FundsAdded  {
    decimal(12,4) amount;
  }

  record FundsReserved  {
    string reservationId;
    string description;
    decimal(12,4) amount;
  }

  record ReservationCancelled {
    string reservationId;
  }

  record FundsReleased  {
    string reservationId;
    decimal(12,4) amount;
  }

  record AccountEvent {
    union {AccountCreated, FundsAdded, FundsReserved, ReservationCancelled, FundsReleased} event;
  }
```

The `GenericMapper` interface defines functions to convert between domain
objects and Avro objects. They are implemented in `AccountAvroMappers` and
simply return objects in the correct form. 
In the following example we are using domain mapper DSL to define mappers for different account event types. 

Converting from a domain event to an Avro object:

```java
private static GenericMapper<AccountEvents.AccountEvent, GenericRecord> buildEventMapper() {
        DomainMapperBuilder builder = new DomainMapperBuilder(new DomainMapperRegistry());

        builder.mapperFor(AccountEvents.AccountCreated.class, AccountCreated.class)
                .toSerialized(event -> new AccountCreated(event.username(), event.initialFunds().getAmount()))
                .fromSerialized(event -> new AccountEvents.AccountCreated(event.getUsername(), 
                    Money.valueOf(event.getInitialAmount())))
                .register();

        builder.mapperFor(AccountEvents.AccountUpdated.class, AccountUpdated.class)
                .toSerialized(event -> new AccountUpdated(event.username()))
                .fromSerialized(event -> new AccountEvents.AccountUpdated(event.getUsername()))
                .register();

        builder.mapperFor(AccountEvents.FundsAdded.class, FundsAdded.class)
                .toSerialized(event -> new FundsAdded(event.addedFunds().getAmount()))
                .fromSerialized(event -> new AccountEvents.FundsAdded(Money.valueOf(event.getAmount())))
                .register();

        builder.mapperFor(AccountEvents.FundsReserved.class, FundsReserved.class)
                .toSerialized(event -> new FundsReserved(event.reservationId().asString(),  event.description(), 
                    event.amount().getAmount()))
                .fromSerialized(event -> new AccountEvents.FundsReserved(new ReservationId(event.getReservationId()),
                        event.getDescription(), Money.valueOf(event.getAmount())))
                .register();

        builder.mapperFor(AccountEvents.FundsReservationCancelled.class, ReservationCancelled.class)
                .toSerialized(event -> new ReservationCancelled(event.reservationId().asString()))
                .fromSerialized(event -> new AccountEvents.FundsReservationCancelled(new ReservationId(event.getReservationId())))
                .register();

        builder.mapperFor(AccountEvents.ReservationConfirmed.class, FundsReleased.class)
                .toSerialized(event -> new FundsReleased(event.reservationId().asString(), event.amount().getAmount()))
                .fromSerialized(event -> new AccountEvents.ReservationConfirmed(new ReservationId(event.getReservationId()),
                        Money.valueOf(event.getAmount())))
                .register();

        builder.withExceptionSupplierForNotRegisteredMapper(() -> new IllegalArgumentException("Event Class not supported"));

        return builder.build();
}
```
Another example of defining domain mapper for `AccountKey` by using domain mapper DSL
```java
static GenericMapper<AccountKey, GenericRecord> accountKeyDomainMapper() {
    DomainMapperBuilder builder = new DomainMapperBuilder();

    builder.mapperFor(AccountKey.class, AccountId.class)
            .toSerialized(k -> new io.simplesource.example.auction.account.wire.AccountId(k.asString()))
            .fromSerialized(k -> new AccountKey(k.getId()))
            .register();

    return builder.build();
}
```
## Aggregators

An `Aggregator` is a function which takes the current
aggregate, applies an event to it to produce a new updated aggregate.
One `Aggregator` is required for each event and we implement these alongside the events. 
For more details please check  [event aggregators](eventaggregators.html)

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
```  

### Commands

Commands are requests to create an event which indicates the aggregate
was changed. There should be one command for each event. For Accounts we
have the following commands:

```java
public interface AccountCommand {
    @Data
    class CreateAccount implements AccountCommand {
        private final String username;
        private final Money initialFunds;
    }

    @Data
    class UpdateAccount implements AccountCommand {
        private final String username;
    }

    @Data
    class AddFunds implements AccountCommand {
        private final Money funds;
    }

    @Data
    class ReserveFunds implements AccountCommand {
        private final ReservationId reservationId;
        private final Money funds;
        private final String description;
    }

    @Data
    class CancelReservation implements AccountCommand {
        private final ReservationId reservationId;
    }

    @Data
    class ConfirmReservation implements AccountCommand {
        private final ReservationId reservationId;
        private final Money finalAmount;
    }
}
```

Commands are also stored in Kafka as such they are required to be
serializable. If using Avro then avro idl, and domain mappers will be required.

## Command Handlers

When the system reads commands it must decide whether or not the
command can be processed, and if it can create the appropriate event(s).
This is done by implementing `CommandHandlers`. For example:

```java
private static CommandHandler<AccountKey, AccountCommand.CreateAccount, AccountEvents.AccountEvent, Optional<Account>> doCreateAccount() {
    return CommandHandler.ifSeq(
            (accountId, expectedSeq, currentSeq, currentAggregate, command) -> currentAggregate
                    .map(d -> failure("Account already created: " + accountId.id()))
                    .orElse(success(new AccountEvents.AccountCreated(command.username(),command.initialFunds()))));
}

private static CommandHandler<AccountKey, AccountCommand.UpdateAccount, AccountEvents.AccountEvent, Optional<Account>> doUpdateAccount() {
    return CommandHandler.ifSeq(
            (accountId, expectedSeq, currentSeq, currentAggregate, command) -> currentAggregate
                    .map(d -> AccountMappedAggregate.success(new AccountEvents.AccountUpdated(command.username())))
                    .orElse(failure("Attempted to update non-existent account: " + accountId.id())));
}
```

Here you see we use a helper function `CommandHandler.ifSeq`. This function
checks that the current sequence number is the one we expect and creates
an optimistic lock on changes to the account. In other words, if we think
the account owner's name is called John and want to update his name to Jonny, this would
fail if another update has already changed his name to Jon.

## Projections

The purpose of event sourcing is to have the ability to generate different types of projections that are optimised 
for reading. Then using the generated events, we could build inifinite number of projections in different shapes.
In this example we created 2 projections from the account event stream. 

![Projections](/images/projection.png?raw=true "Projections")

### Account projection

It is amlost the same as the `Account` aggregate (we could say that aggregate is a special case of projection)
In this example we use KStream to consume account events to generate the account projection as shown in the following KStream diagram

```java
@Data
@Builder(toBuilder = true)
@AllArgsConstructor
public final class AccountProjection {
    private final String username;
    private final Money funds;
    private final List<Reservation> draftReservations;
}
```

![Account projexction KStream topology](/images/account_projection_stream.png?raw=true "Account projection KStream topology")

### Account transaction projection

This projection is designed for account transactions(fund reservations), it also gets generated by consuming 
account's reservations events. All other account events (like `CreateAccount` and `AccountUpdated`) are filtered out.
Also, the events get re-grouped by different key, which in this case is like:

```
record AccountTransactionId {
    string accountId;
    string reservationId;
}
```

![Account transaction projection](/images/account_transaction_projection_flow.png?raw=true "Account transactions Projections") 


## Wiring it all together

For events and commands we need to map the event objects to `Aggregators`
and the command objects to `CommandHandlers`. 

### Commands and events

For events and commands we need to map the event objects to `Aggregators`
and the command objects to `CommandHandlers`. Both are achieved in a very
similar way:

```java
private static CommandHandler<AccountKey, AccountCommand, AccountEvents.AccountEvent, Optional<Account>> accountCommandHandlers() {
    return CommandHandlerBuilder.<AccountKey, AccountCommand, AccountEvents.AccountEvent, Optional<Account>>newBuilder()
            .onCommand(AccountCommand.CreateAccount.class, doCreateAccount())
            .onCommand(AccountCommand.UpdateAccount.class, doUpdateAccount())
            .onCommand(AccountCommand.AddFunds.class, doAddFunds())
            .onCommand(AccountCommand.ReserveFunds.class, doReserveFund())
            .onCommand(AccountCommand.CancelReservation.class, doCancelReservation())
            .onCommand(AccountCommand.ConfirmReservation.class, doConfirmReservation())
            .build();
}
``` 

and

```java
public static Aggregator<AccountEvent, Optional<Account>> getAggregator() {
    return AggregatorBuilder.<AccountEvent, Optional<Account>>newBuilder()
            .onEvent(AccountCreated.class, handleAccountCreated())
            .onEvent(AccountUpdated.class, handleAccountUpdated())
            .onEvent(FundsAdded.class, handleFundsAdded())
            .onEvent(FundsReserved.class, handleFundsReserved())
            .onEvent(FundsReservationCancelled.class, handleCancelledFundsReservation())
            .onEvent(ReservationConfirmed.class, handleConfirmedFundsReservation())
            .build();
}
```

### AggregatorSpecs

An `AggregatorSpec` defines the settings required for Simple Sourcing to
run. In particular it calls `getAggregator()` and `accountCommandHandlers()`.

```java
static <S> AggregateSpec<AccountKey, AccountCommand, AccountEvents.AccountEvent, S, Optional<Account>> createSpec(
        final String name,
        final DomainSerializer<AccountKey, AccountCommand, AccountEvents.AccountEvent, S, Optional<Account>> domainSerializer,
        final ResourceNamingStrategy resourceNamingStrategy,
        final InitialValue<AccountKey, Optional<Account>> initialValue) {
    return AggregateBuilder.<AccountKey, AccountCommand, AccountEvents.AccountEvent, S, Optional<Account>>newBuilder()
            .withName(name)
            .withDomainSerializer(domainSerializer)
            .withResourceNamingStrategy(resourceNamingStrategy)
            .withInitialValue(initialValue)
            .withAggregator(getAggregator())
            .withCommandHandler(accountCommandHandlers())
            .build();
}
```

## Business logic

With all the bits in place we can now write some business logic to create
and update account. As shown in the following diagram, projection store (MongoDB in our example) is queried 
to support uniqeness of account's username. 

![Business logic validation](/images/simple-sourcing-update-account-command.png?raw=true "Business logic validation")

### Validation

The following code is taken from `AccountWriteServiceImpl` shows how account projection repository is used to validate 
uniqueness of username and account ID. Then creates and sends `CreateAccount` command to the Simple Sourcing framework

```java    
@Override
public FutureResult<CommandAPI.CommandError, Sequence> createAccount(AccountKey accountKey, Account account) {
    requireNonNull(account);
    requireNonNull(accountKey);

    Optional<AccountView> existingAccount = accountRepository
            .findByAccountId(accountKey.id().toString());

    List<Reason<CommandAPI.CommandError>> validationErrorReasons =
            Stream.of(
                    validUsername(account.username()),
                    validateFunds(account.funds(), "Initial fund can not be negative"),
                    existingAccount.map(acc -> Reason.of(CommandAPI.CommandError.InternalError, "Account with same ID already exist")),
                    usernameNotTakenBefore(accountKey, account.username())
            )
                    .filter(Optional::isPresent)
                    .map(Optional::get)
                    .collect(Collectors.toList());

    if (!validationErrorReasons.isEmpty()) {
        return FutureResult.fail(NonEmptyList.fromList(validationErrorReasons));
    }

    logger.info("Creating account with username {} and initial fund is {}", account.username(), account.funds());
    AccountCommand.CreateAccount command = new AccountCommand.CreateAccount(account.username(), account.funds());
    return accountCommandAPI.publishAndQueryCommand(new CommandAPI.Request<>(accountKey, Sequence.first(), UUID.randomUUID(), command),
            Duration.ofMinutes(1)).map(NonEmptyList::last);

}
```

### Optimistic locking

Simple Sourcing framework support optimistic locking to avoid concurrent updates to the same aggregate instance.
For this feature to work, client needs to send the sequence of the last event used to update the projection. 
This means that the client needs to keep track of the consumed events. The following code shows how to updae an account

```java
sendCommandForAccount(accountKey, new AccountCommand.UpdateAccount(username));
private <C extends AccountCommand> FutureResult<CommandAPI.CommandError, UUID> sendCommandForAccount(AccountKey accountKey, C command) {
    Optional<AccountView> maybeAccount = accountRepository.findByAccountId(accountKey.asString());

    return maybeAccount
            .map(a -> accountCommandAPI.publishCommand(new CommandAPI.Request<>(accountKey,
                    Sequence.position(a.getLastEventSequence()), UUID.randomUUID(), command)))
            .orElse(FutureResult.fail(Reason.of(CommandAPI.CommandError.CommandPublishError, "Account does not exist")));
}
```

The client needs to pass the last event sequence while publishing update command. 
This sequence is loaded from the account projection repository.

### Projection store

As we mentioned before, in auction example we use KStream to generate 2 projections from account events. We then use
Kafka Connect SinkConnector for MongoDB. All the configuration for this connector can be found in `examples/auction/scripts`
