# Quarkus Observability Tracing Configuration Reference

Use this file for high-value tracing, OTLP export, propagation, and instrumentation settings.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.otel.enabled` | `true` | You need to disable the OpenTelemetry extension at build time |
| `quarkus.otel.traces.enabled` | `true` | Tracing should be disabled while keeping the extension present |
| `quarkus.otel.exporter.otlp.enabled` | `true` | Context propagation should remain but OTLP export must be turned off |
| `quarkus.otel.exporter.otlp.endpoint` | `http://localhost:4317/` | All telemetry should go to one collector endpoint |
| `quarkus.otel.exporter.otlp.traces.endpoint` | `http://localhost:4317/` | Traces should go to a dedicated collector endpoint |
| `quarkus.otel.exporter.otlp.protocol` | `grpc` | Collector requires `http/protobuf` instead of gRPC |
| `quarkus.otel.exporter.otlp.headers` | - | OTLP requests need auth or tenant headers |
| `quarkus.otel.exporter.otlp.compression` | disabled | Export payload size should be reduced |
| `quarkus.otel.exporter.otlp.timeout` | `10s` | Slow collectors need a larger export timeout |
| `quarkus.otel.simple` | `false` | Serverless or short-lived processes should export spans immediately |
| `quarkus.otel.traces.sampler` | `parentbased_always_on` | Trace volume should be reduced or disabled |
| `quarkus.otel.traces.sampler.arg` | `1.0d` | Ratio-based sampler needs a percentage |
| `quarkus.otel.propagators` | `tracecontext,baggage` | You must interoperate with B3, Jaeger, or another propagation format |
| `quarkus.otel.traces.suppress-non-application-uris` | `true` | You need `/q` endpoints traced for diagnostics |
| `quarkus.otel.traces.suppress-application-uris` | - | Specific application paths should not generate spans |
| `quarkus.otel.traces.include-static-resources` | `false` | Static resource requests should be traced |
| `quarkus.otel.instrument.rest` | `true` | Quarkus REST server instrumentation should be toggled |
| `quarkus.otel.instrument.resteasy-client` | `true` | RESTEasy Classic client tracing should be toggled |
| `quarkus.otel.instrument.grpc` | `true` | gRPC tracing should be toggled |
| `quarkus.otel.instrument.messaging` | `true` | Messaging tracing should be toggled |
| `quarkus.datasource.jdbc.telemetry` | `false` | JDBC spans should be emitted for datasource calls |
| `quarkus.otel.traces.eusp.enabled` | `false` | End-user identity attributes should be added to spans |
| `quarkus.otel.security-events.enabled` | `false` | Quarkus security events should be emitted as span events |
| `quarkus.application.name` | artifact id | Service name should be stable across deployments |
| `quarkus.otel.resource.attributes` | - | Resource metadata like environment or namespace should be attached |

## Baseline OTLP setup

```properties
quarkus.application.name=checkout
quarkus.otel.exporter.otlp.endpoint=http://localhost:4317
quarkus.otel.exporter.otlp.headers=authorization=Bearer my-token
```

## Switch to OTLP HTTP/protobuf

```properties
quarkus.otel.exporter.otlp.protocol=http/protobuf
quarkus.otel.exporter.otlp.endpoint=http://localhost:4318
```

If you change protocol, also change the port from `4317` to `4318`.

## Reduce trace volume

```properties
quarkus.otel.traces.sampler=traceidratio
quarkus.otel.traces.sampler.arg=0.1
quarkus.otel.traces.suppress-application-uris=/health,/internal/*
```

## Propagation compatibility

```properties
quarkus.otel.propagators=tracecontext,baggage,b3
```

Extra propagators such as `b3`, `b3multi`, `jaeger`, `ottrace`, or `xray` need their matching OpenTelemetry library on the classpath.

## Trace correlation in logs

```properties
quarkus.log.console.format=%d{HH:mm:ss} %-5p traceId=%X{traceId}, parentId=%X{parentId}, spanId=%X{spanId}, sampled=%X{sampled} [%c{2.}] (%t) %s%e%n
quarkus.http.access-log.pattern="...traceId=%{X,traceId} spanId=%{X,spanId}"
```

## Build-time reminder

Sampler selection, instrumentation toggles, and major enable/disable switches are commonly build-time sensitive. If a property change appears ignored, rebuild or restart the app instead of relying on hot reload.
