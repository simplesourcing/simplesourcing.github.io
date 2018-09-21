---
layout: page
title: Command Handlers
permalink: commandhandlers.html
---

Once you have decided on the aggregates you wish to use for your application, you'll need to consider the commands
and corresponding events to support.

The command handler's job is to validate an incoming command, and either accept or reject it. If the command is accepted it must emit one or 
more events. If it is rejected, or command processing fails, it should return with one or more reasons for failure.

An aggregate can have multiple commands that operate on it, and requires a command handler for each command. These command handlers can 
be combined to create a single command handler for the aggregate.

## Define command handlers

To define a command handler, provide an implementation of the [CommandHandler](/simplesource-command-api/src/main/java/io/simplesource/api/CommandHandler.java)
interface for each aggregate type you wish to support. 

### Command handler builder DSL

Most aggregates are acted upon by multiple commands. 

A [CommandHandlerBuilder](/simplesource-command-api/src/main/java/io/simplesource/dsl/CommandHandlerBuilder.java)
builder can be used to define a multi-command command handler on an aggregate from a set single-command command handlers.

For example:

```java
CommandHandlerBuilder.<AccountKey, AccountCommand, AccountEvents.AccountEvent, Optional<Account>>newBuilder()
        .onCommand(AccountCommand.CreateAccount.class, doCreateAccount())
        .onCommand(AccountCommand.UpdateAccount.class, doUpdateAccount())
        .onCommand(AccountCommand.AddFunds.class, doAddFunds())
        .build();

```

### Best practices for Command Handlers

1. It is recommended to do some sanity checking on command parameters prior to invoking the `CommandAPI`. E.g. testing for negative bounds.
1. The command handler function must perform all validation required to ensure that the aggregate always has consistent state.
1. The command handler should check the command sequence number to ensure that it matches the aggregate sequence number. 
This is left up to the user to perform, as there are cases where it is appropriate to relax this restriction.
1. Command handlers, like event handlers, should also be pure functions, with no side effects, and not modify incoming state.
