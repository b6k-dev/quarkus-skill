# Observability Metrics Reference

Use this module when the task is about Quarkus application metrics, Micrometer registries, Prometheus exposure, custom meters, binder tuning, metrics endpoint placement, or sending Micrometer metrics through OpenTelemetry.

## Overview

Quarkus metrics guidance should default to Micrometer.

- Use Micrometer for automatic HTTP, JVM, client, messaging, and extension metrics.
- Use Micrometer APIs for custom counters, timers, gauges, and summaries.
- Expose metrics through the built-in metrics endpoint, most commonly for Prometheus scraping.
- Use the Micrometer-to-OpenTelemetry bridge when metrics should join OpenTelemetry traces and logs on OTLP output.
- Treat direct OpenTelemetry metrics as secondary and advanced; use them when you intentionally want OpenTelemetry-native instruments instead of Quarkus' default metrics path.

## General guidelines

- Start with `quarkus-micrometer-registry-prometheus` unless another backend or OTLP unification is the real requirement.
- Prefer low-cardinality tags such as method, outcome, status, region, or tenant tier; avoid user IDs, raw URLs, request IDs, or other unbounded values.
- Keep automatic binders on by default, then trim noisy meters with ignore and match patterns instead of replacing the whole setup.
- Put metrics on the management interface when operations access should differ from public traffic.
- Use the Micrometer-to-OpenTelemetry bridge when you want Micrometer APIs and Quarkus auto-instrumentation, but OTLP as the export path.

## Quick Routing

1. Meter APIs, annotations, binders, customizers, and bridge behavior -> `api.md`
2. High-value Micrometer, Prometheus, management, and OTLP-related properties -> `configuration.md`
3. Repeatable setup, tagging, export, and bridge workflows -> `patterns.md`
4. Common cardinality, endpoint, binder, and bridge pitfalls -> `gotchas.md`

## In This Reference

- [api.md](./api.md) - Core Micrometer APIs, automatic binders, and advanced extension points
- [configuration.md](./configuration.md) - High-value metrics, export, and bridge properties
- [patterns.md](./patterns.md) - Repeatable metrics implementation workflows
- [gotchas.md](./gotchas.md) - Common metrics pitfalls and fixes

## See Also

- [Configuration Reference](../configuration/README.md) - Quarkus config layering, profiles, and build-time vs runtime rules
