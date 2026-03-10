# Quarkus Data Migrations Gotchas

Common Flyway pitfalls, symptoms, and fixes.

## Migration history safety

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Validation fails with checksum mismatch | An applied migration file was edited after execution | Restore the original file if possible; otherwise create a new migration and use `repair()` only as a deliberate recovery step |
| Team databases diverge after a rebase | Older migration files were renamed, deleted, or reordered | Treat applied migrations as append-only and add a new versioned file instead |

## Baseline mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Flyway marks a schema as initialized but expected objects are missing | `baseline-on-migrate` was enabled on the wrong database or at the wrong version | Use baseline only when onboarding an existing schema you already trust, and choose an explicit baseline version |
| First migration never runs on a legacy database | Baseline created history without matching the actual schema state | Compare the real schema to the intended baseline before enabling automatic baseline |

## Reactive and datasource wiring

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Flyway does not run for an otherwise reactive application | Only the reactive client is configured | Add JDBC datasource support and the matching JDBC driver extension because Flyway uses JDBC internally |
| Named datasource migrations hit the wrong schema | Flyway properties were configured on the default datasource instead of the named one | Use `quarkus.flyway.<name>.*` keys and `@FlywayDataSource("name")` for injection/customization |

## Validation and startup surprises

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Startup fails with missing or future migration errors | The history table references scripts not present in configured locations | Restore the expected files, correct `locations`, or intentionally use the relevant ignore setting after review |
| Migration succeeds locally but fails in another environment | Database-specific SQL or schema assumptions differ | Test migrations against the same database family and schema layout used by the target environment |

## Clean and dev-mode hazards

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Local development keeps wiping data unexpectedly | `%dev.quarkus.flyway.clean-at-start=true` is active | Limit `clean-at-start` to disposable environments and disable it for shared databases |
| Cleanup is blocked when you expected it to work | `clean-disabled=true` or database permissions prevent clean | Reserve clean for dev/test and do not rely on it as a production recovery path |

## Oracle Dev Services caveat

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Multi-schema Oracle Dev Services migrations fail with privilege errors | The `quarkus` user cannot perform required DDL across schemas | Grant needed privileges before startup or use `quarkus.datasource.devservices.init-privileged-script-path` to prepare the container |

## See Also

- [./configuration.md](./configuration.md) - The startup and validation flags behind most of these failures
- [./patterns.md](./patterns.md) - Safer rollout patterns for dev, CI, and production
