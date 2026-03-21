# Service Communication REST Reference

Use this module when a Quarkus application calls another HTTP service through a typed REST client.

## Overview

Quarkus REST Client provides interface-based outbound HTTP calls built on the same reactive REST stack used by Quarkus REST.

- Use it for service-to-service HTTP calls with typed request and response models.
- Prefer `@RegisterRestClient` plus configuration over hand-built `WebTarget` or low-level HTTP clients.
- Keep downstream concerns such as auth headers, timeouts, TLS, and retries close to the client boundary.
- Return concrete payload types or `RestResponse<T>` when status and headers matter.

## Scope split

- This module covers outbound HTTP communication.
- For inbound REST endpoint design and request handling use `web-rest`.
- For asynchronous broker-backed integration use `messaging`.

## Quick Routing

1. Client annotations, return types, headers, providers, and overrides -> `api.md`
2. Base URL, timeouts, redirects, TLS, proxies, and pool tuning -> `configuration.md`
3. Repeatable service-to-service implementation workflows -> `patterns.md`
4. WireMock-based client and integration test workflows -> `testing.md`
5. Common client-side failures and debugging guidance -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Runtime REST client APIs and copy-ready examples
[configuration.md](./configuration.md) - High-value REST client configuration keys and patterns
[patterns.md](./patterns.md) - Repeatable downstream client usage and resilience workflows
[testing.md](./testing.md) - WireMock-based client and integration testing flows
[gotchas.md](./gotchas.md) - Common outbound HTTP pitfalls and fixes

## See Also

- [../web-rest/README.md](../web-rest/README.md) - Server-side endpoint implementation with Quarkus REST
- [../messaging/README.md](../messaging/README.md) - Broker-backed async communication between services
- [../configuration/README.md](../configuration/README.md) - Profile-aware property management for client setup
