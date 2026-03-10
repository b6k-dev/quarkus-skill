# Quarkus Data ORM Gotchas

Common Hibernate ORM pitfalls, symptoms, and fixes.

## Transactions and write paths

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `persist`, `remove`, or dirty updates fail or do not commit | Write method is not inside a transaction | Put the write path behind a CDI method annotated with `@Transactional` |
| ORM behavior feels inconsistent across layers | Transaction boundary is buried deep or split awkwardly | Put transactional boundaries at clear entry methods such as REST or service methods |

## Configuration clashes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Application fails at startup with ORM configuration errors | `persistence.xml` is mixed with `quarkus.hibernate-orm.*` properties | Pick one configuration style; set `quarkus.hibernate-orm.persistence-xml.ignore=true` if a stray XML file must be ignored |
| SQL is worse than expected or startup fails on version checks | `db-version` is missing, too low, or higher than a real target DB | Set `quarkus.datasource.db-version` to a realistic highest common version |

## Schema strategy mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Local dev data disappears after reload | `drop-and-create` rebuilds the schema each startup | Use it only for disposable dev/test data, or switch to `update`/`none` |
| Production schema changes unexpectedly | `drop-and-create` or `update` leaked into non-dev profiles | Set `%prod.quarkus.hibernate-orm.schema-management.strategy=none` |

## `import.sql` surprises

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Seed script only partially loads or fails strangely | Statements in `import.sql` are missing semicolons | Terminate every SQL statement with `;` |
| Demo/test data does not load in production | Quarkus defaults `sql-load-script` to `no-file` outside `dev` and `test` | Set the property explicitly if production loading is intentional, or keep the safer default |

## Dialect and environment assumptions

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Custom dialect setting adds confusion for a normal PostgreSQL/MySQL setup | `quarkus.hibernate-orm.dialect` was copied from generic Hibernate examples unnecessarily | Remove the override and let Quarkus infer the dialect from the datasource |
| Dev profile works but production startup policy is unsafe | Dev-oriented schema or seed settings were not profile-scoped | Keep destructive settings under `%dev` or `%test` and review `%prod` explicitly |
