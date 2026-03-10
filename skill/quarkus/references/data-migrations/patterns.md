# Quarkus Data Migrations Usage Patterns

Use these patterns for repeatable Flyway workflows in Quarkus.

## Pattern: Start local development with Dev Services and Flyway

When to use:

- You want a disposable local database and schema managed from SQL migrations.

Setup:

```properties
quarkus.datasource.db-kind=postgresql
quarkus.flyway.migrate-at-start=true
%dev.quarkus.flyway.clean-at-start=true
```

Workflow:

1. Add `quarkus-flyway` and the JDBC driver extension.
2. Create `src/main/resources/db/migration/V1__init.sql`.
3. Run `quarkus dev`.
4. Let Dev Services start the database container and Flyway apply the scripts.

Why this works well:

- You get reproducible schema bootstrapping without hand-managed local SQL.
- Restarting dev mode replays migrations after schema cleanup.

## Pattern: Transition from Hibernate-generated schema to Flyway

When to use:

- Early development used Hibernate schema generation and you now need durable migrations.

Recommended handoff:

1. Keep Hibernate schema generation only long enough to stabilize the first model.
2. Use the Dev UI Flyway support to create the initial migration from the generated schema.
3. Review the generated `V1.0.0__*.sql` carefully before committing it.
4. Enable `quarkus.flyway.migrate-at-start=true`.
5. Set `quarkus.flyway.baseline-on-migrate=true` if you are adopting Flyway against an already populated schema.
6. Move production-facing environments to `quarkus.hibernate-orm.schema-management.strategy=none` or `validate`.

Reasoning:

- Hibernate is convenient for exploration.
- Flyway is the safer long-term source of truth for shared environments.

## Pattern: Additive schema evolution with versioned SQL

When to use:

- You are shipping a normal schema change to an existing application.

Example:

`V2__add_registered_at_column.sql`

```sql
ALTER TABLE person ADD COLUMN registered_at TIMESTAMP;
UPDATE person SET registered_at = CURRENT_TIMESTAMP WHERE registered_at IS NULL;
ALTER TABLE person ALTER COLUMN registered_at SET NOT NULL;
```

Workflow:

1. Create a new `V...__description.sql` file.
2. Make the change additive first when possible.
3. Backfill data explicitly if the new constraint depends on it.
4. Run the app or tests against a fresh database and an upgraded database.
5. Commit the new migration without changing older files.

Prefer forward-only migrations. If a change is risky, split it into multiple smaller versions.

## Pattern: Organize multiple datasource migrations

When to use:

- Separate datasources own separate schemas and lifecycle.

Example layout:

```text
src/main/resources/
  db/default/V1__init.sql
  db/users/V1__init.sql
  db/inventory/V1__init.sql
```

Example config:

```properties
quarkus.flyway.locations=db/default
quarkus.flyway.users.locations=db/users
quarkus.flyway.inventory.locations=db/inventory

quarkus.flyway.migrate-at-start=true
quarkus.flyway.users.migrate-at-start=true
quarkus.flyway.inventory.migrate-at-start=true
```

Guidelines:

- Keep each datasource's migrations in a distinct directory tree.
- Do not mix unrelated schemas into one history table.
- Apply customizers with `@FlywayDataSource("name")` when only one datasource needs special handling.

## Pattern: Run migrations outside normal application startup

When to use:

- Production has multiple replicas or strict rollout control.

Recommended rollout:

1. Package the same migration scripts with the application artifact.
2. Run Flyway once in a controlled startup phase, such as a Kubernetes init job or a dedicated deployment step.
3. Start application replicas with schema already updated.
4. Keep application startup compatible with the migrated schema version.

Quarkus note:

- Generated Kubernetes manifests can externalize Flyway work into an init `Job`.
- If your platform already runs migrations separately, disable the generated init-task defaults and keep one source of execution.

## Pattern: Validate migration history in CI

When to use:

- You want early detection of missing, renamed, or invalid migration scripts.

Example profile:

```properties
%test.quarkus.flyway.migrate-at-start=true
%test.quarkus.flyway.validate-at-start=true
```

What to verify:

- Fresh schema creation from all migrations
- Upgrade path from a representative older version
- No checksum drift in already-applied scripts

## See Also

- [./configuration.md](./configuration.md) - Property combinations used by these workflows
- [./gotchas.md](./gotchas.md) - Safety limits around baseline, clean, validation, and reactive access
- [../data-orm/README.md](../data-orm/README.md) - ORM-side context for the Hibernate-to-Flyway handoff
