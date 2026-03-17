# Vert.x Event Bus Reference

Use this module when communication stays inside the application or Vert.x cluster and you want non-blocking message passing, request/reply, or publish/subscribe semantics without a broker connector.

## Overview

Quarkus exposes the Vert.x event bus through declarative consumers and the injected `EventBus` API.

- Use it for local asynchronous request/reply, fire-and-forget, or local pub/sub by address.
- Prefer it when event-loop integration and lightweight local messaging matter more than portability.
- It supports clustering, but it is still not a durable broker and does not provide stream processing semantics.

## When to Choose This Over Other Options

- Choose `vertx-event-bus` over `dependency-injection` when you need request/reply, event-loop friendly handlers, or Vert.x cluster transport.
- Choose `messaging` instead when events cross service boundaries and need durability, replay, or broker-managed scaling.

## What This Covers

- `@ConsumeEvent`, `EventBus`, `Message`, blocking vs non-blocking consumers, and virtual-thread handlers.
- Address configuration, request/reply, publish, codecs, and HTTP-to-bus bridging.
- Event bus and cluster configuration relevant to local or clustered transport.

## What This Does Not Cover

- Broker-backed messaging connectors and channel configuration.
- Plain CDI observer events for portable in-process domain events.
- General Vert.x client APIs outside event bus usage.

## Quick Routing

1. Consumers, send/publish/request, replies, codecs -> `api.md`
2. Cluster, transport, SSL, and timeout settings -> `configuration.md`
3. HTTP bridge, pub/sub, blocking, and codec workflows -> `patterns.md`
4. Common address, delivery, and execution mistakes -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Core Event Bus APIs and compact examples
[configuration.md](./configuration.md) - High-value transport and clustering properties
[patterns.md](./patterns.md) - Repeatable request/reply, pub/sub, and codec workflows
[gotchas.md](./gotchas.md) - Common pitfalls around delivery mode, blocking, and clustering assumptions

## See Also

- [../messaging/README.md](../messaging/README.md) - Broker-backed asynchronous messaging and streaming
- [../dependency-injection/README.md](../dependency-injection/README.md) - Portable CDI events for local observer patterns
- [../web-rest/README.md](../web-rest/README.md) - HTTP endpoints that call into the event bus
- [../tooling/README.md](../tooling/README.md) - Dev mode and CLI workflows while iterating on reactive applications
