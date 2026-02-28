# Dependency Injection Reference

Use this module when the task is about Quarkus CDI/ArC dependency injection: bean discovery, scopes, qualifiers, producers, lifecycle, interception, events, and ArC settings.

## Overview

Quarkus dependency injection is powered by ArC, a build-time optimized CDI implementation.

- Based on Jakarta CDI (CDI Lite) with selected CDI Full features.
- Optimized for fast startup and low memory via build-time analysis.
- Focuses on type-safe injection, explicit scopes, and fail-fast resolution.
- Includes Quarkus-specific capabilities for defaults, conditional beans, and lookup ergonomics.

## General guidelines

- Default to `@ApplicationScoped` unless a narrower scope is required.
- Prefer constructor injection for required dependencies and store them in private final fields.
- Prefer type-safe qualifiers; use `@Identifier` over `@Named` for internal wiring.
- Avoid private injection points, observers, and producer methods to reduce reflection usage in native builds.
- Use programmatic lookup (`Instance<T>`) only when static injection is not sufficient.

## Quick Routing

1. Injection errors (`UnsatisfiedResolutionException`, `AmbiguousResolutionException`) -> `gotchas.md` + `api.md`
2. Bean model, scopes, qualifiers, lifecycle callbacks -> `api.md`
3. ArC and bean discovery properties -> `configuration.md`
4. Repeatable implementation patterns -> `patterns.md`

## In This Reference

[api.md](./api.md) - CDI/ArC runtime APIs and dependency injection concepts
[configuration.md](./configuration.md) - ArC and bean discovery configuration
[patterns.md](./patterns.md) - Repeatable dependency injection and lifecycle workflows
[gotchas.md](./gotchas.md) - Common CDI/ArC pitfalls and fixes
