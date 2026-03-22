# Security JWT Reference

Use this module when the task is about verifying bearer JWTs locally with SmallRye JWT, injecting `JsonWebToken`, protecting endpoints with roles, or building/signing/encrypting JWTs inside a Quarkus application.

## Overview

Quarkus SmallRye JWT covers stateless bearer-token verification and JWT creation flows without requiring full OpenID Connect integration.

- Use `quarkus-smallrye-jwt` to verify inbound JWTs and map claims to Quarkus security identities.
- Use `quarkus-smallrye-jwt-build` to create signed, encrypted, or nested signed-then-encrypted tokens.
- Prefer this module when keys, issuer, and audience checks are enough and the application does not need browser login flows or token acquisition.
- Prefer OIDC when you need discovery, authorization code flow, token introspection, provider metadata, or tighter IdP integration.

## JWT vs OIDC

- Choose SmallRye JWT when the service only needs to accept bearer tokens and verify them against configured keys and claims.
- Choose OIDC when Quarkus must talk to an identity provider as a protocol client, for example login redirects, multi-tenancy, refresh tokens, or opaque token support.
- It is common to start with JWT for service-to-service APIs and move to OIDC when identity-provider features become part of the runtime contract.

## Quick Routing

1. Injection APIs, `JWTParser`, and token build APIs -> `api.md`
2. Verification, signing, encryption, and token-source properties -> `configuration.md`
3. Repeatable bearer-token and token-building workflows -> `patterns.md`
4. Common validation, key, and claim pitfalls -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Runtime verification, claim access, and token build APIs
[configuration.md](./configuration.md) - High-value JWT verification and build properties
[patterns.md](./patterns.md) - Repeatable verification and token generation flows
[gotchas.md](./gotchas.md) - Common JWT validation and crypto pitfalls

## See Also

- [../security-core/README.md](../security-core/README.md) - shared Quarkus authentication and authorization concepts
- [../security-oidc/README.md](../security-oidc/README.md) - full OpenID Connect provider integration
