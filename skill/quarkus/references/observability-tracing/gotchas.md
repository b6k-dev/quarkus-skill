# Quarkus Observability Tracing Gotchas

Common tracing pitfalls, symptoms, and fixes.

## Export and collector issues

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Spans appear in logs but never reach the backend | Collector endpoint, protocol, or port does not match Quarkus defaults | Verify `quarkus.otel.exporter.otlp.endpoint`, and use `4317` for `grpc` or `4318` for `http/protobuf` |
| No telemetry leaves the app even though requests are traced | OTLP exporter disabled with `quarkus.otel.exporter.otlp.enabled=false` or traces exporter set to `none` | Re-enable OTLP export or use an appropriate traces exporter |
| Short-lived jobs lose final spans | Batch export does not flush before shutdown | Use `quarkus.otel.simple=true` or increase shutdown wait and ensure graceful shutdown |

## Span shape and sampling

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| HTTP requests show duplicate nested spans | `@WithSpan` was added to a JAX-RS method that is already auto-instrumented | Remove `@WithSpan` from the endpoint and keep manual spans in CDI service methods |
| `/q` endpoints are missing from traces | Non-application URIs are suppressed by default | Set `quarkus.otel.traces.suppress-non-application-uris=false` when you need diagnostics for `/q` |
| Specific paths still generate spans after suppression config | `quarkus.http.root-path` was not included in the configured URI patterns | Configure suppressed URIs using the final served path |
| Sampling or URI suppression changes seem ignored | Relevant properties are build-time sensitive or app was not restarted | Rebuild or fully restart after changing tracing build-time properties |

## Propagation and instrumentation

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Downstream service starts a new trace instead of joining the caller trace | Headers or message metadata are not propagated | Use built-in Quarkus client/messaging instrumentation or inject/extract context manually with propagators |
| B3 or Jaeger propagation config fails at runtime | Matching OpenTelemetry propagator library is missing | Add the required propagator dependency before setting `quarkus.otel.propagators` |
| Expected spans disappear for REST, gRPC, or messaging operations | Corresponding `quarkus.otel.instrument.*` property was disabled | Re-enable the specific instrumentation toggle |
| JDBC calls are not visible in traces | Datasource telemetry wrapper is off by default | Set `quarkus.datasource.jdbc.telemetry=true` on the datasource that should emit spans |

## Migration from OpenTracing

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Build fails with unsatisfied dependency for `io.opentracing.Tracer` | Project now uses `quarkus-opentelemetry` but code still injects the legacy tracer | Replace injection with OpenTelemetry APIs or create a temporary shim tracer from `OpenTelemetry` |
| Span names or counts change after migration | OpenTelemetry automatic instrumentation differs from old OpenTracing behavior | Re-check endpoint annotations, remove redundant manual spans, and validate traces in the backend |
| Error tags no longer show as expected | OpenTracing tags were copied directly instead of using OpenTelemetry error APIs | Use `span.setStatus(StatusCode.ERROR)` and `span.recordException(e)` |

## Security-related enrichment

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| End-user attributes never appear on spans | `quarkus.otel.traces.eusp.enabled` is off or authentication has not completed before the span is created | Enable the feature and ensure spans are created after authentication, optionally with proactive auth |
| Security events are not visible in traces | Security event export is disabled | Set `quarkus.otel.security-events.enabled=true` and confirm the relevant security extension is active |
