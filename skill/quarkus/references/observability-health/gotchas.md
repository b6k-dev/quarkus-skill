# Quarkus Health Gotchas

Common SmallRye Health and management-placement pitfalls, symptoms, and fixes.

## Probe design

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Pods restart during a database outage | A dependency check was placed in liveness | Move external dependency validation to readiness |
| Traffic reaches the app before bootstrap is complete | Startup or readiness does not reflect initialization state | Add a startup check or gate readiness until bootstrap finishes |
| Health endpoint becomes slow or flaky under load | Probe performs expensive queries, fan-out, or blocking work | Keep checks cheap, deterministic, and narrowly scoped |
| Health payload leaks internal details | Probe adds exception messages, stack traces, or secrets to `withData` | Return stable operator-friendly reason codes instead |

## Groups and custom checks

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Group endpoint returns fewer checks than expected | Checks were not annotated with the intended health group | Add `@HealthGroup` or `@HealthGroups` to the relevant checks |
| A custom check cannot be disabled by config | The class-specific enablement key is missing or misspelled | Use `quarkus.smallrye-health.check."check-classname".enabled` with the fully qualified check class name |
| Dependency failures disappear after a config change | Extension checks were disabled globally | Re-enable `quarkus.smallrye-health.extensions.enabled` or replace the missing checks explicitly |

## Health UI and paths

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Health UI works in dev but is missing in production | `quarkus.smallrye-health.ui.always-include` defaults to `false` | Set it to `true` before building and secure access intentionally |
| Health or Health UI path is not where expected | Relative paths resolve under different roots on the app vs management server | Verify `quarkus.smallrye-health.*-path` and `quarkus.management.root-path`, or use an absolute path when needed |
| Setting Health UI path to `/` breaks routing | `/` is not a valid UI root path | Use a dedicated path such as `health-ui` or `/ops/health-ui` |

## Management interface

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Packaged app ignores a new management listener setting | `quarkus.management.enabled` is build-time | Rebuild the application after changing it |
| Probes are still on the main port after enabling management | `quarkus.smallrye-health.management.enabled=false` or health extension is not present | Enable health management placement or verify the extension is installed |
| Expected management path includes the main HTTP root | `quarkus.http.root-path` does not affect the management server | Use `quarkus.management.root-path` for management endpoints |
| HTTPS management endpoint rejects plain HTTP checks | The management interface serves either HTTP or HTTPS, not both | Point the probe to the configured scheme and port |

## Imports and runtime behavior

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Startup check never behaves as expected | `io.quarkus.runtime.Startup` was imported instead of the MicroProfile annotation | Import `org.eclipse.microprofile.health.Startup` |
| Health observer code causes event-loop warnings | Observer performs blocking work during health status change events | Keep observers non-blocking or hand off work explicitly |
