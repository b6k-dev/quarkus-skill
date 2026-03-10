# Quarkus Data Migrations Configuration Reference

Use this file for high-value Flyway settings in Quarkus.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.flyway.migrate-at-start` | `false` | The app should apply pending migrations during startup |
| `quarkus.flyway.validate-at-start` | `false` | Startup should fail fast when migration history does not match scripts |
| `quarkus.flyway.repair-at-start` | `false` | History metadata must be repaired before migrate runs |
| `quarkus.flyway.baseline-on-migrate` | `false` | You are onboarding an existing non-empty schema into Flyway |
| `quarkus.flyway.baseline-at-start` | `false` | Startup should create a baseline even before an explicit migrate call |
| `quarkus.flyway.baseline-version` | - | The first tracked version must be something other than `1` |
| `quarkus.flyway.baseline-description` | - | The baseline row should use an explicit label |
| `quarkus.flyway.locations` | `db/migration` | Migrations live outside the default classpath folder |
| `quarkus.flyway.schemas` | datasource default | Flyway should manage one or more explicit schemas |
| `quarkus.flyway.table` | `flyway_schema_history` | The schema history table needs a custom name |
| `quarkus.flyway.clean-at-start` | `false` | Dev/test should rebuild schema from migrations on every restart |
| `quarkus.flyway.clean-disabled` | `false` | Clean operations must be blocked for safety |

## Automatic startup migration

```properties
quarkus.flyway.migrate-at-start=true
quarkus.flyway.validate-at-start=true
```

This is the simplest Quarkus-first setup for local development and many small services.

## Baseline existing schemas

```properties
quarkus.flyway.baseline-on-migrate=true
quarkus.flyway.baseline-version=1.0.0
quarkus.flyway.baseline-description=Initial production baseline
```

`baseline-on-migrate` only matters when the schema is non-empty and history is not initialized.

If you need Quarkus startup itself to perform the baseline step, add:

```properties
quarkus.flyway.baseline-at-start=true
```

## Repair and validation on startup

```properties
quarkus.flyway.repair-at-start=true
quarkus.flyway.validate-at-start=true
```

Use `repair-at-start` sparingly. It is a recovery tool, not a normal steady-state setting.

## Custom locations and schemas

```properties
quarkus.flyway.locations=db/migration,db/common
quarkus.flyway.schemas=app,app_audit
quarkus.flyway.table=app_flyway_history
```

When `schemas` is set, the first schema becomes the default managed schema and holds the history table.

## Named datasource configuration

Flyway keys follow the datasource name:

```properties
quarkus.datasource.users.db-kind=postgresql
quarkus.datasource.users.jdbc.url=jdbc:postgresql://localhost:5432/users
quarkus.datasource.users.username=users
quarkus.datasource.users.password=users

quarkus.flyway.users.locations=db/users
quarkus.flyway.users.migrate-at-start=true
quarkus.flyway.users.schemas=users_app
```

The general shape is:

```text
quarkus.flyway.<datasource-name>.<property>
```

## Dev/test cleanup profile

```properties
%dev.quarkus.flyway.clean-at-start=true
%test.quarkus.flyway.clean-at-start=true
%prod.quarkus.flyway.clean-at-start=false
```

Use profiles so destructive cleanup never leaks into production.

## Reactive datasource note

Flyway uses JDBC internally.

If the application otherwise uses reactive SQL clients, configure the same datasource for JDBC as well and include the JDBC driver extension needed by Flyway.

## Kubernetes init-job note

When generating Kubernetes or OpenShift manifests, Quarkus can externalize Flyway startup into an initialization `Job` so every replica does not run migrations itself.

Disable the default behavior only when your rollout process already handles migrations elsewhere:

```properties
quarkus.kubernetes.init-task-defaults.enabled=false
quarkus.openshift.init-task-defaults.enabled=false
```

## See Also

- [./api.md](./api.md) - Dependency setup, `Flyway` injection, and customizers
- [./patterns.md](./patterns.md) - Practical configuration combinations by environment
- [../configuration/README.md](../configuration/README.md) - Profiles and deployment-specific property organization
