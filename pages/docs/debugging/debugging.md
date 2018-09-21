---
layout: page
title: Debugging and Monitoring
permalink: debugging.html
---

# Debugging and Monitoring

## Logging

- What logging library
- How to configure

## Metrics

- Kafka metrics
- Plugging in a metrics sink (e.g. Datadog, Splunk, ELK...)

## Broker access

Docker manages connections between all the docker containers. However if you want to access the Kafka broker
from outside a container, there's one extra step you need to do. Kafka is very specific about the hostname
it listens on - it must match the value specified via the `KAFKA_ADVERTISED_LISTENERS` environment variable.

If you are using Docker for Mac >= 1.12, Docker for Linux, or Docker for Windows 10, then please add the following lines
to `/etc/hosts` or `C:\Windows\System32\Drivers\etc\hosts`:

```
127.0.0.1	broker
```

Once that's added you can then run and debug your Kafka Streams app in your favourite IDE connecting to the Kafka
broker running within docker.

## Clearing state

Often during development you'll make incompatible changes to your domain model schemas. When running locally, the easiest
approach to handle these changes is to blow away the old environment.

```
docker-compose down
rm -r /tmp/kafka-streams
docker-compose up -d
```
