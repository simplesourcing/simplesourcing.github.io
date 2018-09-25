---
layout: page
title: Quick Start
permalink: quickstart.html
---

## Including Simple Sourcing in your project

![](https://maven-badges.herokuapp.com/maven-central/io.simplesource/simplesource-command-api/badge.svg)

Include the following dependencies in you build file:

* Group ID: `io.simplesource`
* Artifact ID: `simplesource-command-<project>`
* Version: `?.?.?` - Latest version in Maven central, as above.

Where `<project>` is one or more of:
* `api` - core API (required)
* `kafka` - core Kafka implementation (required)
* `serialization` - for serialization helpers (optional)
* `testutils` - for testing utilities (optional)

## Getting the source

#### [Core repo](https://github.com/simplesourcing/simplesource)
```
git clone https://github.com/simplesourcing/simplesource.git
```

#### [Example repo](https://github.com/simplesourcing/simplesource-examples)
```
git clone https://github.com/simplesourcing/simplesource-examples.git
```

## Pre-requisites

Ensure your local developer machine has the following tools installed 

   * [Java 8 SDK or later](http://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html)
   * [Maven 3.5.x](https://maven.apache.org/download.cgi)
   * [Docker](https://download.docker.com/mac/stable/Docker.dmg)
   
## Docker

We have used Docker to allow developers to easily run a full suite of Kafka components locally.
Docker isn't a requirement to use Simple Sourcing. We have found Docker provides 
a productive development environment where software engineers each get their own independent
test environment they can tear down and recreate in seconds.

### IntelliJ setup

If you choose to use IntelliJ as your IDE, make sure you have the Maven and Lombok plugins installed.

## Running examples

Take a look in the [Example repo](https://github.com/simplesourcing/simplesource-examples) for some working examples of event sourcing systems using
Simple Sourcing. 

Each example has the following structure

* `src/main/avro` - Avro schema definitions for aggregate key, commands, events and projections
* `src/main/java` - Example runner along with all command handlers and projectors
* `src/test/java` - Unit and property based tests


Each of the example applications has one or more `Runner` classes with main methods to run simple tests of the included aggregate types. Run these from your IDE or from the command line using Maven

#### User Avro example

1. **Start the backend**
    
    From the project root folder:
    
    ```bash
    docker-compose up
    ```

1. **Start the backend**

    ```bash
    cd examples/user
    mvn install &&  mvn exec:java -Dexec.mainClass=io.simplesource.example.user.avro.UserAvroRunner
    ```

#### Auction example

The dependencies and the front end run in Docker. The backend is run locally.

1. **Starting the dependencies**
    
    ```bash
    cd examples/auction
    docker-compose up
    ```

1. **Starting the backend**
    
    ```bash
    cd examples/auction
    ./run.sh
    ```

1. **Starting the front end** 
    
    *In Docker:*
    
    ```bash
    cd examples/auction-frontend
    docker-compose up
    ```
    
    *Or locally:*
    
    You will need a recent version of Node.js with npm
    
    ```bash
    npm install -g yarn
    cd examples/auction-frontend
    yarn install
    yarn start
    ```
    
1. Open a browser at [http://localhost:3000](http://localhost:3000)

## Troubleshooting

### InvalidStateStoreException

If when trying the example runners you see something like the following exception...

```
Exception in thread "main" java.lang.RuntimeException: java.util.concurrent.ExecutionException: org.apache.kafka.streams.errors.InvalidStateStoreException: The state store, commandResponseStore, may have migrated to another instance.
```

It means you have more than one instance of the application running locally using the same state directory. 
Either run one at once locally, or ensure all local instances use different state store directories.
