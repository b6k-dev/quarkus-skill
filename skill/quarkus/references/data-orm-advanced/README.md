# Advanced ORM Reference

Use this module when Hibernate ORM work in Quarkus goes beyond normal entity mapping and transactions.

## Prerequisite

Read `../data-orm/README.md` first for the baseline ORM model, default datasource behavior, and day-1 entity usage.

## Overview

This module is intentionally opt-in and advanced.

- Focus on architecture choices such as multiple persistence units, multitenancy, runtime activation, and custom ORM extension points.
- Cover migration-oriented features such as `persistence.xml` and XML mappings.
- Highlight operational concerns such as offline startup, cache tuning, metrics, and Quarkus-specific limitations.

## Use This Module For

- `@PersistenceUnit` injection, package attachment, and named persistence units
- `@PersistenceUnitExtension` customizations such as interceptors, statement inspectors, function/type contributors, and format mappers
- Runtime activation or deactivation of persistence units and datasource switching
- Schema, database, or discriminator multitenancy
- `persistence.xml`, `orm.xml`, or `hbm.xml` integration
- Offline startup when the database is not reachable during application boot
- Second-level cache, Envers, Spatial, metrics, Jakarta Data, and static metamodel setup

## Do Not Start Here For

- Basic entity mapping, transactions, and single-datasource ORM setup
- Panache-first CRUD usage
- Migration tooling except where it is required by multitenancy or offline startup

## Quick Routing

1. Injection points, tenant resolvers, extension SPI types, Jakarta Data entry points -> `api.md`
2. Named persistence unit keys, active flags, multitenancy, cache, offline startup -> `configuration.md`
3. Repeatable multi-PU, multitenancy, and extension workflows -> `patterns.md`
4. Quarkus-specific pitfalls, inactive beans, cache limits, and portability constraints -> `gotchas.md`

## In This Reference
[api.md](./api.md) - Advanced runtime APIs, qualifiers, and extension points
[configuration.md](./configuration.md) - Selected high-value advanced Hibernate ORM properties
[patterns.md](./patterns.md) - Repeatable multi-unit, multitenant, and extension integration patterns
[gotchas.md](./gotchas.md) - Common advanced ORM failures and Quarkus-specific limits

## See Also

- [../data-orm/README.md](../data-orm/README.md) - Base Hibernate ORM concepts and defaults
- [../data-migrations/README.md](../data-migrations/README.md) - Flyway companion for startup and tenant schema control
- [../dependency-injection/README.md](../dependency-injection/README.md) - CDI qualifiers, producers, and dynamic lookup patterns
