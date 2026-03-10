# Quarkus Data Panache Configuration Reference

Use this file for the Hibernate ORM and datasource settings that matter most to Panache.

## Key point

Panache does not introduce a separate configuration model for normal runtime use. It uses the same datasource and Hibernate ORM settings as standard Quarkus ORM.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.datasource.db-kind` | - | Selecting the database family |
| `quarkus.datasource.jdbc.url` | - | Pointing the ORM layer at the database |
| `quarkus.datasource.username` | - | Providing DB credentials |
| `quarkus.datasource.password` | - | Providing DB credentials |
| `quarkus.hibernate-orm.schema-management.strategy` | `none` | Creating, updating, or recreating schema during development/tests |
| `quarkus.hibernate-orm.log.sql` | `false` | Inspecting generated SQL from Panache operations |
| `quarkus.hibernate-orm.database.generation.create-schemas` | `false` | Auto-creating database schemas when supported |
| `quarkus.hibernate-orm.sql-load-script` | `import.sql` in dev/test | Loading seed data that Panache queries can use immediately |

## Minimal setup

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/app
quarkus.datasource.username=app
quarkus.datasource.password=app

quarkus.hibernate-orm.schema-management.strategy=update
```

## Helpful development settings

```properties
quarkus.hibernate-orm.log.sql=true
quarkus.hibernate-orm.schema-management.strategy=drop-and-create
quarkus.hibernate-orm.sql-load-script=import.sql
```

Use `drop-and-create` only for disposable environments.

## Configuration guidance

- If a Panache query fails, check datasource connectivity and base Hibernate ORM configuration first.
- If projection or query behavior looks wrong, enable SQL logging before debugging Panache code.
- If you seed demo data for Panache CRUD examples, keep it in `import.sql` or your normal migration workflow.

## See Also

- [`../data-orm/configuration.md`](../data-orm/configuration.md) - Base Hibernate ORM settings that Panache inherits directly
