# Observability Routing Guide

Use this module when the task starts with observability design choices in Quarkus and you need to route quickly to the right signal-specific reference.

## What this module is for

This module is an entrypoint, not a full implementation reference.

- Start here when the question is "which observability module do I need?"
- Use it to choose between logging, health, metrics, and tracing.
- Use it for the small set of cross-cutting concerns shared across signals: management exposure, OTLP egress, and local LGTM Dev Services.
- Move into the leaf modules once the signal is clear.

## Quick Routing

| If the task is about... | Go to |
|---|---|
| log levels, JSON logs, MDC, handlers, or log shipping | `observability-logging` |
| liveness/readiness/startup probes, Health UI, or health groups | `observability-health` |
| Micrometer, Prometheus, custom meters, or metrics export | `observability-metrics` |
| OpenTelemetry tracing, propagation, sampling, or custom spans | `observability-tracing` |
| choosing signals, management-port placement, or local LGTM setup | this module |

## Signal Selection Heuristic

- Start with `health` when the question is "should traffic reach this service?"
- Start with `logging` when the question is "what happened in this process?"
- Start with `metrics` when the question is "how is behavior changing over time?"
- Start with `tracing` when the question is "where did time go across service boundaries?"
- Use more than one signal only when they answer different questions.

## In This Reference

- [routing.md](./routing.md) - task-to-module routing and extension starting points
- [shared-concerns.md](./shared-concerns.md) - management interface, OTLP egress, and local LGTM guidance

## See Also

- [../observability-logging/README.md](../observability-logging/README.md) - Logging reference
- [../observability-health/README.md](../observability-health/README.md) - Health reference
- [../observability-metrics/README.md](../observability-metrics/README.md) - Metrics reference
- [../observability-tracing/README.md](../observability-tracing/README.md) - Tracing reference
