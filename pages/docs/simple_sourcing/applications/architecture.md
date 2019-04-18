---
layout: page
title: Architecture Overview
permalink: /simple_sourcing_architecture.html
---

Simple Sourcing is designed to take advantage of the horizontally scalable deployment model of Kafka Streams. A typical Simple Sourcing implementation will consist of multiple `EventSourcedApp` stream processing instances, each of them running in their own JVM process.

A client application such as a web service communicates with these Simple Sourcing instances via the `CommandApi` client API. The `EventSourcedClient` DSL provides a simple mechanism to create instances of the `CommandApi`. Communication between the client and the Simple Sourcing instances takes place via Kafka. Any Java application can be a simple sourcing client, it just needs to be able to access the same Kafka cluster.

This deployment model allows independent scaling of the web and the stream processing layers.