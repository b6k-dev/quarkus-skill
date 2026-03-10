# Advanced ORM Configuration Reference

Use this file for selected high-value Hibernate ORM properties that matter once you move beyond the default persistence unit.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.hibernate-orm."name".datasource` | required for named unit | A named persistence unit must bind to a datasource |
| `quarkus.hibernate-orm."name".packages` | - | Entities should be attached to a specific named unit by package |
| `quarkus.hibernate-orm."name".active` | auto | A configured unit should be runtime-selectable or disabled by default |
| `quarkus.hibernate-orm.persistence-xml.ignore` | `false` | A transitive `persistence.xml` is present but should be ignored |
| `quarkus.hibernate-orm.mapping-files` | auto-detect `META-INF/orm.xml` | XML mappings should be loaded explicitly or disabled with `no-file` |
| `quarkus.hibernate-orm.multitenant` | `NONE` | A persistence unit uses `SCHEMA`, `DATABASE`, or `DISCRIMINATOR` multitenancy |
| `quarkus.hibernate-orm.datasource` | default datasource | Database multitenancy needs one datasource as the dialect/version reference |
| `quarkus.hibernate-orm.database.start-offline` | `false` | App startup must not connect to the database |
| `quarkus.hibernate-orm.cache."region".memory.object-count` | `10000` | A cache region needs a tighter or larger entry bound |
| `quarkus.hibernate-orm.cache."region".expiration.max-idle` | `100S` | Cached data should expire sooner or remain warm longer |
| `quarkus.hibernate-orm.metrics.enabled` | `false` | Micrometer should expose Hibernate ORM metrics |
| `quarkus.hibernate-orm.request-scoped.enabled` | `true` | Read-only request-scoped session access without transactions should be disabled |

## Named persistence units

```properties
quarkus.datasource."users".db-kind=postgresql
quarkus.datasource."users".jdbc.url=jdbc:postgresql://localhost:5432/users

quarkus.hibernate-orm."users".datasource=users
quarkus.hibernate-orm."users".packages=org.acme.user,org.acme.shared
quarkus.hibernate-orm."users".schema-management.strategy=validate
```

Notes:

- Quote the unit name when using dotted property syntax.
- For named units, `datasource` is mandatory.
- You can point multiple persistence units at the same datasource.
- You can target the default datasource with `<default>` when needed.

## Runtime activation flags

```properties
quarkus.hibernate-orm."pg".active=false
quarkus.datasource."pg".active=false

quarkus.hibernate-orm."oracle".active=false
quarkus.datasource."oracle".active=false
```

Enable exactly one at runtime with config profiles, environment variables, or deployment-specific config.

```properties
%pg.quarkus.hibernate-orm."pg".active=true
%pg.quarkus.datasource."pg".active=true

%oracle.quarkus.hibernate-orm."oracle".active=true
%oracle.quarkus.datasource."oracle".active=true
```

Keep datasource and persistence unit activation in sync.

## `persistence.xml` and XML mapping

Ignore an unwanted `persistence.xml`:

```properties
quarkus.hibernate-orm.persistence-xml.ignore=true
```

Register mapping files explicitly:

```properties
quarkus.hibernate-orm.mapping-files=META-INF/orm.xml,META-INF/legacy/order.hbm.xml
```

Disable the implicit `META-INF/orm.xml` pickup:

```properties
quarkus.hibernate-orm.mapping-files=no-file
```

Do not mix `persistence.xml` with `quarkus.hibernate-orm.*` runtime configuration for the same app.

## Multitenancy modes

Schema multitenancy:

```properties
quarkus.hibernate-orm.multitenant=SCHEMA
quarkus.hibernate-orm.schema-management.strategy=none
```

Database multitenancy:

```properties
quarkus.hibernate-orm.multitenant=DATABASE
quarkus.hibernate-orm.datasource=base
quarkus.hibernate-orm.schema-management.strategy=none
```

Discriminator multitenancy:

```properties
quarkus.hibernate-orm.multitenant=DISCRIMINATOR
```

Guidance:

- Use `SCHEMA` when tenants share one database but isolate data by schema.
- Use `DATABASE` when each tenant has a dedicated datasource/database.
- Use `DISCRIMINATOR` when isolation is row-level and entities can carry `@TenantId`.
- For `SCHEMA` and `DATABASE`, plan on external schema management such as Flyway.

## Offline startup

```properties
quarkus.hibernate-orm.database.start-offline=true
```

With offline startup:

- Hibernate ORM skips connecting to the database during boot.
- Version checks against the live database do not run at startup.
- Automatic schema creation is not available; the schema must already exist.

Dialect-specific tuning can still be set per persistence unit when boot is offline.

## Cache region tuning

```properties
quarkus.hibernate-orm.cache."org.acme.Country".memory.object-count=2000
quarkus.hibernate-orm.cache."org.acme.Country".expiration.max-idle=30M
quarkus.hibernate-orm.cache."default-query-results-region".expiration.max-idle=5M
```

Notes:

- Region names containing dots must stay quoted.
- Entity regions use the entity FQCN.
- Collection regions use `OwnerEntity#collectionField`.
- Query cache uses `default-query-results-region` unless customized by Hibernate.

## Metrics and diagnostics

```properties
quarkus.hibernate-orm.metrics.enabled=true
```

This exposes Hibernate ORM metrics when a metrics extension such as Micrometer is present.

## Transaction interaction reminder

```properties
quarkus.hibernate-orm.request-scoped.enabled=false
```

Set this when you want to forbid convenience read access to request-scoped sessions outside explicit transactions.

## See Also

- [../data-migrations/README.md](../data-migrations/README.md) - Migration tooling for multitenancy and offline startup
- [../configuration/README.md](../configuration/README.md) - Profiles and runtime configuration sourcing
