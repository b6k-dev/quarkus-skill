# Observability Health Reference

Use this module when the task is about Quarkus health probes, SmallRye Health endpoints, Health UI, health groups, extension-provided checks, or exposing probes on the management interface.

## Overview

Quarkus health support is provided by the `quarkus-smallrye-health` extension.

- Exposes built-in liveness, readiness, startup, and aggregate health endpoints.
- Lets applications contribute custom probe logic with CDI beans.
- Registers health checks from supporting extensions such as datasources and messaging components.
- Can publish probes and Health UI on the management interface instead of the main HTTP server.
- Supports grouped health views when a platform or operator needs narrower probe surfaces.

## General guidelines

- Keep liveness checks shallow; they should answer whether the process should be restarted, not whether dependencies are healthy.
- Put dependency reachability in readiness checks so traffic drains before the container is restarted.
- Use startup checks only when readiness is not enough to protect a slow bootstrap path.
- Prefer small, deterministic probe code; avoid blocking work, remote fan-out, or expensive queries.
- Leave extension health checks enabled unless you have a clear replacement strategy.
- Move probes to the management interface when they should not share the public app port.

## Quick Routing

1. Built-in endpoints, health check APIs, groups, async checks, and Health UI entry points -> `api.md`
2. High-value health and management placement properties -> `configuration.md`
3. Repeatable probe design, dependency checks, grouping, and management-interface workflows -> `patterns.md`
4. Common probe design, path, startup, and management placement pitfalls -> `gotchas.md`

## In This Reference

- [api.md](./api.md) - Core health APIs, endpoints, groups, and UI behavior
- [configuration.md](./configuration.md) - High-value health and management-interface settings
- [patterns.md](./patterns.md) - Repeatable probe design and deployment workflows
- [gotchas.md](./gotchas.md) - Common health probe pitfalls and fixes

## See Also

- [Configuration Reference](../configuration/README.md) - Quarkus configuration patterns, profiles, and property placement
