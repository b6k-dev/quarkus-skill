# Advanced ORM Gotchas

Common advanced Hibernate ORM pitfalls, symptoms, and fixes in Quarkus.
These usually appear only after moving beyond the default single-unit setup.

## Persistence unit activation

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| App fails at startup after adding `quarkus.hibernate-orm."name".active=false` | Code still statically injects `Session`, `SessionFactory`, `EntityManager`, or another bean from that inactive unit | Replace direct injection with `InjectableInstance<T>` or a producer that selects the active unit |
| Runtime selection works for datasource but not ORM | Datasource `active` flag and persistence unit `active` flag are not aligned | Activate or deactivate both together |
| Another extension still crashes startup when a unit is disabled | An extension consuming that persistence unit remains enabled | Deactivate or reconfigure the dependent extension too |

## Multiple persistence units

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Entities do not attach to the expected unit | `packages` and package-level `@PersistenceUnit` are mixed | Pick one attachment strategy; prefer `packages` in config |
| Shared embeddables or mapped superclasses behave inconsistently | Dependent model types were not attached to the same unit as the entity using them | Keep related model packages attached consistently |
| Panache model cannot be reused across multiple units | Panache entities are limited to one persistence unit | Use traditional Hibernate ORM entities for multi-unit reuse |

## Multitenancy

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Tenant schema or database is missing tables | Hibernate schema generation is not suitable for normal multitenancy flows | Set `schema-management.strategy=none` and provision schema with Flyway or another external migration tool |
| Database multitenancy behaves unpredictably across tenants | Datasources behind one persistence unit do not share vendor/version characteristics | Keep the same DB vendor/version family or split into separate persistence units |
| Tenant resolution works in REST requests but fails in jobs or async code | Resolver depends directly on `RoutingContext` | Use `CurrentVertxRequest` defensively or support non-HTTP execution paths |

## Caching and XML configuration

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Cached data goes stale across pods or after external writes | Quarkus second-level cache is local, not cluster-synchronized | Cache only immutable or stale-tolerant data; keep expectations explicit |
| Cache tuning property does not apply | Region name with dots was not quoted | Quote region names, for example `cache."org.acme.Country".*` |
| Startup fails after adding both `persistence.xml` and `quarkus.hibernate-orm.*` config | Quarkus does not support mixing the two approaches | Use one approach only, or set `quarkus.hibernate-orm.persistence-xml.ignore=true` |
| Unexpected mappings appear from `META-INF/orm.xml` | Quarkus auto-loads that file when present | Set `quarkus.hibernate-orm.mapping-files=no-file` or manage mappings explicitly |

## Quarkus and native-image limits

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| JMX statistics or management beans disappear in native builds | JMX support is disabled in GraalVM native mode | Use Micrometer or other supported metrics paths instead |
| Legacy ThreadLocal session binding does not work | `ThreadLocalSessionContext` is not implemented in Quarkus integration | Use CDI injection or programmatic CDI lookup |
| JNDI-based ORM wiring from legacy apps fails | Quarkus disables JNDI-based integration for Hibernate ORM | Inject datasources and transaction components directly |

## See Also

- [../dependency-injection/README.md](../dependency-injection/README.md) - CDI lookup and producer patterns for inactive beans
- [../data-migrations/README.md](../data-migrations/README.md) - Schema ownership when multitenancy disables normal ORM generation
