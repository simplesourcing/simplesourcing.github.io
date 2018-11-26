---
layout: page
title: Introduction
permalink: docs_home.html
sidebar: docs_sidebar
---

Simple Sourcing is an API for building event sourcing systems where Kafka is used as the primary data store and System of record.

Simple Sourcing has specifically kept its footprint small and looks to solve a very focused problem. 
It is not meant to be a full blown digital platform, for this there a a number of existing platforms that meet those needs well. 
Instead the focus of this project is to provide the core primitives to build a CQRS system that leverages an event journal as its primary data store (commonly referred to as *event sourcing*). 

We have decided to focus most of our effort on the *command processing* and *event generation* side of the CQRS system with the view that the Kafka Streams technology provides a very rich and expressive API for reading and transforming event streams and generating view projections from them. 

We may investigate some options to providing a lightweight read side API in the future but for now that is out of scope of the core project.

This documentation assumes basic knowledge of [Event Sourcing and CQRS](event_sourcing.html) principles and terminology. 

If you're new to either of these, there's some great introductory articles and talks listed in the *Background reading* section below to get you started.

## Why use Simple Sourcing?

[Here are some of the key benefits](key_benefits.html) of using Simple Sourcing for building an event sourcing application using Kafka.

## Getting Started

If you want to jump in straight away and see an example system running locally see the [Quick Start](quickstart.html)

For more information on the design of the system see the [Design Overview](design.html)

## Project Source

Core repo: [https://github.com/simplesourcing/simplesource](https://github.com/simplesourcing/simplesource)

Examples: [https://github.com/simplesourcing/simplesource-examples](https://github.com/simplesourcing/simplesource-examples)


## Background reading

   * [Event Sourcing and CQRS](overview/event_sourcing.md) - a brief description of these concepts
{% include_relative further_reading.md %}
