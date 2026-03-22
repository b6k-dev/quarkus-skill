# Quarkus Observability Logging API Reference

Use this file for runtime logging APIs and short, copy-ready examples.

## `org.jboss.logging.Logger`

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import org.jboss.logging.Logger;

@ApplicationScoped
class BillingService {
    @Inject
    Logger log;

    void charge(String accountId) {
        log.infov("Charging account {0}", accountId);
    }
}
```

- Quarkus routes JBoss Logging through JBoss Log Manager.
- Use parameterized logging to avoid unnecessary string building.

## `io.quarkus.logging.Log`

```java
import io.quarkus.logging.Log;

class ImportJob {
    void run() {
        Log.debug("Starting import");
        Log.info("Import finished");
    }
}
```

- Handy static facade for application code.
- Prefer one style consistently in a codebase.

## MDC for request correlation

```java
import io.quarkus.logging.Log;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.jboss.logmanager.MDC;

import java.util.UUID;

@Path("/orders")
public class OrderResource {
    @GET
    public String get() {
        MDC.put("request.id", UUID.randomUUID().toString());
        MDC.put("tenant", "acme");
        try {
            Log.info("Order request received");
            return "ok";
        } finally {
            MDC.clear();
        }
    }
}
```

```properties
quarkus.log.console.format=%d{HH:mm:ss} %-5p request.id=%X{request.id} tenant=%X{tenant} [%c{2.}] (%t) %s%n
```

- Quarkus provides MDC propagation for reactive and async flows.
- Use `%X{key}` for a single value or `%X` for the full MDC map.

## Custom logging filter

```java
import io.quarkus.logging.LoggingFilter;
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.config.inject.ConfigProperty;

import java.util.logging.Filter;
import java.util.logging.LogRecord;

@LoggingFilter(name = "health-filter")
@ApplicationScoped
public final class HealthFilter implements Filter {
    private final String suppressedPath;

    public HealthFilter(@ConfigProperty(name = "logging.health.path") String suppressedPath) {
        this.suppressedPath = suppressedPath;
    }

    @Override
    public boolean isLoggable(LogRecord record) {
        return !record.getMessage().contains(suppressedPath);
    }
}
```

```properties
logging.health.path=/q/health
quarkus.log.console.filter=health-filter
```

## Named handlers for category-specific output

```properties
quarkus.log.handler.file.audit.enabled=true
quarkus.log.handler.file.audit.path=logs/audit.log
quarkus.log.handler.file.audit.format=%d{yyyy-MM-dd HH:mm:ss} %-5p [%c] %s%e%n

quarkus.log.category."com.acme.audit".level=INFO
quarkus.log.category."com.acme.audit".handlers=audit
quarkus.log.category."com.acme.audit".use-parent-handlers=false
```

- Category config applies recursively to subcategories.
- Set `use-parent-handlers=false` when you want isolation instead of duplication.

## Structured JSON logging

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-logging-json</artifactId>
</dependency>
```

```properties
quarkus.log.console.json.enabled=true
quarkus.log.console.json.log-format=ecs
quarkus.log.console.json.additional-field."service".value=orders
```

- JSON formatting is handler-specific (`console`, `file`, `syslog`, `socket`).
- When JSON formatting takes over a handler, the normal text `format` pattern is ignored for that handler.

## OpenTelemetry log export bridge

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-opentelemetry</artifactId>
</dependency>
```

```properties
quarkus.otel.logs.enabled=true
quarkus.otel.exporter.otlp.logs.endpoint=http://localhost:4317
quarkus.otel.logs.level=ERROR
```

- `quarkus.otel.logs.enabled` is build-time.
- `quarkus.otel.logs.handler.enabled` controls the runtime bridge from Quarkus logging into OTel log records.

## Trace ID correlation in regular logs

```properties
quarkus.log.console.format=%d{HH:mm:ss} %-5p traceId=%X{traceId} spanId=%X{spanId} [%c{2.}] (%t) %s%e%n
```

- This works with Quarkus tracing/OpenTelemetry MDC enrichment.
- Use this when you want correlation in ordinary logs without switching fully to OpenTelemetry log export.

## Other logging APIs

Add adapters only when a library is not already handled by a Quarkus extension.

```xml
<dependency>
    <groupId>org.jboss.slf4j</groupId>
    <artifactId>slf4j-jboss-logmanager</artifactId>
</dependency>
```

```xml
<dependency>
    <groupId>org.jboss.logmanager</groupId>
    <artifactId>log4j2-jboss-logmanager</artifactId>
</dependency>
```

- Do not bring an alternate logging backend into a Quarkus app unless you know the integration consequences.
