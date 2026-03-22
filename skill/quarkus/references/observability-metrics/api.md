# Quarkus Observability Metrics API Reference

Use this file for Micrometer runtime APIs, automatic binder hooks, metrics endpoints, and the Micrometer-to-OpenTelemetry bridge.

## Default path: Micrometer

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-micrometer-registry-prometheus</artifactId>
</dependency>
```

This is the default Quarkus metrics path. It gives you:

- Micrometer APIs for custom application metrics
- automatic binders for HTTP, JVM, and many Quarkus extensions
- a scrape endpoint, usually `/q/metrics`

If you need Micrometer APIs but OTLP export instead of a Micrometer registry endpoint, see the bridge section below.

## Default endpoint behavior

- Metrics are exposed on the main HTTP server by default
- Prometheus/OpenMetrics endpoint: `/q/metrics`
- JSON export is optional and disabled by default
- Metrics appear lazily; many series do not exist until the code path runs

When the management interface is enabled, metrics can be served there instead of the main application interface.

## Inject `MeterRegistry`

```java
import io.micrometer.core.instrument.MeterRegistry;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/orders")
class OrderResource {
    private final MeterRegistry registry;

    OrderResource(MeterRegistry registry) {
        this.registry = registry;
    }

    @GET
    public String list() {
        registry.counter("orders.requests", "operation", "list").increment();
        return "ok";
    }
}
```

Use CDI injection instead of the global registry unless you have a strong reason not to.

## Create a counter

```java
registry.counter("orders.created", "channel", "api").increment();
```

Use counters for values that only increase.

## Create a timer

```java
import io.micrometer.core.instrument.Timer;

Timer.Sample sample = Timer.start(registry);
Order order = service.create(command);
sample.stop(registry.timer("orders.create", "result", "success"));
```

Use timers for latency. In Prometheus output, timers become `_seconds_count`, `_seconds_sum`, and `_seconds_max` style series.

## Create a gauge

```java
import java.util.ArrayList;
import java.util.List;

List<String> queue = registry.gaugeCollectionSize("orders.queue.size", new ArrayList<>());
```

Use gauges for values observed at scrape time, such as queue size or cache size. Avoid them when a counter or timer models the signal better.

## Create a distribution summary

```java
registry.summary("orders.payload.size", "format", "json").record(payloadBytes);
```

Use a distribution summary for non-time values such as payload size.

## Use annotations

```java
import io.micrometer.core.annotation.Counted;
import io.micrometer.core.annotation.Timed;

@Timed(value = "orders.calculate", extraTags = {"stage", "pricing"})
@Counted(value = "orders.calculate.calls")
public Price calculate(OrderCommand command) {
    return service.calculate(command);
}
```

Micrometer annotations are convenient, but they still need the same low-cardinality discipline as manual meters.

## Add tags from parameters with `@MeterTag`

```java
import io.micrometer.core.annotation.MeterTag;
import io.micrometer.core.annotation.Timed;

@Timed(value = "orders.lookup")
public Order lookup(@MeterTag(key = "source") String source) {
    return service.lookup(source);
}
```

Only use bounded values. Do not tag with user input unless the set is small and controlled.

## Automatic binders and built-in metrics

Quarkus Micrometer automatically instruments common areas including:

- HTTP server requests
- HTTP client requests
- JVM and system metrics
- Vert.x and Netty internals
- many Quarkus extensions such as REST, ORM, messaging, scheduler, cache, and Redis

HTTP server request timing is usually the first metric family you will inspect.

Common Prometheus series include:

- `http_server_requests_seconds_count`
- `http_server_requests_seconds_sum`
- `http_server_requests_seconds_max`

Typical labels include `uri`, `method`, `status`, and `outcome`.

## Control emitted tags and distributions with `MeterFilter`

```java
import io.micrometer.core.instrument.config.MeterFilter;
import io.micrometer.core.instrument.distribution.DistributionStatisticConfig;
import jakarta.enterprise.inject.Produces;
import jakarta.inject.Singleton;

@Singleton
class MetricsConfig {
    @Produces
    @Singleton
    MeterFilter ordersLatencyBuckets() {
        return new MeterFilter() {
            @Override
            public DistributionStatisticConfig configure(
                    io.micrometer.core.instrument.Meter.Id id,
                    DistributionStatisticConfig config) {
                if (id.getName().equals("orders.create")) {
                    return DistributionStatisticConfig.builder()
                            .percentilesHistogram(true)
                            .build()
                            .merge(config);
                }
                return config;
            }
        };
    }
}
```

Use `MeterFilter` for common tags, selective histogram enablement, renaming, and denial rules.

## Add HTTP tags with contributor hooks

- `io.quarkus.micrometer.runtime.HttpServerMetricsTagsContributor`
- `io.quarkus.micrometer.runtime.HttpClientMetricsTagsContributor`

These CDI hooks let you add request-derived tags. Keep values bounded.

## Customize registries

- `io.quarkus.micrometer.runtime.MeterRegistryCustomizer`
- custom `@Produces` method returning a specific `MeterRegistry`

Use these when the registry itself needs custom initialization, backend-specific options, or registry-scoped behavior.

## Management interface placement

Metrics live on the main HTTP interface by default. If `quarkus.management.enabled=true`, metrics can be served from the management port and root path instead.

Typical result:

- main interface stays focused on application traffic
- operations tooling scrapes metrics from the management interface

Use `configuration.md` for path and export property combinations.

## Micrometer to OpenTelemetry bridge

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-micrometer-opentelemetry</artifactId>
</dependency>
```

Use this when you want to:

- keep Micrometer APIs and Quarkus automatic Micrometer instrumentation
- export metrics through the OpenTelemetry SDK and OTLP pipeline
- unify metrics with OpenTelemetry traces and logs

Important bridge behavior:

- the extension is preview
- Micrometer APIs still work normally
- OpenTelemetry SDK handles metric export
- some Micrometer instruments map to different OpenTelemetry instrument shapes; for example, a Micrometer `Timer` becomes a histogram plus a max gauge on the OpenTelemetry side

## Direct OpenTelemetry metrics

Direct OpenTelemetry metrics are advanced and secondary in Quarkus.

```java
import io.opentelemetry.api.metrics.LongCounter;
import io.opentelemetry.api.metrics.Meter;

class MetricResource {
    private final LongCounter counter;

    MetricResource(Meter meter) {
        this.counter = meter.counterBuilder("orders.processed")
                .setDescription("Processed orders")
                .build();
    }
}
```

Use this path only when you intentionally want OpenTelemetry-native instruments and exporter behavior. For most Quarkus metrics work, prefer Micrometer or the Micrometer-to-OpenTelemetry bridge.
