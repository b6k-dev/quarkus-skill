# Security WebAuthn Reference

Use this module when the task is about Quarkus WebAuthn or passkeys: browser registration/login ceremonies, `WebAuthnUserProvider`, credential persistence, built-in WebAuthn endpoints, or custom passkey flows.

## Overview

Quarkus WebAuthn adds passwordless authentication based on public-key credentials tied to a relying party origin.

- Use it for passkey registration and login backed by platform authenticators or roaming security keys.
- Expect a two-step ceremony for both registration and login: challenge first, signed response second.
- Plan durable credential storage on the server side; Quarkus validates assertions but does not own your user model.
- The default browser flow is cookie-based and origin-sensitive, so hostnames, HTTPS, and proxy behavior matter.

## What This Covers

- Adding `quarkus-security-webauthn` and exposing a `WebAuthnUserProvider`.
- Built-in `/q/webauthn/*` challenge, register, login, and logout endpoints.
- The bundled `/q/webauthn/webauthn.js` helper for browser ceremonies.
- Minimal persisted credential fields and username-to-credential mapping expectations.
- Common custom-endpoint entry points through `WebAuthnSecurity`.

## Quick Routing

1. Dependencies, endpoints, provider APIs, and browser helpers -> `api.md`
2. High-value WebAuthn and cookie properties -> `configuration.md`
3. Repeatable registration/login/storage workflows -> `patterns.md`
4. Common passkey and deployment failures -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Runtime WebAuthn entry points and request flow
[configuration.md](./configuration.md) - High-value WebAuthn and session settings
[patterns.md](./patterns.md) - Repeatable registration, login, and storage setups
[gotchas.md](./gotchas.md) - Common operational and security pitfalls

## See Also

- [../security-core/README.md](../security-core/README.md) - Core Quarkus authentication and authorization building blocks
- [../templates/README.md](../templates/README.md) - Server-rendered pages often paired with passkey registration and login flows
