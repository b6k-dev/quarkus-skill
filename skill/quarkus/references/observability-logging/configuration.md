# Quarkus Observability Logging Settings Reference

Use this file for high-value Quarkus logging properties, especially level control, handlers, structured output, and OTel log export touchpoints.

## High-value logging configuration keys

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.log.level` | `INFO` | You need to set the root logger level |
| `quarkus.log.min-level` | `DEBUG` | You need runtime `TRACE` or other low-level logs compiled in |
| `quarkus.log.category."<category>".level` | `inherit` | One package or subsystem needs a different level |
| `quarkus.log.category."<category>".min-level` | `inherit` | A category must allow runtime logs below `DEBUG` |
| `quarkus.log.category."<category>".handlers` | - | A category should write to named handlers |
| `quarkus.log.category."<category>".use-parent-handlers` | `true` | Category output should not also flow to root handlers |
| `quarkus.log.handlers` | - | Extra named handlers should attach to the root logger |
| `quarkus.log.console.enabled` | `true` | Console output must be disabled or re-enabled |
| `quarkus.log.console.level` | `ALL` | Console should be quieter than the root logger |
| `quarkus.log.console.format` | text pattern | Human-readable console layout must change |
| `quarkus.log.console.async.enabled` | `false` | Console logging should be asynchronous |
| `quarkus.log.file.enabled` | `false` | Logs must also be written to a local file |
| `quarkus.log.file.path` | `quarkus.log` | File output needs a known path |
| `quarkus.log.file.level` | `ALL` | File output should use a different threshold |
| `quarkus.log.file.rotation.max-file-size` | `10M` | Rotated file size needs tuning |
| `quarkus.log.file.rotation.max-backup-index` | `5` | Retained file count needs tuning |
| `quarkus.log.file.rotation.file-suffix` | - | Rotation should be time-based or compressed |
| `quarkus.log.syslog.enabled` | `false` | Logs should be shipped to a syslog endpoint |
| `quarkus.log.syslog.endpoint` | - | Syslog target host and port must be configured |
| `quarkus.log.syslog.app-name` | - | Syslog records need an application name |
| `quarkus.log.socket.enabled` | `false` | Logs should be shipped to a raw socket endpoint such as Logstash |
| `quarkus.log.socket.endpoint` | - | Socket shipping target must be configured |

## Structured JSON logging keys (`quarkus-logging-json`)

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.log.console.json.enabled` | `false` | Console logs should be JSON |
| `quarkus.log.console.json.log-format` | `default` | Output should follow `default`, `ecs`, or `gcp` schema |
| `quarkus.log.console.json.pretty-print` | `false` | Humans inspect the JSON directly and downstream tooling tolerates formatting |
| `quarkus.log.console.json.exception-output-type` | `detailed` | Stack traces should be flattened or changed |
| `quarkus.log.console.json.print-details` | `false` | Caller class/file/method/line should be included |
| `quarkus.log.console.json.key-overrides` | - | Default JSON field names must change |
| `quarkus.log.console.json.excluded-keys` | - | Specific JSON fields should be omitted |
| `quarkus.log.console.json.additional-field."<name>".value` | required | Static metadata should be added to every record |
| `quarkus.log.file.json.enabled` | `false` | File logs should be JSON |
| `quarkus.log.syslog.json.enabled` | `false` | Syslog payloads should be JSON |
| `quarkus.log.socket.json.enabled` | `false` | Socket-shipped payloads should be JSON |
| `quarkus.log.socket.json.log-format` | `default` | Socket payloads should use ECS or another supported schema |

Notes:

- JSON formatting replaces normal text formatting for the selected handler.
- `ecs` is the usual choice for Elastic-oriented pipelines.
- `gcp` can flatten trace correlation fields for Google Cloud logging expectations.

## OpenTelemetry log touchpoint keys

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.otel.logs.enabled` | `false` | Quarkus logs should also be emitted as OpenTelemetry log records |
| `quarkus.otel.logs.handler.enabled` | `true` | The Quarkus-to-OTel log bridge must be disabled at runtime |
| `quarkus.otel.logs.level` | `ALL` | Only logs at or above a threshold should be exported to OTel |
| `quarkus.otel.logs.exporter` | `cdi` | Export path should change to `none` or a debugging exporter |
| `quarkus.otel.exporter.otlp.logs.endpoint` | `http://localhost:4317/` | Logs should be sent to a non-default OTLP endpoint |
| `quarkus.otel.exporter.otlp.logs.headers` | - | OTLP log export needs auth or custom headers |
| `quarkus.otel.exporter.otlp.logs.protocol` | `grpc` | OTLP logs should use `http/protobuf` instead of gRPC |
| `quarkus.otel.blrp.schedule.delay` | `1S` | Batch log record export cadence needs tuning |
| `quarkus.otel.blrp.max.queue.size` | `2048` | Export queue size needs tuning |
| `quarkus.otel.blrp.max.export.batch.size` | `512` | Batch size needs tuning |
| `quarkus.otel.blrp.export.timeout` | `30S` | Export timeout needs tuning |
| `quarkus.application.name` | artifact id | Exported logs should carry an explicit service name |

## Common configuration patterns

Root level plus one noisy category:

```properties
quarkus.log.level=INFO
quarkus.log.category."org.hibernate.SQL".level=DEBUG
```

Allow runtime `TRACE` for one category:

```properties
quarkus.log.category."com.acme.billing".min-level=TRACE
quarkus.log.category."com.acme.billing".level=TRACE
```

Human-readable dev, JSON prod:

```properties
%dev.quarkus.log.console.json.enabled=false
%test.quarkus.log.console.json.enabled=false
%prod.quarkus.log.console.json.enabled=true
%prod.quarkus.log.console.json.log-format=ecs
```

Console text plus socket JSON shipping:

```properties
quarkus.log.console.json.enabled=false
quarkus.log.socket.enabled=true
quarkus.log.socket.endpoint=localhost:4560
quarkus.log.socket.json.enabled=true
quarkus.log.socket.json.log-format=ecs
```

Trace correlation in regular logs:

```properties
quarkus.log.console.format=%d{HH:mm:ss} %-5p traceId=%X{traceId} spanId=%X{spanId} [%c{2.}] (%t) %s%e%n
```
