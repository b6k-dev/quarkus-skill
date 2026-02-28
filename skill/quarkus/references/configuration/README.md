# Configuration Reference

Use this module when the task is about Quarkus application configuration: property sources, profiles, typed mappings, expressions, and build-time vs runtime config behavior.

## Overview

Quarkus configuration is powered by SmallRye Config (MicroProfile Config implementation).

- The same system handles both application keys and Quarkus extension keys.
- Config is resolved from multiple sources with deterministic priority.
- You can inject single keys (`@ConfigProperty`) or grouped typed models (`@ConfigMapping`).
- Validation and startup checks help catch bad configuration early.

## General guidelines

- Keep application properties outside the reserved `quarkus.` prefix.
- Prefer `@ConfigMapping` for grouped settings; use `@ConfigProperty` for one-off values.
- Use profiles (`dev`, `test`, `prod`, custom) for environment-specific behavior.
- Keep secrets out of VCS; use environment variables and local `.env` files.
- Rebuild when changing build-time-fixed properties.

## Quick Routing

1. Source precedence, profile order, external locations -> `configuration.md`
2. Injection APIs (`@ConfigProperty`, `@ConfigMapping`, programmatic access) -> `api.md`
3. Repeatable implementation workflows -> `patterns.md`
4. Startup failures and confusing overrides -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Runtime configuration APIs and injection patterns
[configuration.md](./configuration.md) - High-value config keys, source ordering, and profile behavior
[patterns.md](./patterns.md) - Repeatable configuration workflows
[gotchas.md](./gotchas.md) - Common configuration pitfalls and fixes
