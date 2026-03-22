# Quarkus Observability Logging Usage Patterns

Use these patterns for repeatable Quarkus logging workflows.

## Pattern: Raise One Category Without Flooding Everything

When to use:

- One framework or package needs deeper diagnostics.

Example:

```properties
quarkus.log.level=INFO
quarkus.log.category."org.hibernate".level=DEBUG
```

If you need `TRACE`, also lower the category minimum level:

```properties
quarkus.log.category."org.hibernate".min-level=TRACE
quarkus.log.category."org.hibernate".level=TRACE
```

## Pattern: Send One Category to a Dedicated File

When to use:

- Audit, integration, or partner traffic logs should be separated from general app logs.

Example:

```properties
quarkus.log.handler.file.audit.enabled=true
quarkus.log.handler.file.audit.path=logs/audit.log
quarkus.log.handler.file.audit.format=%d{yyyy-MM-dd HH:mm:ss} %-5p [%c] %s%e%n

quarkus.log.category."com.acme.audit".handlers=audit
quarkus.log.category."com.acme.audit".use-parent-handlers=false
quarkus.log.category."com.acme.audit".level=INFO
```

Set `use-parent-handlers=false` if duplicate records in root handlers are not wanted.

## Pattern: Add Request Context with MDC

When to use:

- Operators need request IDs, tenant IDs, or business correlation keys in each log line.

Example:

```java
import io.quarkus.logging.Log;
import org.jboss.logmanager.MDC;

MDC.put("request.id", requestId);
MDC.put("tenant", tenantId);
try {
    Log.info("Request accepted");
} finally {
    MDC.clear();
}
```

```properties
quarkus.log.console.format=%d{HH:mm:ss} %-5p request.id=%X{request.id} tenant=%X{tenant} [%c{2.}] %s%n
```

Prefer small, stable keys. Avoid user payloads or large dynamic values.

## Pattern: Keep Text Logs in Dev and JSON in Production

When to use:

- Developers want readable local logs, but production pipelines require structured records.

Example:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-logging-json</artifactId>
</dependency>
```

```properties
%dev.quarkus.log.console.json.enabled=false
%test.quarkus.log.console.json.enabled=false

%prod.quarkus.log.console.json.enabled=true
%prod.quarkus.log.console.json.log-format=ecs
%prod.quarkus.log.console.json.additional-field."service".value=orders
```

## Pattern: Ship Logs to Centralized Logging with Socket + ECS JSON

When to use:

- A log pipeline such as Logstash expects structured JSON over TCP.

Example:

```properties
quarkus.log.console.json.enabled=false

quarkus.log.socket.enabled=true
quarkus.log.socket.endpoint=localhost:4560
quarkus.log.socket.json.enabled=true
quarkus.log.socket.json.log-format=ecs
quarkus.log.socket.json.exception-output-type=formatted
```

Use this for direct structured shipping while keeping normal console logs for local inspection.

## Pattern: Ship Logs with Syslog

When to use:

- Infrastructure already standardizes on syslog ingestion.

Example:

```properties
quarkus.log.syslog.enabled=true
quarkus.log.syslog.endpoint=localhost:5140
quarkus.log.syslog.protocol=udp
quarkus.log.syslog.app-name=orders
quarkus.log.syslog.hostname=orders-prod-1
```

Prefer syslog over deprecated GELF setups when the receiver supports it.

## Pattern: Correlate Logs with Traces Without Switching to OTel Logs

When to use:

- You want trace/span IDs in ordinary app logs, but the primary sink is still console, file, or socket logging.

Example:

```properties
quarkus.log.console.format=%d{HH:mm:ss} %-5p traceId=%X{traceId} spanId=%X{spanId} [%c{2.}] (%t) %s%e%n
```

This keeps the standard Quarkus logging path while improving trace-to-log correlation.

## Pattern: Export Logs Through OpenTelemetry

When to use:

- A collector-based observability platform expects OTel log records in addition to traces.

Example:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-opentelemetry</artifactId>
</dependency>
```

```properties
quarkus.application.name=orders
quarkus.otel.logs.enabled=true
quarkus.otel.exporter.otlp.logs.endpoint=http://localhost:4317
quarkus.otel.logs.level=ERROR
```

Treat this as an explicit exporter path. It does not replace the need to decide how your normal Quarkus handlers should behave.
