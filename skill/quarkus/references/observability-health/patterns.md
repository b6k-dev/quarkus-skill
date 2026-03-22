# Quarkus Health Usage Patterns

Use these patterns for repeatable SmallRye Health and probe-design workflows.

## Pattern: Add health endpoints to an existing app

When to use:

- You want Kubernetes-style probes without building a custom endpoint.

Command:

```bash
quarkus extension add smallrye-health
```

Verify with `/q/health`, `/q/health/live`, and `/q/health/ready`.

## Pattern: Keep liveness process-only

When to use:

- The app depends on external systems that may fail independently of the JVM process.

Example:

```java
@Liveness
@ApplicationScoped
public class EventLoopHealthCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.up("event-loop");
    }
}
```

Do not call databases, brokers, or remote HTTP services from liveness unless failure should trigger a restart.

## Pattern: Put downstream dependencies in readiness

When to use:

- Traffic should stop when a required dependency is unavailable.

Example:

```java
@Readiness
@ApplicationScoped
public class DatabaseReadinessCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        return pool.isHealthy()
                ? HealthCheckResponse.up("database")
                : HealthCheckResponse.down("database");
    }
}
```

Prefer a cheap capability signal over a full query path.

## Pattern: Delay traffic during bootstrap with startup

When to use:

- Application startup includes migrations, cache warmup, model loading, or other one-time initialization.

Example:

```java
@Startup
@ApplicationScoped
public class WarmupStartupCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        return warmupState.ready()
                ? HealthCheckResponse.up("warmup")
                : HealthCheckResponse.down("warmup");
    }
}
```

Use startup to protect bootstrap only; readiness should still represent steady-state dependency health afterward.

## Pattern: Return useful failure details without leaking internals

When to use:

- Operators need a fast hint about why readiness is DOWN.

Example:

```java
@Readiness
@ApplicationScoped
public class BrokerReadinessCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        try {
            broker.ping();
            return HealthCheckResponse.up("broker");
        } catch (Exception e) {
            return HealthCheckResponse.named("broker")
                    .down()
                    .withData("reason", "unreachable")
                    .build();
        }
    }
}
```

Use stable reason codes instead of stack traces or credentials.

## Pattern: Split checks into operator-friendly groups

When to use:

- Platform automation needs one dependency slice and humans need another.

Example:

```java
import io.smallrye.health.api.HealthGroup;
import io.smallrye.health.api.HealthGroups;

@Readiness
@HealthGroups({
    @HealthGroup("core"),
    @HealthGroup("dependencies")
})
@ApplicationScoped
public class SearchClusterCheck implements HealthCheck {
    @Override
    public HealthCheckResponse call() {
        return HealthCheckResponse.up("search");
    }
}
```

```properties
quarkus.smallrye-health.group-path=group
```

This exposes grouped views such as `/q/health/group/core`.

## Pattern: Keep extension health checks and add custom coverage

When to use:

- Built-in datasource or messaging checks cover only part of the readiness story.

Example:

```properties
quarkus.smallrye-health.extensions.enabled=true
```

Add custom checks for app-level prerequisites such as schema version, remote API reachability, or cache priming instead of replacing extension checks blindly.

## Pattern: Package Health UI in production intentionally

When to use:

- Operators need a browser view for non-production-like support environments.

Example:

```properties
quarkus.smallrye-health.ui.always-include=true
quarkus.smallrye-health.ui.root-path=ops/health-ui
```

Treat UI access as an operational surface; secure it and avoid exposing it on a public interface.

## Pattern: Move probes to the management interface

When to use:

- Health endpoints should not share the app listener, path space, or security posture.

Example:

```properties
quarkus.management.enabled=true
quarkus.management.host=127.0.0.1
quarkus.management.port=9000
quarkus.management.root-path=/q
quarkus.smallrye-health.management.enabled=true
```

With default health paths, readiness moves to `http://127.0.0.1:9000/q/health/ready`.

## Pattern: Keep health on the app server even when management is enabled

When to use:

- You want a management server for other endpoints but probes must remain on the main app port.

Example:

```properties
quarkus.management.enabled=true
quarkus.smallrye-health.management.enabled=false
```

Use this only when your platform expects probes on the main server.
