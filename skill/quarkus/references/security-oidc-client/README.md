# Security OIDC Client Reference

Use this module when a Quarkus application must acquire, refresh, exchange, or propagate outbound OAuth2/OIDC access tokens for downstream service calls.

## Overview

This module covers Quarkus OIDC client support for service-to-service communication and user-token forwarding.

- Use it for `quarkus-oidc-client`, REST client OIDC filters, token propagation filters, and named outbound clients.
- Prefer it when the main question is how a service gets or forwards an access token before calling another service.
- Keep token acquisition and propagation at the outbound boundary instead of spreading header-building logic through business code.
- Treat direct bearer propagation across multiple hops as a deliberate security decision; token exchange is often the safer fit.

## What This Covers

- Outbound token acquisition with `OidcClient`, `Tokens`, and client filters.
- Refresh behavior, preemptive refresh, and refresh-on-`401` handling.
- Named OIDC clients for different downstreams, grants, or credentials.
- Incoming-token propagation and exchange before calling another service.
- Service-to-service patterns for REST clients that need bearer tokens.

## Quick Routing

1. Extensions, annotations, injected types, and copy-ready examples -> `api.md`
2. OIDC client properties, named-client config, refresh, and propagation flags -> `configuration.md`
3. Repeatable service-to-service token workflows -> `patterns.md`
4. Common propagation and refresh mistakes -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Runtime entry points and minimal examples for outbound token work
[configuration.md](./configuration.md) - High-value OIDC client and token propagation properties
[patterns.md](./patterns.md) - Repeatable service-to-service token acquisition and forwarding patterns
[gotchas.md](./gotchas.md) - Common outbound auth mistakes and safer fixes

## See Also

- [../security-oidc/README.md](../security-oidc/README.md) - Inbound OIDC protection for the endpoints that trigger downstream calls
- [../service-communication/README.md](../service-communication/README.md) - Choose the right synchronous service boundary first
- [../service-communication-rest/README.md](../service-communication-rest/README.md) - Typed outbound HTTP clients that commonly carry these tokens
- [../web-rest/README.md](../web-rest/README.md) - Inbound REST endpoints that trigger downstream calls
- [../configuration/README.md](../configuration/README.md) - Profile-aware property management for client credentials and URLs
