# CDI Events Reference

Use this module when the task is about in-process domain events with Jakarta CDI in Quarkus: `Event<T>`, `@Observes`, `@ObservesAsync`, transactional observers, and deciding when CDI events are enough.

## Overview

CDI events are the lightest eventing model in Quarkus.

- Use them when producers and observers live in the same service.
- Prefer them for portable, type-safe in-process choreography.
- Use transactional observers to align side effects with commit outcome.
- Move to `vertx-event-bus` for local request/reply or event-loop driven messaging.
- Move to `messaging` when events must cross service or process boundaries.

## Decision Fit

```text
Is the event crossing a service/process boundary?
├── YES -> messaging
└── NO (in-process only)
      │
      Do you need clustering or non-blocking event loop behavior?
      ├── YES -> vertx-event-bus
      └── NO
            │
            Do you want portability and type safety?
            └── YES -> cdi-events
```

## What This Covers

- `Event<T>` injection and event firing.
- Synchronous and asynchronous observers.
- Transaction phases such as `AFTER_SUCCESS`.
- Error handling expectations and local retry considerations.
- Migration guidance from local observers to broker-backed messaging.

## What This Does Not Cover

- Broker-backed channels and connectors.
- Vert.x Event Bus addresses, request/reply, and codecs.
- General CDI bean discovery and qualifier design beyond what events need.

## Quick Routing

1. `Event<T>`, `@Observes`, `@ObservesAsync`, transaction phases -> `api.md`
2. Relevant ArC and test settings -> `configuration.md`
3. Local event choreography and escalation patterns -> `patterns.md`
4. Common observer and transaction mistakes -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Core CDI event APIs and compact examples
[configuration.md](./configuration.md) - High-value settings that affect observers and tests
[patterns.md](./patterns.md) - Repeatable local eventing workflows
[gotchas.md](./gotchas.md) - Common pitfalls around async delivery and transaction phases

## See Also

- [../dependency-injection/README.md](../dependency-injection/README.md) - Broader CDI/ArC injection, scopes, qualifiers, and bean model
- [../vertx-event-bus/README.md](../vertx-event-bus/README.md) - Local async request/reply and event-loop friendly messaging
- [../messaging/README.md](../messaging/README.md) - Broker-backed asynchronous messaging across service boundaries
