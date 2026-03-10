# Quarkus Data ORM Configuration Reference

Use this file for high-value Hibernate ORM settings that matter in standard Quarkus applications.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.hibernate-orm.schema-management.strategy` | environment-dependent | You need Quarkus/Hibernate to create, update, validate, or ignore schema changes |
| `quarkus.hibernate-orm.sql-load-script` | `import.sql` in `dev`/`test`, `no-file` otherwise | Seed SQL should load automatically or be disabled explicitly |
| `quarkus.datasource.db-version` | minimum supported version | Hibernate should target the actual database version for better SQL |
| `quarkus.hibernate-orm.dialect` | inferred from datasource when possible | You use an unsupported database or need a non-default dialect |
| `quarkus.hibernate-orm.database.version-check.enabled` | `true` | Startup should skip version validation because DB reachability is limited |
| `quarkus.hibernate-orm.persistence-xml.ignore` | `false` | A stray `persistence.xml` is on the classpath and Quarkus config should win |

## Schema management strategy

```properties
%dev.quarkus.hibernate-orm.schema-management.strategy=drop-and-create
%test.quarkus.hibernate-orm.schema-management.strategy=drop-and-create
%prod.quarkus.hibernate-orm.schema-management.strategy=none
```

Common values:

- `drop-and-create` - rebuild schema on startup; best for clean dev loops.
- `update` - best-effort schema update; acceptable for development, not production.
- `validate` - verify schema matches mappings without changing it.
- `none` - disable schema generation; safest production baseline.

## SQL seed loading

```properties
%dev.quarkus.hibernate-orm.sql-load-script=import.sql
%test.quarkus.hibernate-orm.sql-load-script=import.sql
%prod.quarkus.hibernate-orm.sql-load-script=no-file
```

Quarkus defaults to loading `/import.sql` only in `dev` and `test`; production defaults intentionally avoid accidental data resets.

## Database version

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.db-version=16.3
```

Set `quarkus.datasource.db-version` as high as possible without exceeding any real target database version. This lets Hibernate generate more efficient SQL and catches mismatches at startup.

## Explicit dialect only when needed

```properties
quarkus.datasource.db-kind=postgresql
quarkus.hibernate-orm.dialect=Cockroach
quarkus.datasource.db-version=25.1
```

For mainstream supported databases, let Quarkus infer the dialect. Set `quarkus.hibernate-orm.dialect` only for unsupported databases or intentional overrides.

## Important default behavior

- Dialect is usually inferred from the datasource.
- ORM configuration is usually done in `application.properties`, not `persistence.xml`.
- Mixing `persistence.xml` with `quarkus.hibernate-orm.*` configuration causes a startup failure.

Keep the explicit configuration surface small: set schema strategy, SQL loading, and database version first, then add dialect overrides only for real edge cases.

For production, treat `%prod.quarkus.hibernate-orm.schema-management.strategy=none` and `%prod.quarkus.hibernate-orm.sql-load-script=no-file` as the safe default starting point.
