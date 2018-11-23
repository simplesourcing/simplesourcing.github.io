---
layout: page
title: Architecture Overview
permalink: architecture.html
---

Simple Sourcing is designed to take advantage of the horizontally scalable deployment model of Kafka Streams. A typical Simple Sourcing implementation will consist of multiple `EventSourcedApp` stream processing instances, each of them running in their own JVM process.

A client application, frequently a web server or web application, communicates with these Simple Sourcing instances via the `CommandApi` client API. The `EventSourcedClient` DSL provides a simple mechanism to create instances of the `CommandApi`.

This deployment model allows independent scaling of the web and the stream processing layers.