# Quarkus Data Panache Usage Patterns

Use these patterns for repeatable Panache implementation workflows.

## Pattern: CRUD with active record

When to use:

- The entity owns straightforward persistence behavior.
- Static query helpers keep the feature easier to navigate.

Example:

```java
import jakarta.persistence.Entity;
import jakarta.transaction.Transactional;
import io.quarkus.hibernate.orm.panache.PanacheEntity;

@Entity
public class Person extends PanacheEntity {
    public String name;
    public Status status;

    public static List<Person> findAlive() {
        return list("status", Status.Alive);
    }
}

@Transactional
void create(String name) {
    Person p = new Person();
    p.name = name;
    p.status = Status.Alive;
    p.persist();
}
```

## Pattern: CRUD with repository style

When to use:

- You want injected collaborators, instance methods, or clearer separation from the entity class.

Example:

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.transaction.Transactional;
import io.quarkus.hibernate.orm.panache.PanacheRepository;

@ApplicationScoped
public class PersonRepository implements PanacheRepository<Person> {
    public List<Person> findAlive() {
        return list("status", Status.Alive);
    }
}

@ApplicationScoped
public class PersonService {
    @Inject
    PersonRepository people;

    @Transactional
    public void create(Person person) {
        people.persist(person);
    }
}
```

## Pattern: REST resource with Panache

When to use:

- You want small CRUD endpoints without a large service layer.

Example:

```java
import java.net.URI;
import jakarta.transaction.Transactional;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.core.Response;

@Path("/persons")
public class PersonResource {
    @GET
    public List<Person> list() {
        return Person.listAll();
    }

    @POST
    @Transactional
    public Response create(Person person) {
        person.persist();
        return Response.created(URI.create("/persons/" + person.id)).build();
    }
}
```

## Pattern: Paging instead of loading everything

When to use:

- The table can grow beyond a small dataset.

Example:

```java
import io.quarkus.hibernate.orm.panache.PanacheQuery;
import io.quarkus.panache.common.Page;

PanacheQuery<Person> query = Person.find("status", Status.Alive)
        .page(Page.ofSize(50));

List<Person> page1 = query.list();
List<Person> page2 = query.nextPage().list();
long total = query.count();
```

Prefer `find()` plus `PanacheQuery` for large datasets; reserve `listAll()` and `streamAll()` for bounded cases.

## Pattern: Projection for read models

When to use:

- The endpoint only needs a subset of columns.
- You want to avoid returning the full entity graph.

Example:

```java
import io.quarkus.runtime.annotations.RegisterForReflection;

@RegisterForReflection
public record PersonSummary(String name) {
}

List<PersonSummary> summaries = Person.find("status", Status.Alive)
        .project(PersonSummary.class)
        .list();
```

Use records when possible; they make projection constructors explicit and compact.

## Pattern: Early flush for fast failure

When to use:

- You need to catch a persistence failure before the transaction finishes.

Example:

```java
import jakarta.persistence.PersistenceException;
import jakarta.transaction.Transactional;

@Transactional
void create(Person person) {
    try {
        person.persistAndFlush();
    } catch (PersistenceException e) {
        auditFailure(person);
        throw e;
    }
}
```

## Pattern: Testing active record code

When to use:

- Your feature calls static Panache methods on entities.

Example:

```java
import io.quarkus.panache.mock.PanacheMock;
import org.mockito.Mockito;

PanacheMock.mock(Person.class);
Mockito.when(Person.count()).thenReturn(23L);
assertEquals(23L, Person.count());
PanacheMock.verify(Person.class).count();
```

Add `io.quarkus:quarkus-panache-mock` in test scope for active record mocking.

## Pattern: Testing repository code

When to use:

- Your feature injects a Panache repository bean.

Example:

```java
import io.quarkus.test.InjectMock;
import org.mockito.Mockito;

@InjectMock
PersonRepository people;

Mockito.when(people.count()).thenReturn(23L);
assertEquals(23L, people.count());
Mockito.verify(people).count();
```

Use normal bean mocking via `quarkus-junit-mockito` for repository-style code.
