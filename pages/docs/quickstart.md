---
layout: page
title: Quick Start
permalink: quickstart.html
---

We have used Docker to allow developers to easily run a full suite of Kafka components locally.
Docker isn't a requirement to use Simple Sourcing. We have found Docker provides 
a highly productive development environment where software engineers each get their own independent
test environment they can tear down and recreate in seconds. This is especially useful
when dealing with strongly typed data in Kafka where you are likely to make many breaking changes
to schemas during development.

## Getting the source

Core repo: 
```
git clone https://github.com/simplesourcing/simplesource.git
```

Examples: 
```
git clone https://github.com/simplesourcing/simplesource-examples.git
```

## Pre-requisites

Ensure your local developer machine has the following tools installed 

   * [Java 8 SDK or later](http://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html)
   * [Maven 3.5.x](https://maven.apache.org/download.cgi)
   * [Docker](https://download.docker.com/mac/stable/Docker.dmg)
   
## Local setup

Docker manages all the infrastructure, however there are a few extra steps required to
make some customisations to the environment required for the library.

### IntelliJ setup

If you choose to use IntelliJ as your IDE, make sure you have the Maven and Lombok plugins installed.

## Running examples

Take a look in the Examples repo for some working examples of event sourcing systems using
Simple Sourcing. Each example has the following structure

```
src/main/avro - Avro schema definitions for aggregate key, commands, events and projections
src/main/java - Example runner along with all command handlers and projectors
src/test/java - Unit and property based tests
```

Each of the example applications has one or more `Runner` classes with main methods to run simple tests of the included aggregate types. Run these from your IDE or from the command line using Maven

#### User Avro example
```bash
mvn install && \
    mvn exec:java -pl examples/user \
    -Dexec.mainClass=io.simplesource.example.user.avro.UserAvroRunner
```

#### Auction example

##### Dependencies

```bash
cd examples/auction
docker-compose up
```

##### Backend
```bash
mvn install -DskipTests && \ 
    mvn exec:java -pl examples/auction \
    -Dexec.mainClass="io.simplesource.example.auction.RestApplication"
```
 from the project root folder, or run `./run.sh` from the `examples/auction` folder.

### Running examples in Docker

The `auction` example comes with a React front end application. Docker compose files are provided to run the stack partly or fully in Docker.

*   Just the dependencies in Docker (including Kafka, Zookeeper, MongoDb):

    Run `docker-compose up` from the `examples/auction` folder.

*   The front end in Docker:

    Run `docker-compose up` from the `examples/auction-frontend` folder, and go to [http://localhost:3000](http://localhost:3000).

The backend must be run locally, as described above.

### Running front end locally

1. Change to `examples/auction_frontend`
1. Run `yarn install` to install the dependencies.
1. Run `yarn start` to run the front end application.
1. go to [http://localhost:3000](http://localhost:3000) to view in browser.

##### Frontend

You will need a current version of the Node.js with npm

```bash
npm install -g yarn
cd examples/auction-frontend
yarn install
yarn start
```

## Troubleshooting

### InvalidStateStoreException

If when trying the example runners you see something like the following exception...

```
Exception in thread "main" java.lang.RuntimeException: java.util.concurrent.ExecutionException: org.apache.kafka.streams.errors.InvalidStateStoreException: The state store, commandResponseStore, may have migrated to another instance.
```

It means you have more than one instance of the application running locally using the same state directory. 
Either run one at once locally, or ensure all local instances use different state store directories.
