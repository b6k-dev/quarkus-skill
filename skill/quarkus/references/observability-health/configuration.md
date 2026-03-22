# Quarkus Health Configuration Reference

Use this file for corrected, high-value SmallRye Health and management-interface configuration keys.

## Rules that matter first

- `quarkus.management.enabled` is build-time; rebuild packaged apps after changing it.
- Health endpoints live on the main HTTP server by default and move to the management server when both the management interface and `quarkus.smallrye-health.management.enabled` are enabled.
- Relative health paths resolve under the non-application root on the main server, and under `quarkus.management.root-path` on the management server.
- Health UI is packaged only in dev and test by default; production inclusion requires `quarkus.smallrye-health.ui.always-include=true`.
- Prefer leaving extension checks enabled unless you intentionally replace them.

## Core health endpoints

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.smallrye-health.enabled` | `true` | All health endpoints must be disabled entirely |
| `quarkus.smallrye-health.root-path` | `health` | The aggregate health root should use a custom path |
| `quarkus.smallrye-health.liveness-path` | `live` | The liveness sub-path should be renamed |
| `quarkus.smallrye-health.readiness-path` | `ready` | The readiness sub-path should be renamed |
| `quarkus.smallrye-health.startup-path` | `started` | The startup sub-path should be renamed |
| `quarkus.smallrye-health.group-path` | `group` | Group endpoints should use a custom segment |
| `quarkus.smallrye-health.default-health-group` | - | Ungrouped checks should appear in a default named group |
| `quarkus.smallrye-health.max-group-registries-count` | implementation default | Group creation should be capped explicitly |

## Behavior and payload shaping

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.smallrye-health.extensions.enabled` | `true` | Extension-provided checks should be disabled globally |
| `quarkus.smallrye-health.check."check-classname".enabled` | `false` | A specific custom health check should be toggled by config |
| `quarkus.smallrye-health.additional.property."property-name"` | - | Extra top-level JSON metadata should be returned with every response |
| `quarkus.smallrye-health.include-problem-details` | `false` | DOWN responses should use RFC 9457 Problem Details style |
| `quarkus.smallrye-health.openapi.included` | `false` | Liveness and readiness endpoints should appear in generated OpenAPI |

Avoid putting secrets or verbose diagnostics into health payloads.

## Health UI

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.smallrye-health.ui.enabled` | `true` | Included Health UI should still be switchable on or off |
| `quarkus.smallrye-health.ui.root-path` | `health-ui` | Health UI should live at a custom path |
| `quarkus.smallrye-health.ui.always-include` | `false` | Health UI must be packaged in production |

Deprecated key to avoid: `quarkus.smallrye-health.ui.enable`.

## Management placement

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.management.enabled` | `false` | Health endpoints should move to a separate server and port |
| `quarkus.smallrye-health.management.enabled` | `true` | Management is enabled but health should stay off the management server |
| `quarkus.management.root-path` | `/q` | Management endpoints need a different shared root |
| `quarkus.management.host` | environment-specific | The management server should bind to a specific interface |
| `quarkus.management.port` | `9000` | The management server should listen on a custom port |
| `quarkus.management.test-port` | `9001` | Tests should use a different management port |
| `quarkus.management.tls-configuration-name` | - | The management server should use a named TLS configuration |
| `quarkus.management.auth.enabled` | derived | The management interface should require authentication |

If `quarkus.smallrye-health.root-path` starts with `/`, it becomes absolute and ignores the management root path.

## Build-time reminders

- `quarkus.management.enabled` is build-time fixed.
- Rebuild packaged applications after changing management-interface enablement.
- Health path changes are runtime-configurable, but their effective URL still depends on whether management is enabled.

## See Also

- [Patterns](./patterns.md) - repeatable health probe and management-interface workflows.
