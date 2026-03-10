# Data Panache Reference

Use this module when the task is about Quarkus Panache on top of Hibernate ORM: `PanacheEntity`, `PanacheRepository`, active record, repository-style data access, simplified HQL, paging, projections, or Panache-specific mocking.

## Overview

Panache reduces common ORM boilerplate while still using Hibernate ORM underneath.

- Use `PanacheEntity` for the active record style with built-in `id` and static query helpers.
- Use `PanacheEntityBase` when you want Panache helpers but need your own ID mapping.
- Use `PanacheRepository<T>` or `PanacheRepositoryBase<T, ID>` when you prefer injected repositories over static entity methods.
- Use Panache query helpers for common CRUD, simplified HQL fragments, sorting, paging, projections, and locking.

## When Panache is a good fit

- Most persistence work is straightforward CRUD or filtered lookup.
- You want less repository and DAO boilerplate.
- You want query helpers close to the entity or repository code.
- You still want full Hibernate ORM behavior when needed.

## Scope boundaries

- This module covers the Panache programming model and Panache-specific APIs.
- It does not re-explain basic JPA mapping, datasource setup, or advanced persistence unit design in depth.
- Panache still depends on normal Hibernate ORM concepts: entities, transactions, flush, locking, and HQL.

## Quick Routing

1. Core Panache types and query helpers -> `api.md`
2. Config inherited from Hibernate ORM and datasource setup -> `configuration.md`
3. CRUD, REST, projection, and testing workflows -> `patterns.md`
4. Common Panache pitfalls -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Panache types, query helpers, projections, paging, flushing, locking
[configuration.md](./configuration.md) - High-value Hibernate ORM settings that affect Panache
[patterns.md](./patterns.md) - Repeatable CRUD, REST, projection, and test workflows
[gotchas.md](./gotchas.md) - Panache-specific pitfalls and limits

## See Also

- [`../data-orm/README.md`](../data-orm/README.md) - Base Hibernate ORM concepts Panache builds on
- [`../dependency-injection/README.md`](../dependency-injection/README.md) - CDI scopes and injection patterns for repositories
