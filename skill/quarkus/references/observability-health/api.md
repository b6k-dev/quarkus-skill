# Quarkus Health API Reference

Use this file for runtime health endpoints, health check types, group annotations, and Health UI behavior.

## Extension entry point

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-health</artifactId>
</dependency>
```

The extension exposes health endpoints immediately, even before you add custom checks.

## Default endpoints

- Aggregate: `/q/health`
- Liveness: `/q/health/live`
- Readiness: `/q/health/ready`
- Startup: `/q/health/started`
- Group root: `/q/health/group`
- Health UI: `/q/health-ui/` in dev and test by default

When the management interface is enabled, these paths move under the management server unless health is excluded from it.

## Liveness check

```java
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;
import org.eclipse.microprofile.health.Liveness;

@Liveness
@ApplicationScoped
public class ProcessHealthCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.up("process-alive");
    }
}
```

Use liveness for local process invariants only.

## Readiness check

```java
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;
import org.eclipse.microprofile.health.HealthCheckResponseBuilder;
import org.eclipse.microprofile.health.Readiness;

@Readiness
@ApplicationScoped
public class DownstreamReadinessCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        HealthCheckResponseBuilder response = HealthCheckResponse.named("orders-api");

        try {
            client.ping();
            response.up();
        } catch (Exception e) {
            response.down().withData("error", e.getMessage());
        }

        return response.build();
    }
}
```

Prefer a cheap dependency signal such as a ping, pool state, or cached status over a full business transaction.

## Startup check

```java
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;
import org.eclipse.microprofile.health.Startup;

@Startup
@ApplicationScoped
public class BootstrapHealthCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        return initializationComplete
                ? HealthCheckResponse.up("bootstrap")
                : HealthCheckResponse.down("bootstrap");
    }
}
```

Import `org.eclipse.microprofile.health.Startup`, not `io.quarkus.runtime.Startup`.

## Group a check

```java
import io.smallrye.health.api.HealthGroup;
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;
import org.eclipse.microprofile.health.Readiness;

@Readiness
@HealthGroup("dependencies")
@ApplicationScoped
public class BrokerReadinessCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.up("broker");
    }
}
```

Grouped checks are exposed under `/q/health/group/<group-name>` by default.

## Multiple groups for one check

```java
import io.smallrye.health.api.HealthGroup;
import io.smallrye.health.api.HealthGroups;

@Readiness
@HealthGroups({
    @HealthGroup("core"),
    @HealthGroup("dependencies")
})
@ApplicationScoped
public class SharedDependencyCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.up("shared-dependency");
    }
}
```

Use groups when different platforms or operators need different readiness slices.

## Reactive check

```java
import io.smallrye.health.api.AsyncHealthCheck;
import io.smallrye.mutiny.Uni;
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.health.HealthCheckResponse;
import org.eclipse.microprofile.health.Readiness;

@Readiness
@ApplicationScoped
public class ReactiveDependencyCheck implements AsyncHealthCheck {
    @Override
    public Uni<HealthCheckResponse> call() {
        return client.ping()
                .map(ignored -> HealthCheckResponse.up("reactive-dependency"));
    }
}
```

Use this when the dependency API is already reactive; do not wrap slow blocking code just to make the probe look reactive.

## Observe health state changes

```java
import io.smallrye.health.api.event.HealthStatusChangeEvent;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.event.Observes;
import org.eclipse.microprofile.health.Readiness;

@ApplicationScoped
public class HealthEvents {
    void onReadinessChange(@Observes @Readiness HealthStatusChangeEvent event) {
        auditService.record(event.getStatus());
    }
}
```

Keep observers non-blocking because they may run on the event loop.

## Extension health checks

Many Quarkus extensions contribute health checks automatically. Common examples include datasource and messaging integrations.

- These checks usually participate in readiness.
- They appear in `/q/health` and the specific type endpoint they belong to.
- They can be disabled globally with `quarkus.smallrye-health.extensions.enabled=false`.

Prefer selective replacement over blanket disabling; otherwise you can hide real dependency failures.

## Health UI

Health UI ships with the extension and is enabled in dev and test by default.

- Default path: `/q/health-ui/`
- Intended for quick inspection, not for probe consumers
- Can be packaged in production with configuration

Use `configuration.md` for path and inclusion settings.
