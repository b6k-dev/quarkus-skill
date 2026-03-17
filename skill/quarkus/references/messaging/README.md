# Messaging Reference

Use this module when an event crosses a service or process boundary, or when you need broker-backed asynchronous messaging with durability, buffering, replay, or consumer scaling.

## Overview

Quarkus Messaging is built on MicroProfile Reactive Messaging with SmallRye connectors.

- Use it for Kafka, RabbitMQ, AMQP, Pulsar, MQTT, and similar broker-backed channels.
- Model application flow with named channels, `@Incoming`, `@Outgoing`, and `@Channel`.
- Keep business code connector-agnostic where possible; push broker details into configuration.
- Treat it as the default choice when messages must survive restarts or move between services.

## Boundary-First Chooser

```text
Is the event crossing a service/process boundary?
├── YES -> SmallRye Reactive Messaging + broker connector
└── NO (in-process only)
      │
      Do you need clustering or non-blocking event loop behavior?
      ├── YES -> Vert.x Event Bus (@ConsumeEvent)
      └── NO
            │
            Do you want portability and type safety?
            └── YES -> Jakarta CDI Events (@Observes)
```

Use this module for the first branch. For the other branches, route to the modules linked below.

## What This Covers

- Reactive Messaging channel model and connector-backed channels.
- `@Incoming`, `@Outgoing`, `@Channel`, `Emitter`, `MutinyEmitter`, and `Message` usage.
- Channel wiring, fan-in/fan-out, execution model, context propagation, and pausable channels.
- Health, metrics, tracing, TLS integration, Dev Services, and `InMemoryConnector` testing.

## What This Does Not Cover

- Plain CDI events for in-process observer patterns.
- Vert.x local event bus request/reply and pub/sub addresses.
- Low-level provider clients used directly instead of Reactive Messaging.

## Quick Routing

1. Channel annotations, emitters, metadata, execution control -> `api.md`
2. Connector and channel properties -> `configuration.md`
3. End-to-end integration and testing workflows -> `patterns.md`
4. Common delivery-model mistakes and Quarkus-specific limits -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Core channel APIs and copy-ready examples
[configuration.md](./configuration.md) - High-value messaging properties and channel config patterns
[patterns.md](./patterns.md) - Repeatable producer, consumer, stream, and test workflows
[gotchas.md](./gotchas.md) - Common pitfalls around execution, wiring, context, and testing

## See Also

- [../vertx-event-bus/README.md](../vertx-event-bus/README.md) - Local async request/reply and pub/sub with Vert.x addresses
- [../dependency-injection/README.md](../dependency-injection/README.md) - CDI events for same-process observer patterns
- [../configuration/README.md](../configuration/README.md) - Profiles and property layering for connector setup
- [../web-rest/README.md](../web-rest/README.md) - HTTP endpoints that publish or stream messages
