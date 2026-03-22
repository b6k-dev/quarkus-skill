# Observability Routing

Use this file to route an observability task to the right Quarkus reference module or starter extension.

## Route by primary question

| Primary question | Module | Typical starting point |
|---|---|---|
| Why is the app noisy, quiet, or hard to diagnose from logs? | `observability-logging` | `quarkus.log.*`, `quarkus-logging-json` |
| Should this instance be restarted or removed from traffic? | `observability-health` | `quarkus-smallrye-health` |
| What are request rates, error rates, latency trends, or resource trends? | `observability-metrics` | `quarkus-micrometer-registry-prometheus` |
| Where is latency or failure happening across services? | `observability-tracing` | `quarkus-opentelemetry` |

## Route by concrete task

| Task | Go to |
|---|---|
| Add JSON logs for containers or log pipeline ingestion | `observability-logging` |
| Add MDC fields or trace/log correlation to log lines | `observability-logging` |
| Implement readiness for a database or broker dependency | `observability-health` |
| Move `/q/health` off the main app listener | `observability-health` plus `shared-concerns.md` |
| Expose Prometheus metrics | `observability-metrics` |
| Add custom counters, timers, or tag rules | `observability-metrics` |
| Send Micrometer metrics over OTLP | `observability-metrics` |
| Configure OTLP tracing export, sampling, or propagation | `observability-tracing` |
| Add custom spans around internal work | `observability-tracing` |
| Migrate from OpenTracing | `observability-tracing` |
| Decide whether to use the management interface | `shared-concerns.md` |
| Bring up a disposable local collector plus dashboards | `shared-concerns.md` |

## Recommended extension starting points

| Need | Start with | Notes |
|---|---|---|
| Distributed tracing with OTLP export | `quarkus-opentelemetry` | Quarkus-native tracing path |
| Micrometer metrics scraped by Prometheus | `quarkus-micrometer-registry-prometheus` | Default metrics path in Quarkus |
| Micrometer metrics exported through OpenTelemetry | `quarkus-micrometer-opentelemetry` | Keep Micrometer instrumentation, unify export |
| Health endpoints and probe APIs | `quarkus-smallrye-health` | Liveness, readiness, startup, groups |
| Structured JSON logs | `quarkus-logging-json` | Use with console, file, syslog, or socket handlers |
| Local all-in-one LGTM stack in dev/test | `quarkus-observability-devservices-lgtm` | Keep in `provided` scope |
