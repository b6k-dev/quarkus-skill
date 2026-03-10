# Quarkus Data Panache API Reference

Use this file for Panache runtime APIs with short, copy-ready examples.

## Extension entry point

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-hibernate-orm-panache</artifactId>
</dependency>
```

## Active record with `PanacheEntity`

```java
import jakarta.persistence.Entity;
import io.quarkus.hibernate.orm.panache.PanacheEntity;

@Entity
public class Person extends PanacheEntity {
    public String name;
    public Status status;
}
```

`PanacheEntity` gives you a generated `Long id` plus static helpers like `listAll()`, `findById()`, `count()`, and `delete()`.

## Active record with custom IDs: `PanacheEntityBase`

```java
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import io.quarkus.hibernate.orm.panache.PanacheEntityBase;

@Entity
public class Person extends PanacheEntityBase {
    @Id
    @GeneratedValue
    public Integer id;

    public String name;
}
```

## Repository pattern with `PanacheRepository`

```java
import jakarta.enterprise.context.ApplicationScoped;
import io.quarkus.hibernate.orm.panache.PanacheRepository;

@ApplicationScoped
public class PersonRepository implements PanacheRepository<Person> {
}
```

## Repository pattern with custom IDs: `PanacheRepositoryBase`

```java
import jakarta.enterprise.context.ApplicationScoped;
import io.quarkus.hibernate.orm.panache.PanacheRepositoryBase;

@ApplicationScoped
public class PersonRepository implements PanacheRepositoryBase<Person, Integer> {
}
```

## Common CRUD helpers

```java
Person p = new Person();
p.name = "Stef";
p.status = Status.Alive;

p.persist();
boolean managed = p.isPersistent();

Person one = Person.findById(1L);
List<Person> all = Person.listAll();
List<Person> alive = Person.list("status", Status.Alive);

long total = Person.count();
long aliveCount = Person.count("status", Status.Alive);

boolean deleted = Person.deleteById(1L);
long removed = Person.delete("status", Status.Deceased);
long updated = Person.update("status = ?1 where name = ?2", Status.Alive, "stef");
```

## Add custom query helpers

```java
@Entity
public class Person extends PanacheEntity {
    public String name;
    public Status status;

    public static Person findByName(String name) {
        return find("name", name).firstResult();
    }

    public static List<Person> findAlive() {
        return list("status", Status.Alive);
    }
}
```

## Simplified queries

```java
Person.find("name", name).firstResult();
Person.list("status", Status.Alive);
Person.list("order by name");
Person.find("name = ?1 and status = ?2", name, Status.Alive).list();
Person.find("name = :name and status = :status", Map.of("name", name, "status", Status.Alive)).list();
```

Panache expands short forms into normal HQL. Full HQL still works when needed.

## Paging and range queries

```java
import io.quarkus.hibernate.orm.panache.PanacheQuery;
import io.quarkus.panache.common.Page;

PanacheQuery<Person> query = Person.find("status", Status.Alive).page(Page.ofSize(25));

List<Person> first = query.list();
List<Person> second = query.nextPage().list();
int pages = query.pageCount();
long count = query.count();
```

```java
List<Person> firstRange = Person.find("status", Status.Alive)
        .range(0, 24)
        .list();
```

Do not mix `range()` and page-navigation methods on the same query state.

## Sorting

```java
import io.quarkus.panache.common.Sort;

List<Person> people = Person.list(Sort.by("name").and("birth"));
List<Person> alive = Person.list("status", Sort.by("name"), Status.Alive);
```

## Named queries

```java
import jakarta.persistence.Entity;
import jakarta.persistence.NamedQuery;

@Entity
@NamedQuery(name = "Person.byName", query = "from Person where name = ?1")
public class Person extends PanacheEntity {
    public String name;
}

Person p = Person.find("#Person.byName", "stef").firstResult();
```

Prefix the query name with `#` for `find`, `count`, `update`, or `delete`.

## Projections

```java
import io.quarkus.runtime.annotations.RegisterForReflection;

@RegisterForReflection
public record PersonName(String name) {
}

List<PersonName> names = Person.find("status", Status.Alive)
        .project(PersonName.class)
        .list();
```

`project(Class)` builds a DTO projection from constructor parameters. Use `@ProjectedConstructor` or `@ProjectedFieldName` when constructor selection or nested field mapping is ambiguous.

## Flush helpers

```java
@Transactional
void create(Person p) {
    p.persistAndFlush();
}
```

Use `flush()` or `persistAndFlush()` only when you need early database feedback inside the transaction.

## Locking

```java
import jakarta.persistence.LockModeType;

Person locked = Person.findById(id, LockModeType.PESSIMISTIC_WRITE);

Person alsoLocked = Person.find("name", name)
        .withLock(LockModeType.PESSIMISTIC_WRITE)
        .firstResult();
```

Lock queries must run inside a transaction.

## Streams

```java
try (Stream<Person> stream = Person.streamAll()) {
    return stream.map(p -> p.name).toList();
}
```

`stream*` methods require a transaction and should be closed.

## One persistence unit per Panache entity

A Panache entity can belong to only one persistence unit. Panache resolves the right `EntityManager` for that entity automatically.
