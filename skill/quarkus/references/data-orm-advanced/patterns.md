# Advanced ORM Usage Patterns

Use these patterns for repeatable Hibernate ORM workflows that involve multiple persistence units, multitenancy, or Quarkus-specific extension points.

## Pattern: Split entities across multiple persistence units

When to use:

- Different model areas bind to different datasources.
- You want package-level ownership for entities.

```properties
quarkus.datasource."users".db-kind=postgresql
quarkus.datasource."inventory".db-kind=postgresql

quarkus.hibernate-orm."users".datasource=users
quarkus.hibernate-orm."users".packages=org.acme.users,org.acme.shared

quarkus.hibernate-orm."inventory".datasource=inventory
quarkus.hibernate-orm."inventory".packages=org.acme.inventory
```

```java
@Inject @PersistenceUnit("users") EntityManager usersEm;
@Inject @PersistenceUnit("inventory") EntityManager inventoryEm;
```

Attach dependent types consistently: embeddables, mapped superclasses, and shared model packages must follow the same unit split.

## Pattern: Select one active persistence unit at runtime

When to use:

- One artifact must target one of several predeclared backends.

```properties
quarkus.hibernate-orm."pg".packages=org.acme.model.shared
quarkus.hibernate-orm."pg".datasource=pg
quarkus.hibernate-orm."pg".active=false
quarkus.datasource."pg".active=false

quarkus.hibernate-orm."oracle".packages=org.acme.model.shared
quarkus.hibernate-orm."oracle".datasource=oracle
quarkus.hibernate-orm."oracle".active=false
quarkus.datasource."oracle".active=false

%pg.quarkus.hibernate-orm."pg".active=true
%pg.quarkus.datasource."pg".active=true
```

```java
@Inject @Any InjectableInstance<SessionFactory> sessionFactories;

SessionFactory get() {
    return sessionFactories.getActive();
}
```

Avoid direct injection of `Session` or `SessionFactory` for units that may start inactive.

## Pattern: Produce a default bean backed by the active named unit

When to use:

- Most code should inject one unqualified `Session`.

```java
@ApplicationScoped
class SessionProducer {
    @Inject @PersistenceUnit("pg") InjectableInstance<Session> pg;
    @Inject @PersistenceUnit("oracle") InjectableInstance<Session> oracle;

    @Produces
    @ApplicationScoped
    Session session() {
        if (pg.getHandle().getBean().isActive()) return pg.get();
        if (oracle.getHandle().getBean().isActive()) return oracle.get();
        throw new IllegalStateException("No active persistence unit");
    }
}
```

This centralizes backend selection in one CDI producer.

## Pattern: Schema multitenancy with Flyway-managed schemas

When to use:

- Tenants share one database and isolate by schema.

```properties
quarkus.hibernate-orm.multitenant=SCHEMA
quarkus.hibernate-orm.schema-management.strategy=none
quarkus.flyway.schemas=base,mycompany
quarkus.flyway.locations=classpath:schema
quarkus.flyway.migrate-at-start=true
```

```java
@RequestScoped
@PersistenceUnitExtension
class TenantFromPath implements TenantResolver {
    @Override public String getDefaultTenantId() { return "base"; }
    @Override public String resolveTenantId() { return "mycompany"; }
}
```

Do not rely on Hibernate schema generation here; provision every tenant schema through migrations.

## Pattern: Database multitenancy with named datasources

When to use:

- Tenants need separate databases and the list is fixed at build time.

```properties
quarkus.hibernate-orm.multitenant=DATABASE
quarkus.hibernate-orm.datasource=base
quarkus.hibernate-orm.schema-management.strategy=none

quarkus.datasource.base.db-kind=postgresql
quarkus.datasource.mycompany.db-kind=postgresql
```

```java
@RequestScoped
@PersistenceUnitExtension
class TenantFromHeader implements TenantResolver {
    @Override public String getDefaultTenantId() { return "base"; }
    @Override public String resolveTenantId() { return "mycompany"; }
}
```

Returned tenant ids must match datasource names such as `base` and `mycompany`.

## Pattern: Programmatic tenant connection resolution

When to use:

- Tenant JDBC details come from a registry or onboarding database.

```java
@ApplicationScoped
@PersistenceUnitExtension
class DynamicConnections implements TenantConnectionResolver {
    @Override
    public ConnectionProvider resolve(String tenantId) {
        return buildProviderFromRegistry(tenantId);
    }
}
```

Choose this only when config-defined datasources are too static; Quarkus integrations around datasource-based tenants may need manual replacement.

## Pattern: Add SQL diagnostics with a statement inspector

When to use:

- You need SQL tagging or tracing without touching repository code.

```java
@PersistenceUnitExtension("users")
class TraceInspector implements StatementInspector {
    @Override
    public String inspect(String sql) {
        return sql + " /* users-pu */";
    }
}
```

Use an interceptor instead when you need entity lifecycle hooks.

## Pattern: Start offline and let migrations own schema setup

When to use:

- The database is unreachable during app boot.

```properties
quarkus.hibernate-orm.database.start-offline=true
quarkus.hibernate-orm.schema-management.strategy=none
```

- Run Flyway or another external migration tool before the app serves traffic.
- Treat startup success as metadata boot, not live database validation.
- Set explicit dialect/version-related options when auto-detection cannot run.

## See Also

- [../data-migrations/README.md](../data-migrations/README.md) - Migration ownership for schemas and tenant provisioning
- [../dependency-injection/README.md](../dependency-injection/README.md) - Producer and dynamic lookup techniques for active-unit routing
