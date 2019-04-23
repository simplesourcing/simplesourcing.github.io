---
layout: page
title: Key Concepts
permalink: /simple_sagas_key_concepts.html
sidebar: docs_sidebar
---

### Sagas

A *Saga* is a protocol for executing distributed transactions. It is also very useful in executing long running transactions that may
take minutes to complete, or even days.
Indeed sagas were first proposed in a paper by [Garcia-Molina and Salem](https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf) 
as a solution to the problem of long running transactions.

A saga is defined as a set of *actions* or *transactions* that are executed in a specific sequence.
If any of the actions fail to complete, *compensating actions* may be executed for each of the actions that have
successfully completed. The aim of compensating actions is to restore system state to the state prior to the saga. 

The saga completes successfully once all actions have successfully completed. A saga completes with failure if an action fails
and all the compensating actions have completed.

### Saga graph

A useful representation of a saga is as a stateful directed acyclical graph. 
The direction of the arrows determines the dependencies, and hence the order in which saga actions are executed. 
Actions that are not dependent on each other can be executed in parallel.

If a saga action fails and the compensation process begins, the arrows of the saga graph are reversed, and the compensation operations are executed in the reverse order.

### Actions and commands

Actions are the individual steps involved in saga execution. 
However, depending on the saga state, not necessarily the same operation gets executed.
When the saga is in failure mode, the compensating actions get executed.

So from a Simple Saga nomenclature point of view we make the following distinction:
* *action* represents a vertex in the saga dependency graph.
* Each action has an associated *action command* or simply *command*.
* Each action may have an associated *compensation command* or *undo command*.

Compensation commands must be the *inverse* of the action command - they have the property that the state changes 
applied by an action command are reversed by the corresponding compensation command. 

Compensation commands should also have the property that they can't fail. If they do fail, this 
is considered an unexpected exception.  The remaining compensation actions are still executed, 
but the failure of the saga should be escalated.

### Consistency

Traditional relational databases satisfy the so called [ACID](https://en.wikipedia.org/wiki/ACID_(computer_science)) properties for transactions.
Saga transactions are able to preserve these properties, with the exception of the isolation property:
* The inverse property of compensation commands ensures the *atomicity* of the saga as a whole. A failed saga should have no overall effect on system state.
* Sagas do not preserve the *isolation*, as the intermediate states are visible as the saga progresses.
* Sagas transactions are *eventually consistent*.

### Idempotence

If a saga action command is executed, and no reply is received,
the saga coordinator may retry action execution. 
However it may be that the operation itself succeeded first time, but there was a failure sending the acknowledgement. 
In this case the operation may be executed twice.

*Idempotence* is the property that the effect of the operation is only applied the first time it is executed, 
and on subsequent executions it is ignored.

In a robust saga implementation action commands are designed to be idempotent.

