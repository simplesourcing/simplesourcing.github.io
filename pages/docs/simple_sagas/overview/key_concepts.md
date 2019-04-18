---
layout: page
title: Key Concepts
permalink: /simple_sagas_key_concepts.html
sidebar: docs_sidebar
---

### Sagas

A *Saga* is a protocol for executing distributed transactions. It is also very useful in executing long running transactions that may
take minutes to complete, or even days.
 Sagas were first proposed in a paper by [Garcia-Molina and Salem](https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf) 
as a solution to the problem of long running transactions.

A saga is defined as a set of *actions* or *transactions* that are executed in a specific sequence.
If any of the actions fail to complete, *compensating actions* may be executed for each of the actions that have
successfully completed. The aim of compensating actions is to restore system state to the state prior to the saga. 

The saga completes successfully once all actions have successfully completed. A saga completes with failure if an action fails
and all the compensating actions have completed.
This makes a saga an transaction an atomic action, in that either all actions succeed, or the saga fails and the compensating actions 
nullify the overall effect of the saga on system state.

### Saga graph

A useful representation of a saga is as a stateful directed acyclical graph. 
The direction of the arrows determines the dependencies, and hence the order in which saga actions are executed. 
Actions that are not dependent on each other can be executed in parallel.

### Actions and commands

Actions are the individual steps involved in saga execution. 
However depending on the saga state not necessarily the same thing gets executed.
When the saga is in failure mode, only the compensating actions get executed.

So from a Simple Saga nomenclature point of view we make the following distinction:
* *action* represents a vertex in the saga dependency graph.
* Each action has an associated *action command* or simply *command*.
* Each action may have an associated *compensation command* or *undo command*.

