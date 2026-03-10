# Quarkus Data ORM API Reference

Use this file for plain Hibernate ORM runtime APIs with short, copy-ready examples.

## Extension entry point

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-hibernate-orm</artifactId>
</dependency>

<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jdbc-postgresql</artifactId>
</dependency>
```

Quarkus creates the ORM setup from the datasource plus the Hibernate ORM extension; `persistence.xml` is usually unnecessary.

## Minimal entity

```java
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;

@Entity
public class Fruit {
    @Id
    @GeneratedValue
    public Long id;

    public String name;
}
```

Use normal Jakarta Persistence annotations; Quarkus handles build-time enhancement automatically.

## Inject `EntityManager`

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.persistence.EntityManager;
import jakarta.transaction.Transactional;

@ApplicationScoped
public class FruitService {
    @Inject
    EntityManager em;

    @Transactional
    public void create(String name) {
        Fruit fruit = new Fruit();
        fruit.name = name;
        em.persist(fruit);
    }
}
```

For writes, make the CDI method a transaction boundary with `@Transactional`.

## Inject `Session`

```java
import jakarta.inject.Inject;
import jakarta.transaction.Transactional;
import org.hibernate.Session;

@ApplicationScoped
public class InventoryService {
    @Inject
    Session session;

    @Transactional
    public void rename(Long id, String name) {
        Fruit fruit = session.find(Fruit.class, id);
        fruit.name = name;
    }
}
```

Use `Session` when you want Hibernate-native APIs; otherwise `EntityManager` is fine.

## Transaction boundaries

```java
import jakarta.transaction.Transactional;

@Transactional
public void updateStock(Long id, int stock) {
    Fruit fruit = em.find(Fruit.class, id);
    fruit.stock = stock;
}
```

- Writes should run inside `@Transactional` methods.
- A transaction boundary is usually best at the application edge: REST resource, messaging consumer, or service entry method.
- Managed entities flush on commit.

## Basic persistence operations

```java
Fruit fruit = new Fruit();
fruit.name = "Apple";
em.persist(fruit);

Fruit byId = em.find(Fruit.class, fruit.id);

em.remove(byId);
```

## Basic query examples

```java
List<Fruit> fruits = em.createQuery(
        "select f from Fruit f order by f.name", Fruit.class)
    .getResultList();

Fruit fruit = em.createQuery(
        "select f from Fruit f where f.name = :name", Fruit.class)
    .setParameter("name", "Apple")
    .getSingleResult();
```

Typed queries keep the result shape explicit and are usually easier to maintain.

## Seed data with `import.sql`

Place `import.sql` in `src/main/resources/`:

```sql
insert into Fruit(id, name) values (1, 'Apple');
insert into Fruit(id, name) values (2, 'Pear');
```

- In `dev` and `test`, Quarkus loads `/import.sql` by default when present.
- Every statement must end with a semicolon.
- Override the file with `quarkus.hibernate-orm.sql-load-script`.
- Disable SQL loading with `quarkus.hibernate-orm.sql-load-script=no-file`.
