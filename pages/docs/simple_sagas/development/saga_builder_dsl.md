---
layout: page
title: Saga Builder DSL
permalink: /simple_sagas_saga_builder_dsl.html
---

A saga can be built using the [`SagaBuilder` DSL](/apidocs-sagas/io/simplesource/saga/client/dsl/SagaDSL.html).

This DSL is used to create saga actions, and define dependencies between them.
1. Create a builder:
    ```java
    import io.simplesource.saga.client.dsl.SagaDSL;
    import static io.simplesource.saga.client.dsl.SagaDSL;
    
    SagaBuilder<A> sagaBuilder = SagaDSL.createBuilder();
    ```

1. Define some actions:
    ```java
    SubSaga<A> a = sagaBuilder.addAction(actionIdA, actionCommandA, undoCommandA);
    SubSaga<A> b = sagaBuilder.addAction(actionIdB, actionCommandB, undoCommandB);
    SubSaga<A> c = sagaBuilder.addAction(actionIdC, actionCommandC, undoCommandC);
    ...
    ```
    `A` is the [serializable action command type](/simple_sagas_serialization.html), typically something like 
    [`SpecificRecord`](https://avro.apache.org/docs/1.8.2/api/java/org/apache/avro/specific/SpecificRecord.html).


1. Create dependencies between them:

    **Examples:**
    
    Execute `a`, then `b`, then `c`:
    ```java
    a.andThen(b).andThen(c)
    ```
    
    ----------
    
    Execute `a`, then `b`, `c` and `d` in parallel, then `e`:
    ```java
    a.andThen(b).andThen(e)
    a.andThen(c).andThen(e)
    a.andThen(d).andThen(e)
    ```
    This can also be expressed as:
    ```java
    a.andThen(inParallel(b, c, d)).andThen(e)
    ```
    
    ----------
    
    Execute `a`, then `b`, `c` and `d` in series, then `e`. The following are equivalent:

    ```java
    a.andThen(b).andThen(c).andThen(d).andThen(e)
    inSeries(a, b, c).andThen(d).andThen(e)
    inSeries(a, b, c, d, e)
    a.andThen(inSeries(b, c, d)).andThen(e)
    inSeries(List.of(a, b, c, d, e))
    ```
    The latter is useful when we have a sequence of actions of length only known at runtime.
    
    The `andThen` operator is associative. This means that the following are equivalent:
    ```java
    a.andThen(b).andThen(c)
    ```
    and
    ```java
    a.andThen(b.andThen(c))
    ```
    This associativity property enables building larger sagas from smaller ones.

1. Build the saga:
    ```java
    Result<SagaError, Saga<A>> sagaBuildResult = sagaBuilder.build();
    ```
    
    If the Saga fails validation,such as has cyclical references or attempts to combine subsagas created with different builders, an error is created.


### Scala DSL

A DSL is available for Scala users, loosely based on Akka Streams, equivalent to the Java DSL, but prettier and easier to read.

Examples:
```scala
a ~> b ~> c
```
```scala
a ~> inParallel(b, c, d) ~> e
a ~> List(b, c, d).inParallel ~> e
```
  
