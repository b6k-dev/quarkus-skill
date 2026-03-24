# Web REST Reference

Use this module when the task is about Quarkus REST and HTTP APIs: endpoint mapping, JSON payloads, request/response handling, exception mapping, filters, multipart, and reactive streaming.

## Overview

Quarkus REST (formerly RESTEasy Reactive) is Quarkus' Jakarta REST implementation built on Vert.x with build-time optimization.

- Supports blocking and non-blocking endpoints in the same application.
- Supports JSON with Jackson (`quarkus-rest-jackson`) or JSON-B (`quarkus-rest-jsonb`).
- Includes multipart handling, streaming, content negotiation, and typed responses.
- Shares provider infrastructure with Quarkus REST Client.

## General guidelines

- Prefer concrete return types (`Fruit`, `List<Fruit>`, `RestResponse<Fruit>`) over raw `Response`.
- Keep blocking work off the event-loop thread; use reactive APIs or `@Blocking`.
- Map domain/service exceptions to HTTP with `@ServerExceptionMapper`.
- Move uploaded files to durable storage during request handling.

## Scope split

- This module covers REST and HTTP behavior.
- For server-side templates use `templates`.
- For API contract generation and docs use `openapi`.
- For outbound HTTP clients use `service-communication-rest`.

## Quick Routing

1. Endpoint annotations, params, response APIs, filters -> `api.md`
2. High-value REST/JSON/multipart/compression settings -> `configuration.md`
3. Repeatable implementation workflows -> `patterns.md`
4. Common failures and debugging guidance -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Runtime REST APIs and annotation patterns
[configuration.md](./configuration.md) - Quarkus REST and JSON configuration keys
[patterns.md](./patterns.md) - Repeatable endpoint implementation workflows
[gotchas.md](./gotchas.md) - Common web/REST pitfalls and fixes

## See Also

- [../openapi/README.md](../openapi/README.md) - OpenAPI contract generation and Swagger UI guidance
- [../templates/README.md](../templates/README.md) - Qute templates and server-side rendering
- [../dependency-injection/README.md](../dependency-injection/README.md) - CDI patterns used by REST resources and providers
- [../service-communication-rest/README.md](../service-communication-rest/README.md) - Typed outbound HTTP clients and client-side testing
