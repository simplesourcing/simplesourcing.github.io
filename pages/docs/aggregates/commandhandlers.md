---
layout: page
title: Command Handlers
permalink: commandhandlers.html
---

An aggregate has one or more commands associated with it, and each of these commands, once invoked, may result in one or more events being generated.

The command handler's job is to validate an incoming command, and either accept or reject it. If the command is accepted it must emit one or 
more events. If it is rejected, or command processing fails, it should return with one or more reasons for failure.

An aggregate requires a command handler for each command that operates on it. These command handlers can be combined to create a single command handler for the aggregate.

## Defining command handlers

To define a command handler, provide an implementation of the [`CommandHandler`](/apidocs/io/simplesource/api/CommandHandler.html)
interface for each aggregate type you wish to support, and add them using the Command handler builder DSL.

### Command handler builder DSL

Most aggregates are acted upon by multiple commands. 

A [`CommandHandlerBuilder`](/apidocs/io/simplesource/dsl/CommandHandlerBuilder.html)
builder can be used to define a multi-command command handler on an aggregate from a set single-command command handlers.

For example:

```java
CommandHandlerBuilder.<AccountKey, AccountCommand, AccountEvents.AccountEvent, Optional<Account>>newBuilder()
        .onCommand(AccountCommand.CreateAccount.class, doCreateAccount())
        .onCommand(AccountCommand.UpdateAccount.class, doUpdateAccount())
        .onCommand(AccountCommand.AddFunds.class, doAddFunds())
        .build();
```

Here `doCreateAccount`, `doUpdateAccount` etc. are command handlers for individual accounts.


### Best practices for Command Handlers

1. It is recommended to do some sanity checking on command parameters prior to invoking the `CommandAPI`. E.g. testing for negative bounds.
1. The command handler function must perform all validation required to ensure that the aggregate always has consistent state.
1. Command handlers, should be pure functions and should neither block execution, nor have side effects, nor modify incoming state.
1. Sequence checking is performed by default before the command handler is invoked. This ensures that the user is always executing a command 
based on the latest version of the aggregate. This check can be suppressed by adding the build step `withInvalidSequenceStrategy(InvalidSequenceStrategy.LastWriteWins)` to the `AggregateBuilder`.