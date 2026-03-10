# Quarkus Data Panache Gotchas

Common Panache pitfalls, symptoms, and fixes.

## Transactions and writes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `persist()`, `delete()`, or `update()` fails or behaves inconsistently | Write operation runs outside a transaction | Put `@Transactional` on the application boundary or delegated service method |
| Data appears changed in memory but not yet in SQL | Hibernate flush is deferred until commit or before a query | Use `flush()` or `persistAndFlush()` only when early feedback is required |

## Streams and query loading

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `streamAll()` or `stream()` throws or warns about resource handling | Stream query is used without a transaction or not closed | Wrap the call in a transaction and use try-with-resources |
| Large table causes memory pressure | `listAll()` or broad `list()` loads too much data | Switch to `find()` with paging or a projection |

## Persistence-unit and model boundaries

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Entity does not work with multiple persistence units the way expected | A Panache entity can belong to only one persistence unit | Split the model per unit or drop to lower-level ORM patterns where needed |
| Feature is hard to follow because queries live in several places | Active record and repository styles are mixed carelessly | Pick one dominant style per feature and use the other only with a clear reason |

## Projection pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `project(Class)` selects the wrong constructor or fails unexpectedly | DTO has multiple constructors and none is explicitly marked | Use a single constructor, or annotate the intended one with `@ProjectedConstructor` |
| Nested value is null or not selected | Projection needs an explicit path | Use `@ProjectedFieldName` for referenced fields |
| Projection works on JVM but fails in native mode | DTO is not retained for reflection | Add `@RegisterForReflection` to the projection class |

## Query shorthand pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Simplified query does something unexpected | Shorthand expansion differs from the intended HQL | Use full HQL when the short form is no longer obvious |
| Paging helpers fail after using `range()` | Range and page state are mutually exclusive on one query | Re-apply `page(...)` or stay with one navigation style |

These issues usually come from Panache convenience features sitting on top of normal Hibernate ORM rules.
