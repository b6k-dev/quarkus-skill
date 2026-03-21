# Service Communication Reference

Use this module when a Quarkus service must call another service and you first need to choose the right communication style.

## Overview

This is a routing module for synchronous inter-service communication.

- Use it to decide between typed HTTP clients and gRPC clients.
- Keep transport choice explicit because REST and gRPC differ in contracts, tooling, observability, and failure modes.
- Route asynchronous cross-service delivery to `messaging` instead of forcing request/response semantics.

## Boundary-First Chooser

```text
Do you need asynchronous delivery, buffering, replay, or broker-managed fan-out?
├── YES -> messaging
└── NO
      │
      Do both sides share a protobuf contract and need HTTP/2 RPC or streaming?
      ├── YES -> service-communication-grpc
      └── NO  -> service-communication-rest
```

## When REST Is Usually the Better Fit

- The downstream service already exposes HTTP and JSON.
- You need simpler interoperability across teams, languages, gateways, or edge systems.
- cURL-level debugging, headers, status codes, and browser-friendly semantics matter.
- You want Quarkus REST Client with typed interfaces but standard HTTP behavior.

## When gRPC Is Usually the Better Fit

- Both sides are internal services that can share `.proto` contracts.
- Low-latency request/response and streaming matter.
- You want generated strongly typed clients and service interfaces.
- HTTP/2 transport and protobuf encoding are acceptable operational constraints.

## What This Covers

- Transport selection between REST and gRPC for service-to-service calls.
- Boundary guidance for when not to use synchronous RPC at all.
- Cross-links to the detailed transport-specific references.

## Quick Routing

1. REST client interfaces, HTTP config, retries, and testing -> `../service-communication-rest/README.md`
2. Proto-first RPC, streaming, and generated stubs -> `../service-communication-grpc/README.md`
3. Async cross-service events and durable delivery -> `../messaging/README.md`

## In This Reference

[patterns.md](./patterns.md) - Repeatable decision patterns for common service boundaries
[gotchas.md](./gotchas.md) - Common transport-selection mistakes and fixes

## See Also

- [../service-communication-rest/README.md](../service-communication-rest/README.md) - Typed outbound HTTP clients with Quarkus REST Client
- [../service-communication-grpc/README.md](../service-communication-grpc/README.md) - Internal RPC with protobuf and HTTP/2
- [../messaging/README.md](../messaging/README.md) - Broker-backed asynchronous communication across services
- [../web-rest/README.md](../web-rest/README.md) - Server-side HTTP endpoint implementation
- [../openapi/README.md](../openapi/README.md) - REST contract generation and API documentation
