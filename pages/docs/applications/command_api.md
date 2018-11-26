---
layout: page
title: Command API
permalink: command_api.html
---

A `CommandAPISet` provides a `CommandAPI` for each of the aggregates added. The `CommandAPI` has methods to publish commands,
and wait for the result to these commands.

The `CommandAPISet` is created by the [client builder](client_builder.html).

Note that result of the command only will only be whether the command succeeded or failed:
* If successful, the command sequence number
* If failure, the reason(s) for failure

Here is an example, submitting a command, and when this returns, submits another.

```java
CommandAPI commandAPI = ...;
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
```

A Simple Sourcing application will often be deployed as a service that exposes endpoints for the commands that will be executed.
These endpoints interact with the `CommandAPI` interface. 
