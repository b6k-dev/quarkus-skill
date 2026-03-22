# Quarkus Observability Tracing Usage Patterns

Use these patterns for repeatable tracing setup, propagation, and migration workflows.

## Pattern: Bootstrap OTLP tracing in a service

When to use:

- You are enabling Quarkus-native tracing for a service.

Command:

```bash
quarkus create app com.acme:checkout --extension='rest,opentelemetry' --no-code
```

Configuration:

```properties
quarkus.application.name=checkout
quarkus.otel.exporter.otlp.endpoint=http://localhost:4317
```

Quarkus REST endpoints are traced automatically once the extension is present.

## Pattern: Add a business child span inside an already traced request

When to use:

- Automatic HTTP or messaging spans exist, but one internal step needs its own timing and attributes.

Example:

```java
@ApplicationScoped
class InventoryService {
    @WithSpan("inventory.reserve")
    void reserve(@SpanAttribute("sku") String sku) {
    }
}
```

Prefer `@WithSpan` on CDI service methods. Avoid it on JAX-RS resource methods already wrapped by automatic server spans.

## Pattern: Control trace volume without removing instrumentation

When to use:

- Production traffic is too noisy or too expensive to export in full.

Configuration:

```properties
quarkus.otel.traces.sampler=traceidratio
quarkus.otel.traces.sampler.arg=0.2
quarkus.otel.traces.suppress-application-uris=/internal/*,/q/swagger-ui*
```

Use sampler ratio for global volume control and URI suppression for known low-value endpoints.

## Pattern: Propagate traces across custom or brokered boundaries

When to use:

- Work continues in another service, message, or client library that does not propagate trace headers automatically.

Example:

```java
TracingMetadata metadata = TracingMetadata.withPrevious(Context.current());
Message<String> message = Message.of(payload).withMetadata(metadata);
```

For non-standard transports, inject `OpenTelemetry` and use the active `TextMapPropagator` to inject and extract headers from your carrier.

## Pattern: Migrate from OpenTracing in one pass

When to use:

- The app uses legacy OpenTracing extension or properties and you want the Quarkus 3.x default model.

Steps:

1. Replace `quarkus-smallrye-opentracing` with `quarkus-opentelemetry`.
2. Remove `quarkus.jaeger.*` properties.
3. Migrate common settings: `service-name` -> `quarkus.application.name`, `endpoint` -> `quarkus.otel.exporter.otlp.traces.endpoint`, sampler settings -> `quarkus.otel.traces.sampler*`, propagation -> `quarkus.otel.propagators`.
4. Replace `io.opentracing.Tracer` with `io.opentelemetry.api.trace.Tracer` and `@Traced` with `@WithSpan` only on non-automatic spans.

If the codebase has many manual instrumentation points, use the OpenTracing shim temporarily while converting call sites incrementally.

## Pattern: Keep legacy OpenTracing code working during staged migration

When to use:

- Manual instrumentation is widespread and cannot be rewritten in one change.

Example:

```java
@Inject
OpenTelemetry openTelemetry;

Tracer legacyTracer = OpenTracingShim.createTracerShim(openTelemetry);
```

Use the shim only as a bridge. New code should use OpenTelemetry APIs directly.
