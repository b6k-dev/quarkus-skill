# Observability Shared Concerns

Use this file for the few observability topics that cut across logging, health, metrics, and tracing.

## Management interface

Use the management interface when operational endpoints should not share the main listener, host, port, or security policy.

High-value properties:

| Property | Default | Use when |
|---|---|---|
| `quarkus.management.enabled` | `false` | Operational endpoints should move off the main HTTP listener |
| `quarkus.management.host` | dev/test `localhost`, prod `0.0.0.0` | The management listener must bind differently |
| `quarkus.management.port` | `9000` | Operations traffic needs a dedicated port |
| `quarkus.management.root-path` | `/q` | Management endpoints need a different base path |
| `quarkus.management.tls-configuration-name` | main TLS config or none | Management traffic needs separate TLS policy |

Rules that matter:

- `quarkus.management.enabled` is build-time sensitive.
- `quarkus.http.root-path` does not control management routes.
- Health and metrics modules contain the signal-specific placement details after the listener is enabled.

## OTLP as shared egress

When telemetry leaves the service, prefer a single OTLP path where practical.

Common properties:

| Property | Default | Use when |
|---|---|---|
| `quarkus.otel.exporter.otlp.endpoint` | `http://localhost:4317/` or auto-wired locally | OpenTelemetry signals should go to a collector |
| `quarkus.otel.exporter.otlp.protocol` | `grpc` | OTLP must use `grpc` or `http/protobuf` |
| `quarkus.micrometer.export.otlp.url` | not set unless configured or auto-wired | Micrometer OTLP registry should export to a collector |

Use `observability-tracing` for tracing-specific OTLP details and `observability-metrics` for Micrometer bridge/export details.

## Local LGTM Dev Services

Use local LGTM Dev Services when you want disposable dashboards and a collector during development or tests.

Dependency:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-observability-devservices-lgtm</artifactId>
    <scope>provided</scope>
</dependency>
```

Useful properties:

| Property | Default | Use when |
|---|---|---|
| `quarkus.observability.enabled` | `true` | Dev Services-based observability should be disabled |
| `quarkus.observability.dev-resources` | `false` | You want dev resources instead of full Dev Services behavior |
| `quarkus.observability.lgtm.grafana-port` | random host port | Grafana should use a stable local port |
| `quarkus.observability.lgtm.otel-grpc-port` | random host port | OTLP gRPC should use a stable local port |
| `quarkus.observability.lgtm.otel-http-port` | random host port | OTLP HTTP should use a stable local port |

Guidelines:

- Keep LGTM dependencies out of production runtime artifacts.
- Prefer random ports unless humans or scripts need stable endpoints.
- Keep local-stack settings in dev or test profiles.

## Common mistakes

| Symptom | Likely cause | Fix |
|---|---|---|
| `/q/health` or `/q/metrics` still uses the main app port | The management interface is not enabled | Set `quarkus.management.enabled=true` and rebuild if needed |
| Production points at a local collector | Dev-only LGTM or OTLP config leaked into shared properties | Move local settings into `%dev` or `%test` profiles |
| Grafana URL changes every run | LGTM uses random ports by default | Read startup logs or pin ports only when stability is needed |
| Packaged artifacts include Testcontainers libraries | LGTM dependency is not `provided` | Mark the dependency with `provided` scope |
