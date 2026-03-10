# Advanced ORM API Reference

Use this file for advanced Hibernate ORM entry points in Quarkus with short, copy-ready examples.

## `@PersistenceUnit` injection

```java
@Inject
@io.quarkus.hibernate.orm.PersistenceUnit("users")
EntityManager em;
```

The same qualifier works for `Session`, `SessionFactory`, `EntityManagerFactory`, `CriteriaBuilder`, `Metamodel`, and cache entry points.

## Package attachment for named units

```properties
quarkus.hibernate-orm."users".datasource=users
quarkus.hibernate-orm."users".packages=org.acme.user,org.acme.shared
```

Alternative package-level attachment:

```java
@io.quarkus.hibernate.orm.PersistenceUnit("users")
package org.acme.user;
```

Do not mix `packages` and package-level `@PersistenceUnit`.

## Dynamic lookup for inactive units

```java
@Inject
@Any
InjectableInstance<Session> sessions;

Session activeSession() {
    return sessions.getActive();
}
```

Use this instead of static injection when `quarkus.hibernate-orm."name".active` may be `false`.

## `@PersistenceUnitExtension`

Bind advanced Hibernate components to one persistence unit:

```java
@PersistenceUnitExtension
class DefaultUnitExtension {
}

@PersistenceUnitExtension("users")
class UsersUnitExtension {
}
```

Supported Quarkus-registered extension bean types include:

- `org.hibernate.Interceptor`
- `org.hibernate.resource.jdbc.spi.StatementInspector`
- `org.hibernate.type.format.FormatMapper`
- `io.quarkus.hibernate.orm.runtime.tenant.TenantResolver`
- `io.quarkus.hibernate.orm.runtime.tenant.TenantConnectionResolver`
- `org.hibernate.boot.model.FunctionContributor`
- `org.hibernate.boot.model.TypeContributor`

## Tenant resolution

```java
@RequestScoped
@PersistenceUnitExtension
class RequestTenantResolver implements TenantResolver {
    @Override
    public String getDefaultTenantId() {
        return "base";
    }

    @Override
    public String resolveTenantId() {
        return "acme";
    }
}
```

Use `CurrentVertxRequest` rather than directly injecting `RoutingContext` if resolution must also work outside HTTP request handling.

## Programmatic tenant connections

```java
@ApplicationScoped
@PersistenceUnitExtension
class DynamicTenantConnections implements TenantConnectionResolver {
    @Override
    public ConnectionProvider resolve(String tenantId) {
        return createProviderFor(tenantId);
    }
}
```

Use this when tenant JDBC details are not fixed at build time.

## Interceptor and statement inspector

```java
@PersistenceUnitExtension
class AuditInterceptor implements Interceptor, Serializable {
}

@PersistenceUnitExtension
class SqlTagInspector implements StatementInspector {
    @Override
    public String inspect(String sql) {
        return sql + " /* users */";
    }
}
```

`@ApplicationScoped` interceptors must be thread-safe. Use `@Dependent` only when you need one instance per entity manager.

## Functions, types, and format mappers

```java
@ApplicationScoped
@PersistenceUnitExtension
class CustomFunctions implements FunctionContributor {
    @Override
    public void contributeFunctions(FunctionContributions fc) {
    }
}

@ApplicationScoped
@PersistenceUnitExtension
class CustomTypes implements TypeContributor {
    @Override
    public void contribute(TypeContributions tc, ServiceRegistry sr) {
    }
}

@JsonFormat
@PersistenceUnitExtension
class CustomJsonMapper implements FormatMapper {
}
```

For XML use `@XmlFormat`. Each persistence unit can have at most one JSON mapper and one XML mapper.

## `persistence.xml` and XML mappings

Use `META-INF/persistence.xml` mainly for migration:

```xml
<persistence-unit name="default">
    <mapping-file>META-INF/orm.xml</mapping-file>
</persistence-unit>
```

Or register mappings in Quarkus config:

```properties
quarkus.hibernate-orm.mapping-files=META-INF/orm.xml,META-INF/legacy/user.hbm.xml
```

`META-INF/orm.xml` is auto-included unless disabled with `no-file`.

## Cache entry points

```java
@Entity
@Cacheable
class Country {
    @OneToMany
    @org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
    List<City> cities;
}
```

```java
query.setHint("org.hibernate.cacheable", Boolean.TRUE);
```

Use `@Cacheable` for entities and query hints for query cache.

## Static metamodel and Jakarta Data

Both require the `org.hibernate.orm:hibernate-processor` annotation processor.

```java
var cb = session.getCriteriaBuilder();
var q = cb.createQuery(MyEntity.class);
var root = q.from(MyEntity.class);
q.where(cb.equal(root.get(MyEntity_.name), name));
```

```java
@Repository(dataStore = "users")
interface UserRepository extends CrudRepository<User, Long> {
}
```

Use `dataStore` for non-default persistence units.

## Envers, Spatial, and metrics

- Add `io.quarkus:quarkus-hibernate-envers` for auditing.
- Add `org.hibernate.orm:hibernate-spatial` for spatial support.
- Enable `quarkus.hibernate-orm.metrics.enabled=true` with a metrics extension for ORM metrics.

## See Also

- [../data-orm/README.md](../data-orm/README.md) - Base ORM lifecycle rules
- [../dependency-injection/README.md](../dependency-injection/README.md) - CDI lookup and qualifier patterns
