---
layout: page
title: Quick Start
permalink: /simple_sagas_quick_start.html
---

## Including Simple Sagas in your project

[![Latest version](https://maven-badges.herokuapp.com/maven-central/io.simplesource/simplesaga-saga/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.simplesource/simplesaga-saga)

Include core saga dependencies in your build file:

* Group ID: `io.simplesource`
* Artifact ID: `simplesaga-<module>`
* Version: `?.?.?` - Latest version in Maven Central, as in above badge.

Where `<module>` is one or more of:
* `saga` - the Saga coordinator
* `kafka` - the action processor
* `serialization` - serialization helpers (recommended)

## Getting the source

#### [Core repo](https://github.com/simplesourcing/simplesaga)
```bash
git clone https://github.com/simplesourcing/simplesaga.git
```

#### [Scala repo](https://github.com/simplesourcing/simplesaga-scala)
```bash
git clone https://github.com/simplesourcing/simplesaga-scala.git
```

### Pre-requisites

Ensure your local developer machine has the following tools installed 

   * [Java 8 SDK or later](http://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html)
   * [Maven 3.5.x](https://maven.apache.org/download.cgi)
   * [Docker](https://download.docker.com/mac/stable/Docker.dmg)
   * [SBT (for Scala code and examples)](https://www.scala-sbt.org/)
   
### IntelliJ setup

If you choose to use IntelliJ as your IDE, make sure you have the Maven and Lombok plugins installed.

### Building the source code 

Simply run the following:

```bash
make build
```

### Running sample code

The Sample code is now in the [Scala Repo](https://github.com/simplesourcing/simplesagas-scala)

From the project folder for this repo, in separate terminal windows:

1. Start kafka stack
    ```bash
    docker-compose up
    ```
1. Start the command processor, the action processor and the saga coordinator:
    
    ```bash
    sbt "user/runMain io.simplesource.saga.user.all.App"
    ```

1. Run the client app to submit some saga requests
    ```bash
    sbt "user/runMain io.simplesource.saga.user.client.App"
    ```
    
The Kafka topics are created as required.
    
If it all runs correctly, you should see some console output that ends with this sort of thing:
```text
09:02:24.725 [saga-...] INFO  SagaStream - stateTransitionsActionResponse: 1b230f=SagaActionStatusChanged(1b230fc9-fa9b-40f7-8007-01e5831f4d93,a2ad32f5-54b0-43b4-bb24-49678e508c56,Completed)
09:02:24.963 [saga-...] INFO  SagaStream - sagaState: 1b230f=InProgress=>5-(e06ac7,Completed)-(719a58,Completed)-(a2ad32,Completed)
09:02:25.222 [saga-...] INFO  SagaStream - sagaState: 1b230f=Completed=>6-(e06ac7,Completed)-(719a58,Completed)-(a2ad32,Completed)
```

