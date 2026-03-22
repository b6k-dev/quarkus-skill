# Observability Tracing Reference

Use this module when the task is about Quarkus distributed tracing with OpenTelemetry: default server/client spans, OTLP export, sampling, propagation, manual spans, instrumentation toggles, and migration from legacy OpenTracing.

## Overview

Quarkus tracing is powered by the `quarkus-opentelemetry` extension and is enabled by default once the extension is present.

- Quarkus instruments supported extensions directly; the generic OpenTelemetry Java agent is usually unnecessary and not recommended.
- OTLP export is the Quarkus-native default, using gRPC to `http://localhost:4317` unless configured otherwise.
- W3C `tracecontext` and `baggage` propagation are enabled by default.
- Metrics and logs are out of scope here except where they affect trace export or log correlation.

## General guidelines

- Start with Quarkus automatic instrumentation before adding manual spans.
- Prefer OTLP exporter configuration over vendor-specific tracer settings.
- Do not add `@WithSpan` to JAX-RS endpoints already traced automatically.
- Use sampling and URI suppression to control volume before disabling tracing entirely.
- Treat OpenTracing shim usage as a migration aid, not a long-term design.

## Quick Routing

1. Tracer injection, `@WithSpan`, manual spans, propagation APIs -> `api.md`
2. High-value tracing and OTLP properties -> `configuration.md`
3. Repeatable setup and migration workflows -> `patterns.md`
4. Common missing-span, duplicate-span, and migration failures -> `gotchas.md`

## In This Reference

- [api.md](./api.md) - Runtime tracing APIs and copy-ready examples
- [configuration.md](./configuration.md) - High-value OpenTelemetry tracing properties and defaults
- [patterns.md](./patterns.md) - Repeatable tracing setup, customization, and migration workflows
- [gotchas.md](./gotchas.md) - Common tracing pitfalls and fixes

## See Also

- [../web-rest/README.md](../web-rest/README.md) - REST endpoint behavior that receives automatic server spans
- [../messaging/README.md](../messaging/README.md) - Cross-service message handling and context propagation patterns
- [../security-core/README.md](../security-core/README.md) - Security identity behavior relevant to end-user span attributes and security events
