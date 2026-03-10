# Quarkus Data ORM Usage Patterns

Use these patterns for repeatable plain Hibernate ORM workflows in Quarkus.

## Pattern: Basic CRUD service with `EntityManager`

When to use:

- You want straightforward JPA-style persistence without Panache.

Example:

```java
import java.util.List;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.persistence.EntityManager;
import jakarta.transaction.Transactional;

@ApplicationScoped
public class FruitService {
    @Inject
    EntityManager em;

    public List<Fruit> list() {
        return em.createQuery("select f from Fruit f order by f.name", Fruit.class)
            .getResultList();
    }

    public Fruit find(long id) {
        return em.find(Fruit.class, id);
    }

    @Transactional
    public Fruit create(String name) {
        Fruit fruit = new Fruit();
        fruit.name = name;
        em.persist(fruit);
        return fruit;
    }

    @Transactional
    public void delete(long id) {
        Fruit fruit = em.find(Fruit.class, id);
        if (fruit != null) {
            em.remove(fruit);
        }
    }
}
```

Keep write methods transactional; simple reads can stay non-transactional unless consistency rules require otherwise.

## Pattern: Dev mode with `drop-and-create` plus `import.sql`

When to use:

- You want live reload to rebuild schema and repopulate fixtures after entity changes.

Configuration:

```properties
%dev.quarkus.hibernate-orm.schema-management.strategy=drop-and-create
%dev.quarkus.hibernate-orm.sql-load-script=import.sql
```

`src/main/resources/import.sql`:

```sql
insert into Fruit(id, name) values (1, 'Apple');
insert into Fruit(id, name) values (2, 'Pear');
```

This is the fastest feedback loop for local development demos, prototyping, and tests with controlled fixture data.

## Pattern: Work against real data with `update` or `none`

When to use:

- You need a copy of production-like data while still iterating on entities.

Configuration:

```properties
%dev-with-data.quarkus.hibernate-orm.schema-management.strategy=update
%dev-with-data.quarkus.hibernate-orm.sql-load-script=no-file
```

Use `update` only as a development convenience. If you need full control over schema changes, switch to `none` and manage schema separately.

## Pattern: Production-safe baseline

When to use:

- The application should never mutate schema or load seed SQL on startup in production.

Configuration:

```properties
%prod.quarkus.hibernate-orm.schema-management.strategy=none
%prod.quarkus.hibernate-orm.sql-load-script=no-file
```

This is the baseline to reach before deployment. Add `validate` selectively when startup schema verification is useful and the database is already managed elsewhere.

## Pattern: Hand off schema management to Flyway

When to use:

- Early development used Hibernate schema generation, but environments now need reviewed migrations.

Approach:

1. Use Hibernate ORM schema generation during early development.
2. Generate or author an initial Flyway migration from the current schema.
3. Switch the application baseline to `schema-management.strategy=none`.
4. Let Flyway own future schema changes.

Quarkus Dev UI can help bootstrap the first Flyway migration from the Hibernate-generated schema, but the generated script still needs review.

## Pattern: Profile different persistence behaviors

When to use:

- Teams need fast local rebuilds in one profile and production-like data in another.

Configuration:

```properties
%dev.quarkus.hibernate-orm.schema-management.strategy=drop-and-create
%dev.quarkus.hibernate-orm.sql-load-script=import-dev.sql

%dev-with-data.quarkus.hibernate-orm.schema-management.strategy=update
%dev-with-data.quarkus.hibernate-orm.sql-load-script=no-file

%prod.quarkus.hibernate-orm.schema-management.strategy=none
%prod.quarkus.hibernate-orm.sql-load-script=no-file
```

This keeps the ORM model constant while changing environment behavior with profiles.
