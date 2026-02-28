# Quarkus Configuration Gotchas

Common configuration pitfalls, symptoms, and fixes.

## Source Priority and Overrides

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Value in `application.properties` appears ignored | Higher-priority source (`-D`, env var, `.env`, `$PWD/config`) overrides it | Check effective value by reviewing higher-priority sources first |
| Runtime value differs from local dev expectation | Dev run picks up `.env` or shell vars unexpectedly | Inspect and clean local `.env`/env vars, then rerun |
| Property set in external file never applies | `quarkus.config.locations` missing/wrong URI scheme | Use valid URI (`file:`, `classpath:`, `jar:`, `http:`) and verify path |

## Environment Variable Naming Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Env var override does not work | Env key name does not match expected conversion | Use canonical env naming (for example `QUARKUS_HTTP_PORT`) |
| Profile-specific `.env` value is ignored | Profile key not prefixed with `_PROFILE_` style | Use `_DEV_...`, `_TEST_...`, etc. for profile-scoped dotenv keys |
| Quoted/dynamic datasource env var is ignored | Reverse mapping from env var to dotted key is ambiguous | Add dotted key in another source (can be empty), e.g. `quarkus.datasource."datasource-name".jdbc.url=` |

## Profile Resolution Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `%dev` values do not apply | Application not running in `dev` profile | Start with dev mode or set `quarkus.profile=dev` |
| Profile file values seem to "win" over inline profile keys | `application-{profile}.properties` has higher precedence than `%profile.` entries | Keep one source of truth per key per profile |
| Multiple profiles resolve unexpectedly | Last profile in `quarkus.profile=a,b,c` has highest priority | Order profiles intentionally and document the priority |
| Startup behavior is unstable across profiles | `quarkus.profile` configured inside profile-aware file | Set profile activation only in base sources, not profile-aware files |

## Mapping and Validation Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Startup fails with missing config mapping value | Required `@ConfigMapping` member has no value and no default | Add key, switch to optional type, or add `@WithDefault` |
| Mapping validation fails for unknown fields | Unknown-key validation is enabled for mapping | Remove unknown keys or tune mapping validation setting |
| Validation annotations are ignored | Hibernate Validator extension missing | Add `quarkus-hibernate-validator` |
| `@InjectMock` fails for config mapping | Mapping implementation is not a normal CDI proxy bean | Mock via producer methods (for example `@io.quarkus.test.Mock`) |

## Build-Time vs Runtime Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Property change has no effect until rebuild | Property is build-time-fixed | Rebuild/repackage after changing the key |
| Startup fails due to static init mismatch | Config read during static init differs at runtime | Use `@io.quarkus.runtime.annotations.StaticInitSafe` only when mismatch is safe |
| Hard to detect why artifact behavior changed | Build-time config drift not tracked | Enable `quarkus.config-tracking.enabled` and compare builds |

## Expressions and Defaults Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `${key:default}` default is not used | Key exists but is explicitly empty | Decide whether empty should be valid; if not, avoid setting empty value |
| Startup fails on expression resolution | Nested expression references unresolved key | Verify all referenced keys and fallback chain |
| Secret expression fails to decrypt | Secret handler or encryption key missing | Configure matching `smallrye.config.secret-handler.*` handler settings |

## Namespace Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| App settings conflict with framework behavior | Application keys use reserved `quarkus.` prefix | Move app keys to your own namespace (for example `acme.*`) |
