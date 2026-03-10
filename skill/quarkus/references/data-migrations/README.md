# Data Migrations Reference

Use this module when the task is about schema evolution in Quarkus with Flyway: migration setup, versioned SQL files, startup migration, baseline and repair, or moving from Hibernate-managed schema creation to managed migrations.

## Overview

Quarkus integrates Flyway as a first-class extension and wires it to the configured datasource.

- Flyway tracks applied migrations in a schema history table.
- Migrations normally live under `src/main/resources/db/migration`.
- Quarkus can run migrations automatically at startup or you can inject `Flyway` and call it directly.
- Named datasources use the same Flyway model with datasource-specific prefixes.

## General guidelines

- Treat migration files as append-only once applied anywhere shared.
- Prefer versioned SQL migrations for schema evolution; review repeatable migrations carefully.
- Keep Hibernate schema generation for prototyping, then switch production-facing environments to Flyway-managed changes.
- Use profiles to separate dev conveniences such as `clean-at-start` from production behavior.

## Scope split

- This module covers schema lifecycle and migration execution.
- For entity mapping and persistence behavior use `data-orm`.
- For Panache APIs use `data-panache`.

## Quick Routing

1. Dependency, migration naming, `Flyway` injection, and customizers -> `api.md`
2. Startup, baseline, validation, repair, and datasource-specific properties -> `configuration.md`
3. Dev-to-prod workflows and rollout patterns -> `patterns.md`
4. Common migration failures and safety rules -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Runtime Flyway entry points and file conventions
[configuration.md](./configuration.md) - High-value Quarkus Flyway settings
[patterns.md](./patterns.md) - Repeatable migration workflows
[gotchas.md](./gotchas.md) - Common pitfalls and recovery guidance

## See Also

- [../data-orm/README.md](../data-orm/README.md) - ORM schema-management context and the handoff to managed migrations
- [../data-panache/README.md](../data-panache/README.md) - Panache applications use the same Flyway strategy underneath
- [../configuration/README.md](../configuration/README.md) - Profiles and environment-specific configuration overrides
