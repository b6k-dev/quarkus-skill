# Quarkus Configuration Settings Reference

Use this file for source precedence, profile controls, and high-value Quarkus config keys.

## Config source priority (high to low)

| Source | Typical location | Ordinal |
|--------|------------------|---------|
| System properties | `-Dkey=value` | `400` |
| Environment variables | shell/container env | `300` |
| Dotenv file | `$PWD/.env` | `295` |
| Working directory config | `$PWD/config/application.properties` | `260` |
| Classpath app config | `src/main/resources/application.properties` | `250` |
| MicroProfile default file | `META-INF/microprofile-config.properties` | `100` |

If the same key exists in multiple sources, the highest-priority source wins.

## High-value Quarkus configuration keys

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.profile` | `prod` (outside dev/test) | You need to activate one or more custom profiles |
| `quarkus.config.profile.parent` | - | Profile values should fall back to a parent profile |
| `quarkus.config.locations` | - | Config must be loaded from extra files/URIs |
| `quarkus.config.sources.system-only` | - | You want to restrict resolution to system properties only |
| `quarkus.config.mapping.validate-unknown` | - | You want stricter mapping checks for unknown properties |
| `quarkus.config.log.values` | - | You need diagnostics for resolved config values |
| `quarkus.config.build-time-mismatch-at-runtime` | - | You need explicit behavior for build-time/runtime value mismatch |
| `quarkus.config-tracking.enabled` | `false` | You want build-time config tracking artifacts |
| `quarkus.config-tracking.directory` | - | Tracking output should go to a custom directory |
| `quarkus.config-tracking.file-prefix` | - | Tracking filename prefix must be customized |
| `quarkus.config-tracking.file-suffix` | - | Tracking filename suffix must be customized |
| `quarkus.config-tracking.file` | - | Tracking output should use a fixed path |
| `quarkus.config-tracking.exclude` | - | Specific properties should be excluded from tracking |
| `quarkus.config-tracking.hash-options` | - | Tracked option values should be hashed |

## External configuration locations

Load additional config sources via URI list:

```properties
quarkus.config.locations=file:/etc/acme/app.properties,classpath:tenant-defaults.properties
```

Supported schemes include `file:`, `classpath:`, `jar:`, and `http:`.

## Profiles and precedence controls

Activate profiles:

```properties
quarkus.profile=staging,tenant-a
quarkus.config.profile.parent=common
```

Inline profile-specific keys:

```properties
quarkus.http.port=9090
%dev.quarkus.http.port=8181
%staging.quarkus.http.port=8088
```

Profile-aware files:

- `application-dev.properties`
- `application-staging.properties`

Rules that affect resolution order:

- Profile-aware files override inline `%profile.` entries in `application.properties`.
- For multiple active profiles, the last profile listed has highest priority.
- Do not set `quarkus.profile` or `quarkus.config.profile.parent` inside profile-aware files.

## Environment variable and dotenv usage

Runtime env var example:

```bash
export QUARKUS_DATASOURCE_PASSWORD=secret
```

Runner JAR and native executable examples:

```bash
export QUARKUS_DATASOURCE_PASSWORD=youshallnotpass
java -jar target/quarkus-app/quarkus-run.jar

export QUARKUS_DATASOURCE_PASSWORD=youshallnotpass
./target/myapp-runner
```

Dotenv example (`$PWD/.env`):

```properties
QUARKUS_DATASOURCE_PASSWORD=secret
_DEV_QUARKUS_HTTP_PORT=8181
```

Keep `.env` files local and out of version control.

### Environment variable name conversion

For a property like `foo.BAR.baz`, config checks these env names:

1. `foo.BAR.baz` (exact)
2. `foo_BAR_baz` (replace non-alphanumeric/non-underscore chars with `_`)
3. `FOO_BAR_BAZ` (same replacement, then uppercase)

SmallRye Config adds additional conversions:

| Property name | Env var name |
|---------------|--------------|
| `foo."bar".baz` | `FOO__BAR__BAZ` |
| `foo.bar-baz` | `FOO_BAR_BAZ` |
| `foo.bar[0]` | `FOO_BAR_0_` |
| `foo.bar[0].baz` | `FOO_BAR_0__BAZ` |

### Dynamic path segments and two-way conversion

For names with user-defined segments, reverse mapping from env var to dotted name can be ambiguous.

Example dotted key:

```properties
quarkus.datasource."datasource-name".jdbc.url=
```

Matching env var:

```bash
export QUARKUS_DATASOURCE__DATASOURCE_NAME__JDBC_URL=jdbc:postgresql://localhost:5432/database
```

Why this is needed:

- `DATASOURCE_NAME` could map back to `datasource-name`, `datasource.name`, or another separator form.
- Supplying the dotted key in another source disambiguates the intended property name.

## Reserved prefixes and lifecycle

- The `quarkus.` prefix is reserved for Quarkus core/extensions.
- Use your own namespace (for example `acme.*`) for application keys.
- Build-time-fixed properties require rebuild to take effect.

## Build-time configuration tracking

Enable tracking:

```properties
quarkus.config-tracking.enabled=true
```

Then compare changes between builds with the Maven goal:

```bash
./mvnw quarkus:track-config-changes
```
