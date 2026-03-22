# Security Identity Providers Reference

Use this module when Quarkus needs a built-in username/password identity store backed by your data: JPA entities, direct JDBC queries, or LDAP.

## Overview

This module covers Quarkus-supported identity providers for application-managed username/password authentication.

- Use JPA when user records live in your domain model and you want entity-level mapping.
- Use JDBC when the schema already exists and direct SQL is simpler than ORM.
- Use LDAP when identities and group membership are owned by a directory server.
- These providers supply `SecurityIdentity` data for username/password mechanisms such as Basic auth and form auth.

## When to choose each provider

- JPA: best when the application already uses Hibernate ORM and user storage is part of the same relational model.
- JDBC: best when you need to authenticate against an existing table layout without introducing ORM entities.
- LDAP: best when credentials and roles must stay in an external corporate directory.

## What This Covers

- Extension entry points for JPA, JDBC, and LDAP identity stores.
- Minimal entity, SQL, and LDAP mapping examples.
- High-value provider-specific configuration.
- Common setup patterns and tradeoffs between the three options.
- Frequent pitfalls around password mapping, role mapping, caching, and schema shape.

## Quick Routing

1. Minimal dependency and setup examples -> `api.md`
2. High-value properties by provider -> `configuration.md`
3. Provider selection and repeatable setups -> `patterns.md`
4. Common provider-specific mistakes -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Extension entry points and minimal JPA, JDBC, LDAP examples
[configuration.md](./configuration.md) - High-value provider settings and how to read them
[patterns.md](./patterns.md) - When to choose each store and repeatable setup patterns
[gotchas.md](./gotchas.md) - Common identity-store pitfalls and fixes

## See Also

- [`../security-core/README.md`](../security-core/README.md) - Core authentication mechanisms and authorization layers that consume these identity stores
- [`../data-orm/README.md`](../data-orm/README.md) - JPA and Hibernate ORM basics behind the JPA identity provider
- [`../data-panache/README.md`](../data-panache/README.md) - Panache style for JPA-backed user entities
