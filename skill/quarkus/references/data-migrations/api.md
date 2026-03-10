# Quarkus Data Migrations API Reference

Use this file for Flyway entry points and short, copy-ready examples.

## Extension entry points

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-flyway</artifactId>
</dependency>

<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jdbc-postgresql</artifactId>
</dependency>
```

Add a database-specific Flyway module when your database requires it, for example:

- `org.flywaydb:flyway-database-postgresql`
- `org.flywaydb:flyway-mysql`
- `org.flywaydb:flyway-database-oracle`

## Default migration location

Place migrations in:

```text
src/main/resources/db/migration
```

Quarkus scans `db/migration` by default.

## Versioned SQL naming

Flyway versioned SQL migrations use this structure:

```text
V1__create_tables.sql
V1.1__add_status_column.sql
V20260310_1200__backfill_reference_data.sql
```

Pattern:

```text
<prefix><version><separator><description><suffix>
```

Default pieces:

- Prefix: `V`
- Separator: `__`
- Suffix: `.sql`

Repeatable migrations use `R__description.sql`.

## Minimal setup

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/app
quarkus.datasource.username=app
quarkus.datasource.password=app

quarkus.flyway.migrate-at-start=true
```

With `V1__init.sql` in `db/migration`, Quarkus runs Flyway during startup.

## Inject `Flyway`

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import org.flywaydb.core.Flyway;

@ApplicationScoped
public class MigrationService {
    @Inject
    Flyway flyway;

    public String currentVersion() {
        return flyway.info().current().getVersion().toString();
    }
}
```

Inject the default datasource Flyway instance with plain `@Inject Flyway`.

## Run migrations programmatically

```java
void migrateNow() {
    flyway.migrate();
}
```

Use this when migration timing is controlled by application logic instead of `migrate-at-start`.

## Repair programmatically

```java
void repairHistory() {
    flyway.repair();
}
```

`repair()` is useful after failed migrations on databases without transactional DDL or when checksums need history cleanup.

## Inspect migration state

```java
import org.flywaydb.core.api.MigrationInfo;

MigrationInfo current = flyway.info().current();
MigrationInfo[] pending = flyway.info().pending();
```

Use `info()` to report current version, pending scripts, and validation state.

## Named datasource injection

```java
import jakarta.inject.Inject;
import io.quarkus.flyway.FlywayDataSource;
import org.flywaydb.core.Flyway;

class MultiDatasourceMigrations {
    @Inject
    @FlywayDataSource("inventory")
    Flyway inventoryFlyway;
}
```

Quarkus also exposes named Flyway beans such as `@Named("flyway_inventory")`, but `@FlywayDataSource` is clearer.

## Customize Flyway configuration

```java
import jakarta.inject.Singleton;
import io.quarkus.flyway.FlywayConfigurationCustomizer;
import org.flywaydb.core.api.configuration.FluentConfiguration;

@Singleton
public class MigrationCustomizer implements FlywayConfigurationCustomizer {
    @Override
    public void customize(FluentConfiguration configuration) {
        configuration.connectRetries(10);
    }
}
```

Use a customizer when Quarkus does not expose the exact Flyway option you need.

## Customizer for a named datasource

```java
import jakarta.inject.Singleton;
import io.quarkus.flyway.FlywayConfigurationCustomizer;
import io.quarkus.flyway.FlywayDataSource;
import org.flywaydb.core.api.configuration.FluentConfiguration;

@Singleton
@FlywayDataSource("users")
public class UsersFlywayCustomizer implements FlywayConfigurationCustomizer {
    @Override
    public void customize(FluentConfiguration configuration) {
        configuration.defaultSchema("users_app");
    }
}
```

## See Also

- [./configuration.md](./configuration.md) - Startup, baseline, validation, and datasource-scoped properties
- [./patterns.md](./patterns.md) - Dev workflow, Hibernate handoff, and rollout patterns
- [./gotchas.md](./gotchas.md) - Common failure modes before editing history or baselines
