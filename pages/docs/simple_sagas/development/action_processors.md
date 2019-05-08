---
layout: page
title: Action Processors
permalink: /simple_sagas_action_processors.html
---

Action processors execute the individual actions that constitute a saga.

A action processor is simply consumes an [action request](/apidocs-sagas/io/simplesource/saga/model/messages/ActionRequest.html), 
and publishes an [action response](/apidocs-sagas/io/simplesource/saga/model/messages/ActionResponse.html). In this event driven paradigm, the message is the interface.

It can also have an effect (such as write to a Kafka topic, send an email), and indeed it normally does. This effect must be idempotent - if an identical action request is received with the same `CommandId` as one that has already processed, 
it should republish the response, but not repeat the effect.

All actions have an associated action type. This is used to:
* Determine which action processor is used to process an action.
* To separate action requests into different topics, so that an action processor only need to listen for action requests that it knows how to process.

To utilise action processors, it is recommended to create an [`ActionApp`](/apidocs-sagas/io/simplesource/saga/action/ActionApp.html) and then add them with the `withActionProcessor` method. 

### Simple Sagas action processors

The following action processors are provided by Simple Sagas out-the-box:

#### Event sourcing action processor
  
This turns action requests into Simple Sourcing command requests and translates the command response into action responses.

It also extends the consistency pattern across a saga. Simple sourcing saves the version number of an [aggregate](/simple_sourcing_key_concepts.html#aggregates) instance each time it is written.
For each aggregate written in the saga, Simple Sagas remembers the version number and if it encounters 
that aggregate instance later in the same saga, it will check the version number to ensure that the aggregate has not been overwritten by another process.

This action processor is indispensible for building more complex event sourcing systems.

To use the event sourcing action processor, use the [`EventSourcingBuilder`](/apidocs-sagas/io/simplesource/saga/action/eventsourcing/EventSourcingBuilder.html), 
passing in an [`EventSourcingSpec`](/apidocs-sagas/io/simplesource/saga/action/eventsourcing/EventSourcingSpec.html),
which contains the data and functions required to define the it.
 
#### Async action processor

This invokes an arbitrary asynchronous function. When this function completes, a response is returned to the saga coordinator. The result of the function can also 
be logged to an arbitrary Kafka topic.

This action processor is extremely useful for sagas that need to coordinate between microservices.

To use the async action processor, use the [`AsyncBuilder`](/apidocs-sagas/io/simplesource/saga/action/async/AsyncBuilder.html), 
passing in an [`AsyncSpec`](/apidocs-sagas/io/simplesource/saga/action/eventsourcing/AsyncSpec.html),
which contains the data and functions required to define the it.

The key parameter is the [`asyncFunction`](/apidocs-sagas/io/simplesource/saga/action/async/AsyncSpec.html#asyncFunction). 

#### Http action processor
  
A thin wrapper around the async action processor is provided to interact with Http web services. It is interface only, as it better left up to the 
client to decide which Http client framework to use.

### Dynamic undo actions

The actions required to execute a saga are defined in advance by the client, prior to submitting the saga. 
The client can also define an undo or compensation action for each action in the saga.

However for certain actions, it is not possible
to know in advance what the undo action should be.

For example consider the scenario where an action is to book a hotel. 
This involves calling the booking endpoint of a hotel booking service.
The endpoint returns a booking confirmation code.
The booking service also has a cancellation endpoint that takes a booking code as a parameter.
This booking point is only known once the booking has been executed.

To satisfy this requirement, the [action response](/apidocs-sagas/io/simplesource/saga/model/messages/ActionResponse.html) message has an optional undo command.

This allows the action processor to dynamically construct an undo command, which represents the compensation action that gets executed should a later action in the saga fail.
This undo command is passed back to the saga coordinator, and it overrides any statically defined undo command that was included in the initial saga graph.

Part of the specification for both an event sourcing action processor and a async processor is a function that returns an undo command.

Note that this function has a different signature for the these two action processors. 
* Simple sourcing commands don't return results, so we can't use the result of the command to generate an undo command.
  
  However in the event sourcing model it should generally not be necessary if we design the domain operations appropriately. 
  For example the undo command for a command that debits an account by $100 is a command that credits an account by $100.
  
  In addition, with Simple Sourcing, a single action type covers all the operations on a single aggregate, so the action type for the undo 
  command should be the same as the original action type. This is enforced by the event sourcing action processor specification.
  
* Async action processors on the other hand generally return results, and this result required to generate the undo command, as described in the hotel booking scenario above.
  In addition, the cancel hotel endpoint is likely to be a different endpoint from the book hotel endpoint, and is better represented as a 
  different action type.
  
  To cover these requirements, the undo function for the async action processor takes into account the result from the async invocation 
  and the original request, and allows the user to specify a different action type for the undo command.
  
  
### Creating an action processor app

To create an action processor stream app, creating an [`ActionApp`](/apidocs-sagas/io/simplesource/saga/action/ActionApp.html),
add the action processors you would like it to support, and run it with the required properties.

Your main method may look something like this:

```java
EventSourcingSpec<SpecificRecord, AccountCommand, AccountId, AccountCommand> accountCommandSpec =
        EventSourcingSpec.builder()
                .actionType("account_action")
                .aggregateName("account")
                .decode(a -> Result.success((AccountCommand) a))
                .commandMapper(c -> c)
                .keyMapper(AccountCommand::getId)
                .sequenceMapper(c -> Sequence.position(c.getSequence()))
                .timeout(Duration.ofSeconds(30))
                .build();

EventSourcingSpec<SpecificRecord, AuctionCommand, AuctionId, AuctionCommand> auctionCommandSpec = ...;

ActionApp
        .of(actionSerdes)
        .withActionProcessor(EventSourcingBuilder.apply(
                accountCommandSpec,
                topicBuilder -> topicBuilder.withTopicPrefix("action-"),
                topicBuilder -> topicBuilder.withTopicPrefix("command-")
        .withActionProcessor(EventSourcingBuilder.apply(
                auctionCommandSpec,
                topicBuilder -> topicBuilder.withTopicPrefix("action-"),
                topicBuilder -> topicBuilder.withTopicPrefix("command-")
        .run(propertiesBuilder -> 
            propertiesBuilder.withStreamAppConfig(appId, bootstrapServers);
```  
  
### Custom action processors  
  
It is possible to get a long way using the built in action processor, but is is also easy to define a custom action processor.

The easiest way to to this is to look at the implementations of the above action processors, and follow the same pattern.

This involves creating an [action processor build step](/apidocs-sagas/io/simplesource/saga/action/app/ActionProcessorBuildStep.html) that can be added to the [`ActionApp`](/apidocs-sagas/io/simplesource/saga/action/ActionApp.html).

This functional interface is a specification for:
* The KStream topology for the action processor
* The topic configuration for the required topics (each action processor has its own set action request and response topics)
* A handler for performing cleanup and freeing resources when the stream ends
