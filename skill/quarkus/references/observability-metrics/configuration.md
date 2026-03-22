# Quarkus Observability Metrics Configuration Reference

Use this file for corrected, high-value Micrometer, export, management, and bridge-related configuration keys.

## Rules that matter first

- Default to Micrometer properties first; reach for direct OpenTelemetry metrics properties only when you intentionally use OpenTelemetry-native metrics.
- `quarkus.management.enabled` is build-time. Rebuild packaged applications after changing metrics interface placement.
- Metrics endpoints are relative to the non-application root by default; use an absolute path when you need an exact public URL.
- The Micrometer-to-OpenTelemetry bridge makes OpenTelemetry exporter properties relevant even though the application still uses Micrometer APIs.

## Core Micrometer activation and binder control

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.micrometer.enabled` | `true` | Micrometer metrics must be disabled entirely |
| `quarkus.micrometer.registry-enabled-default` | `true` | Discovered registries should not auto-activate by default |
| `quarkus.micrometer.binder-enabled-default` | `true` | Automatic instrumentation should be mostly off unless explicitly enabled |
| `quarkus.micrometer.binder.enable-all` | `false` | You need to force-enable all discovered binders |
| `quarkus.micrometer.binder.http-server.enabled` | auto | Inbound HTTP metrics should be explicitly enabled or disabled |
| `quarkus.micrometer.binder.http-client.enabled` | auto | Outbound HTTP client metrics should be explicitly enabled or disabled |
| `quarkus.micrometer.binder.jvm` | auto | JVM metrics should be explicitly enabled or disabled |
| `quarkus.micrometer.binder.system` | auto | System metrics should be explicitly enabled or disabled |
| `quarkus.micrometer.binder.vertx.enabled` | auto | Vert.x metrics should be explicitly enabled or disabled |
| `quarkus.micrometer.binder.netty.enabled` | auto | Netty allocator metrics should be explicitly enabled or disabled |
| `quarkus.micrometer.binder.messaging.enabled` | auto | Reactive Messaging metrics should be explicitly enabled or disabled |
| `quarkus.micrometer.binder.virtual-threads.enabled` | `true` on supported JVMs | Virtual thread metrics should be enabled or disabled intentionally |

## Export endpoints and management placement

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.management.enabled` | `false` | Metrics should move to a separate management interface |
| `quarkus.micrometer.export.prometheus.enabled` | auto | Prometheus export should be explicitly enabled or disabled |
| `quarkus.micrometer.export.prometheus.path` | `metrics` | The Prometheus endpoint should move from `/q/metrics` or use an absolute path |
| `quarkus.micrometer.export.prometheus.default-registry` | `true` | You need to provide your own Prometheus registry instead of the default one |
| `quarkus.micrometer.export.json.enabled` | `false` | JSON metrics output is needed for debugging or integration |
| `quarkus.micrometer.export.json.path` | `metrics` | JSON metrics should live at a custom path |
| `quarkus.micrometer.export.prometheus."configuration-property-name"` | - | Backend-specific Prometheus registry settings must be passed through |

When management is enabled and paths stay relative, they resolve under the management root instead of the main app interface.

## HTTP label and cardinality control

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.micrometer.binder.http-server.match-patterns` | auto templating | Path parameters need a stable custom `uri` tag template |
| `quarkus.micrometer.binder.http-server.ignore-patterns` | - | Specific inbound paths should not produce metrics |
| `quarkus.micrometer.binder.http-server.suppress-non-application-uris` | `true` | `/q` endpoints should be included or excluded intentionally |
| `quarkus.micrometer.binder.http-server.suppress4xx-errors` | `false` | Unmatched client-side errors are causing URI cardinality noise |
| `quarkus.micrometer.binder.http-server.max-uri-tags` | `100` | The application needs a tighter or looser cap on unique inbound URI tags |
| `quarkus.micrometer.binder.http-client.match-patterns` | auto templating | Client request URIs need stable custom label templates |
| `quarkus.micrometer.binder.http-client.ignore-patterns` | - | Specific outbound calls should not produce metrics |
| `quarkus.micrometer.binder.http-client.suppress4xx-errors` | `false` | Unmatched client-side outbound errors are causing URI tag growth |
| `quarkus.micrometer.binder.http-client.max-uri-tags` | `100` | The application needs a tighter or looser cap on unique outbound URI tags |

Avoid deprecated Vert.x HTTP binder path controls; prefer the `http-server.*` properties.

## Micrometer to OpenTelemetry bridge and direct OTel metrics

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.otel.metrics.enabled` | `false` | Direct OpenTelemetry metrics must be enabled |
| `quarkus.otel.metrics.exporter` | `cdi` | Metrics exporter selection must be changed |
| `quarkus.otel.metric.export.interval` | `60S` | Export cadence should be shorter for debugging or faster feedback |
| `quarkus.otel.exporter.otlp.endpoint` | `http://localhost:4317/` | All OTLP signals should share one endpoint |
| `quarkus.otel.exporter.otlp.metrics.endpoint` | `http://localhost:4317/` | Metrics should go to a metrics-specific OTLP endpoint |
| `quarkus.otel.exporter.otlp.headers` | - | Shared OTLP auth or custom headers are required |
| `quarkus.otel.exporter.otlp.metrics.headers` | - | Metrics-only OTLP headers are required |
| `quarkus.otel.exporter.otlp.protocol` | `grpc` | OTLP transport must switch to `http/protobuf` or be made explicit |
| `quarkus.otel.instrument.jvm-metrics` | `true` | Direct OpenTelemetry JVM auto-metrics should be disabled to avoid overlap |
| `quarkus.otel.instrument.http-server-metrics` | `true` | Direct OpenTelemetry HTTP server auto-metrics should be disabled to avoid overlap |

For the Micrometer-to-OpenTelemetry bridge, Micrometer remains the API surface while OpenTelemetry exporter properties control delivery.

## Workflow note

Use `patterns.md` for setup recipes such as Prometheus-first projects, URI normalization, management-port placement, histogram tuning, and bridge-to-OTLP deployments.

## See Also

- [Patterns](./patterns.md) - repeatable metrics configuration workflows.
