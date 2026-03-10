# Data ORM Reference

Use this module for standard Quarkus Hibernate ORM work: entity mapping, `EntityManager` or `Session` usage, transactions, schema generation basics, and database dialect/version configuration.

## Overview

This module covers plain Hibernate ORM with Quarkus configuration in `application.properties`.

- Use it when the task is about `@Entity`, `@Id`, `EntityManager`, `Session`, `@Transactional`, or schema-management defaults.
- Prefer this module when the question is Quarkus + JPA and there is no explicit Panache, Flyway-first, or advanced persistence-unit concern.
- Quarkus usually auto-configures the persistence layer from the datasource, so explicit ORM settings are often minimal.

## What This Covers

- Adding `quarkus-hibernate-orm` with a JDBC driver.
- Basic entity mapping and persistence operations.
- Transaction boundaries for reads and writes.
- Dev/test/prod schema-management choices.
- Dialect and database version guidance.
- `import.sql` and SQL seed loading basics.

## What This Does Not Cover

- Panache APIs and active-record/repository helpers.
- Flyway-led schema migration workflows.
- Multiple persistence units, multitenancy, caching, XML mapping, or SPI extensions.

## Quick Routing

1. Dependencies, entities, `EntityManager`, `Session`, transactions, queries -> `api.md`
2. Key ORM properties and profile defaults -> `configuration.md`
3. Repeatable service and environment setups -> `patterns.md`
4. Common persistence mistakes and fixes -> `gotchas.md`

## In This Reference
[api.md](./api.md) - Runtime ORM entry points and copy-ready examples
[configuration.md](./configuration.md) - High-value Hibernate ORM properties in Quarkus
[patterns.md](./patterns.md) - Repeatable CRUD, dev-mode, and production-safe setups
[gotchas.md](./gotchas.md) - Common Hibernate ORM pitfalls in Quarkus

## See Also

- [../data-panache/README.md](../data-panache/README.md) - Simpler CRUD and query helpers on top of the same ORM foundation
- [../data-migrations/README.md](../data-migrations/README.md) - Managed schema evolution once `drop-and-create` stops being enough
- [../dependency-injection/README.md](../dependency-injection/README.md) - CDI injection and bean-boundary guidance for ORM services
