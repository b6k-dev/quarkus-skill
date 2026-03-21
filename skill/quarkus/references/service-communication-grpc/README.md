# gRPC Service Communication Reference

Use this module when two services need request/response RPC over HTTP/2 with a shared protobuf contract, low latency, and strongly typed clients.

## Overview

Quarkus gRPC supports both service implementation and service consumption from the same `quarkus-grpc` extension.

- Use it for internal service-to-service communication where both sides can share `.proto` contracts.
- Prefer proto-first design: define messages and RPC methods first, then let Quarkus generate service and client types.
- Default to Mutiny-based generated interfaces and stubs; use blocking stubs only at clearly blocking boundaries.
- Keep transport details such as endpoints, TLS, deadlines, and discovery in configuration instead of application code.

## Boundary-First Chooser

```text
Do services need synchronous request/response semantics?
├── YES
│     │
│     Do both sides share a typed protobuf contract and benefit from HTTP/2 streaming?
│     ├── YES -> Quarkus gRPC
│     └── NO  -> REST client/server patterns
└── NO -> Broker-backed messaging
```

Use this module for the first `YES -> Quarkus gRPC` branch.

## What This Covers

- Proto-first workflow, code generation, and generated service/client types.
- `@GrpcService`, `@GrpcClient`, Mutiny interfaces, generated stubs, and `StreamObserver` interop.
- Unary, server-streaming, client-streaming, and bidirectional streaming RPCs.
- High-value server and client configuration for ports, TLS, deadlines, metadata, name resolution, and message sizing.
- Testing and local development patterns for gRPC services.

## Quick Routing

1. Generated APIs, annotations, stubs, streaming, metadata, and deadlines -> `api.md`
2. Server and client properties -> `configuration.md`
3. End-to-end implementation and testing workflows -> `patterns.md`
4. Common contract, threading, and transport mistakes -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Core gRPC annotations, generated types, and compact examples
[configuration.md](./configuration.md) - High-value gRPC server, client, TLS, and discovery properties
[patterns.md](./patterns.md) - Repeatable service implementation, client usage, streaming, and test workflows
[gotchas.md](./gotchas.md) - Common pitfalls around proto generation, naming, threading, and config

## See Also

- [../web-rest/README.md](../web-rest/README.md) - HTTP and JSON APIs for browser-facing or REST-style integration
- [../messaging/README.md](../messaging/README.md) - Asynchronous cross-service delivery through brokers
- [../configuration/README.md](../configuration/README.md) - Profiles and layered properties for client targets, TLS, and environment overrides
