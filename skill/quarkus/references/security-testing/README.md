# Security Testing Reference

Use this module when the task is about testing secured Quarkus applications: shortcut identities with `@TestSecurity`, bearer token tests, OIDC or Keycloak dev services, and auth-focused integration-test workflows.

## Overview

Quarkus supports two complementary security-testing styles:

- In-process security context tests with `quarkus-test-security` and `@TestSecurity`.
- HTTP-level tests that send real bearer tokens to secured endpoints.
- OIDC and Keycloak Dev Services for realistic local and CI auth flows without hand-managed IdP setup.
- Profile-aware fallback auth setups for tests that should avoid production-only identity providers.

## General guidelines

- Use `@TestSecurity` for fast JVM tests that exercise authorization rules inside `@QuarkusTest`.
- Use real HTTP tokens for integration tests, token claim mapping, and end-to-end auth behavior.
- Prefer Dev Services for Keycloak when testing OIDC-protected services against a realistic provider.
- Keep realm files, test users, and client config under `%test` to avoid leaking test auth into dev or prod.

## Scope split

- This module covers testing secured applications.
- For general Quarkus test structure use `testing`.

## Quick Routing

1. `@TestSecurity`, test tokens, and copy-ready snippets -> `api.md`
2. High-value `%test`, OIDC, and Dev Services properties -> `configuration.md`
3. Repeatable secured-endpoint and integration-test workflows -> `patterns.md`
4. Common auth-testing failures and fixes -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Runtime testing APIs and example-first security test snippets
[configuration.md](./configuration.md) - High-value test security, OIDC, and Dev Services properties
[patterns.md](./patterns.md) - Repeatable workflows for secured endpoint and integration testing
[gotchas.md](./gotchas.md) - Common security testing pitfalls and fixes

## See Also

- [../security-core/README.md](../security-core/README.md) - Core Quarkus security concepts, identities, and authorization
- [../security-oidc/README.md](../security-oidc/README.md) - OIDC service and web-app runtime configuration
- [../security-jwt/README.md](../security-jwt/README.md) - JWT claims, principal mapping, and token generation details
