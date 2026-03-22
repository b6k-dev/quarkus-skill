# Quarkus Observability Metrics Usage Patterns

Use these patterns for repeatable Micrometer-first metrics workflows.

## Pattern: Add standard Quarkus metrics with Prometheus output

When to use:

- You want the default Quarkus metrics path for dashboards and scraping.

Command:

```bash
quarkus extension add micrometer-registry-prometheus
```

Verify by starting dev mode, hitting one application endpoint, and checking `/q/metrics` for `http_server_requests_seconds_count`.

## Pattern: Add a custom counter and timer to business code

When to use:

- You need one or two focused application metrics beyond automatic binders.

Example:

```java
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;

class CheckoutService {
    private final MeterRegistry registry;

    CheckoutService(MeterRegistry registry) {
        this.registry = registry;
    }

    Receipt checkout(Cart cart) {
        Timer.Sample sample = Timer.start(registry);
        try {
            Receipt receipt = doCheckout(cart);
            registry.counter("checkout.requests", "result", "success").increment();
            return receipt;
        } catch (RuntimeException e) {
            registry.counter("checkout.requests", "result", "failure").increment();
            throw e;
        } finally {
            sample.stop(registry.timer("checkout.latency", "channel", "api"));
        }
    }
}
```

Keep names stable and tags bounded so the metric stays aggregatable over time.

## Pattern: Expose a gauge from in-memory state

When to use:

- You need to observe queue depth, cache size, or another sampled value.

Example:

```java
import io.micrometer.core.instrument.MeterRegistry;
import java.util.concurrent.LinkedBlockingQueue;

class WorkQueue {
    private final LinkedBlockingQueue<String> queue = new LinkedBlockingQueue<>();

    WorkQueue(MeterRegistry registry) {
        registry.gaugeCollectionSize("work.queue.size", queue);
    }
}
```

Prefer a gauge only when the signal is naturally sampled instead of counted.

## Pattern: Normalize HTTP URI tags and cap cardinality

When to use:

- Dynamic request paths or error paths are creating too many `uri` tag values.

Example:

```properties
quarkus.micrometer.binder.http-server.match-patterns=/orders/[0-9]+=/orders/{id}
quarkus.micrometer.binder.http-server.ignore-patterns=/health/live,/internal/.*
quarkus.micrometer.binder.http-server.max-uri-tags=50
quarkus.micrometer.binder.http-server.suppress4xx-errors=true
```

Use the same approach for outbound calls with `quarkus.micrometer.binder.http-client.*`.

## Pattern: Add common tags and selective histograms with `MeterFilter`

When to use:

- All metrics need an environment tag or one latency family needs histogram buckets.

Example:

```java
import io.micrometer.core.instrument.Tag;
import io.micrometer.core.instrument.config.MeterFilter;
import io.micrometer.core.instrument.distribution.DistributionStatisticConfig;
import jakarta.enterprise.inject.Produces;
import jakarta.inject.Singleton;
import java.util.List;

@Singleton
class MetricsFilters {
    @Produces
    @Singleton
    MeterFilter commonTags() {
        return MeterFilter.commonTags(List.of(Tag.of("env", "prod")));
    }

    @Produces
    @Singleton
    MeterFilter checkoutHistogram() {
        return new MeterFilter() {
            @Override
            public DistributionStatisticConfig configure(
                    io.micrometer.core.instrument.Meter.Id id,
                    DistributionStatisticConfig config) {
                if (id.getName().equals("checkout.latency")) {
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

Prefer common tags that describe deployment context, not request identity.

## Pattern: Add bounded request tags from HTTP context

When to use:

- HTTP metrics need one extra low-cardinality dimension such as API group or tenant tier.

Example:

```java
import io.quarkus.micrometer.runtime.HttpServerMetricsTagsContributor;
import io.vertx.ext.web.RoutingContext;
import jakarta.inject.Singleton;

@Singleton
class ApiGroupTags implements HttpServerMetricsTagsContributor {
    @Override
    public Iterable<io.micrometer.core.instrument.Tag> contribute(RoutingContext context) {
        return java.util.List.of(io.micrometer.core.instrument.Tag.of("api_group", "public"));
    }
}
```

Only add tags with a small, known set of values.

## Pattern: Move metrics to the management interface

When to use:

- Operations endpoints should be isolated from public traffic.

Example:

```properties
quarkus.management.enabled=true
quarkus.micrometer.export.prometheus.path=metrics/prometheus
quarkus.micrometer.export.json.enabled=true
quarkus.micrometer.export.json.path=metrics/json
```

With default management host and port, this places metrics under the management interface instead of the main app interface.

## Pattern: Send Micrometer metrics through OpenTelemetry

When to use:

- You want Micrometer APIs and Quarkus binders, but OTLP as the delivery path.

Command:

```bash
quarkus extension add micrometer-opentelemetry
```

Example:

```properties
quarkus.micrometer.binder.http-server.enabled=true
quarkus.micrometer.binder.jvm=true
quarkus.otel.exporter.otlp.metrics.endpoint=http://otel-collector:4317
```

If you also enable direct OpenTelemetry metrics instrumentation, disable overlapping HTTP server and JVM metric instrumentations unless you explicitly want both models.

## Pattern: Use direct OpenTelemetry metrics intentionally

When to use:

- A team has standardized on OpenTelemetry-native instruments and accepts preview status.

Example:

```properties
quarkus.otel.metrics.enabled=true
quarkus.otel.exporter.otlp.metrics.endpoint=http://otel-collector:4317
quarkus.otel.instrument.jvm-metrics=false
quarkus.otel.instrument.http-server-metrics=false
```

Use this for advanced cases only. For mainstream Quarkus metrics work, prefer Micrometer or the bridge.
