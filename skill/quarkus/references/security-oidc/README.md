# Security OIDC Reference

Use this module when the task is about protecting Quarkus HTTP endpoints with OpenID Connect: bearer-token service APIs, authorization-code-flow web apps, token/session handling, and role mapping.

## Overview

Quarkus OIDC supports two main protection models with the same extension:

- `service` applications validate bearer access tokens sent by clients.
- `web-app` applications redirect users to the provider and keep an authenticated session.
- `hybrid` applications can support browser login and bearer-token API access together.
- Roles can come from token claims, `UserInfo`, or custom identity augmentation.

## General guidelines

- Choose the application type first: `service`, `web-app`, or `hybrid`.
- Keep discovery enabled unless the provider metadata is incomplete or unavailable.
- Verify issuer and audience intentionally instead of relying on provider defaults alone.
- Keep role extraction explicit when claims are not in standard locations.

## Quick Routing

1. Bearer-token protection, code-flow injection points, auth APIs -> `api.md`
2. High-value provider, token, role, and session settings -> `configuration.md`
3. Common implementation workflows -> `patterns.md`
4. Typical OIDC failures and debugging guidance -> `gotchas.md`

## Decision Split

- Protect a REST/service endpoint called with `Authorization: Bearer ...` -> use `service` mode
- Protect HTML/web endpoints with login redirects and a session cookie -> use `web-app` mode
- Protect both browser pages and token-authenticated APIs in one app -> use `hybrid` mode

## In This Reference

[api.md](./api.md) - Runtime OIDC entry points and auth patterns
[configuration.md](./configuration.md) - High-value OIDC configuration keys
[patterns.md](./patterns.md) - Repeatable bearer and code-flow implementation workflows
[gotchas.md](./gotchas.md) - Common OIDC pitfalls and fixes

## See Also

- [`../web-rest/README.md`](../web-rest/README.md) - HTTP endpoint behavior and Jakarta REST patterns
- [`../configuration/README.md`](../configuration/README.md) - Quarkus configuration and profile usage
- [`../service-communication/README.md`](../service-communication/README.md) - Calling downstream services after authentication
- [`../templates/README.md`](../templates/README.md) - Server-rendered web pages often paired with code flow
