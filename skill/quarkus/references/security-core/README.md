# Security Core Reference

Use this module when the task is about core Quarkus security behavior: choosing an authentication mechanism, building or augmenting `SecurityIdentity`, securing endpoints with annotations or HTTP permissions, and understanding how authn/authz fit together.

## Overview

Quarkus Security provides the common framework behind authentication and authorization.

- Authentication mechanisms turn request credentials into a `SecurityIdentity`.
- Identity providers verify credentials and build the identity.
- Authorization can happen with annotations, HTTP path policies, or custom policies.
- The same model works across REST endpoints and CDI beans.

## General guidelines

- Start with standard annotations for endpoint and bean authorization.
- Use HTTP permission config for path-wide rules, mechanism selection, or static/public assets.
- Keep role and permission mapping close to the boundary where identities are created or augmented.
- Treat embedded users and plaintext passwords as test-only setup.
- Prefer the smallest mechanism that fits: Basic/Form for simple username-password flows, mTLS for service identity, OIDC/JWT for token-based systems.

## Quick Routing

1. Security annotations, `SecurityIdentity`, custom policy hooks -> `api.md`
2. High-value auth and authorization properties -> `configuration.md`
3. Repeatable security setup workflows -> `patterns.md`
4. Common authorization and path-matching failures -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Core security APIs, annotations, and minimal examples
[configuration.md](./configuration.md) - High-value auth, RBAC, and mechanism configuration
[patterns.md](./patterns.md) - Repeatable authentication and authorization workflows
[gotchas.md](./gotchas.md) - Common security pitfalls and fixes

## See Also

- [../web-rest/README.md](../web-rest/README.md) - REST endpoint mapping, filters, and request handling details
- [../dependency-injection/README.md](../dependency-injection/README.md) - CDI patterns used by custom providers, augmentors, and policies
- [../configuration/README.md](../configuration/README.md) - General Quarkus config source, profile, and override behavior
