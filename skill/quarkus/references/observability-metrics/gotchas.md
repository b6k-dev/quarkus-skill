# Quarkus Observability Metrics Gotchas

Common Quarkus metrics pitfalls, symptoms, and fixes.

## Cardinality and tagging

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Metrics backend slows down or series count explodes | Tags use unbounded values such as user ID, raw path, request ID, or exception message | Keep tags low-cardinality and normalize dynamic paths with `match-patterns` |
| HTTP metrics stop creating new URI series after a while | `max-uri-tags` limit was reached | Reduce URI variability, add `match-patterns`, or intentionally raise `quarkus.micrometer.binder.http-server.max-uri-tags` or `http-client.max-uri-tags` |
| Extra request tags make dashboards noisy or expensive | `HttpServerMetricsTagsContributor`, `HttpClientMetricsTagsContributor`, or `@MeterTag` uses high-cardinality values | Restrict tags to small enumerations such as channel, region, outcome, or tier |

## Endpoint exposure and placement

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `/q/metrics` is missing in a packaged deployment | Prometheus registry is not present or was disabled | Add `quarkus-micrometer-registry-prometheus` or re-enable Prometheus export |
| Metrics appear on the wrong interface or port | `quarkus.management.enabled` was not considered when choosing endpoint paths | Decide whether metrics belong on the main or management interface and configure paths accordingly |
| Changing management placement has no effect in production | `quarkus.management.enabled` is build-time | Rebuild the application after changing management interface enablement |

## Automatic binders and meter behavior

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Expected metric family never appears | Metrics are lazy and the code path has not executed yet | Trigger the endpoint, scheduled job, or business path before checking output |
| Gauge shows `NaN` or disappears | The observed object was garbage-collected | Keep a strong application reference to the observed object |
| Timer percentile values cannot be aggregated cleanly across instances | Percentiles were precomputed instead of derived from histograms | Prefer histogram buckets for cross-instance aggregation; use percentiles only when local view is enough |
| `/q` endpoints are missing from HTTP metrics | Non-application URIs are suppressed by default | Set `quarkus.micrometer.binder.http-server.suppress-non-application-uris=false` if you really need those metrics |

## Bridge and OpenTelemetry overlap

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Metric names or instrument types differ after moving to OTLP | Micrometer instruments are mapped to OpenTelemetry instrument shapes by the bridge | Validate dashboards and collectors against bridge output; a Micrometer `Timer` becomes a histogram plus max gauge |
| Duplicate or confusing JVM and HTTP metrics appear in OTLP pipelines | Direct OpenTelemetry metrics instrumentation overlaps with Micrometer or bridge-generated metrics | Disable `quarkus.otel.instrument.jvm-metrics` and `quarkus.otel.instrument.http-server-metrics` when Micrometer already covers those signals |
| Bridge-based deployment changes behavior across Quarkus upgrades | `quarkus-micrometer-opentelemetry` is preview | Treat the bridge as advanced, pin versions carefully, and verify dashboards during upgrades |
