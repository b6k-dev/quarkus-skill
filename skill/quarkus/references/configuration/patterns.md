# Quarkus Configuration Usage Patterns

Use these patterns for repeatable configuration workflows.

## Pattern: Required + Optional + Default Property Injection

When to use:

- You need a small number of keys directly in a resource/service.

Example:

```java
import org.eclipse.microprofile.config.inject.ConfigProperty;

import java.util.Optional;

class GreetingService {
    @ConfigProperty(name = "greeting.message")
    String message;

    @ConfigProperty(name = "greeting.suffix", defaultValue = "!")
    String suffix;

    @ConfigProperty(name = "greeting.name")
    Optional<String> name;
}
```

## Pattern: Group Related Keys with `@ConfigMapping`

When to use:

- Multiple related keys belong to one domain concept.

Example:

```java
import io.smallrye.config.ConfigMapping;
import io.smallrye.config.WithDefault;

@ConfigMapping(prefix = "payments")
interface PaymentsConfig {
    String host();

    @WithDefault("30S")
    java.time.Duration timeout();

    Retry retry();

    interface Retry {
        @WithDefault("3")
        int maxAttempts();
    }
}
```

Prefer this over many scattered `@ConfigProperty` fields.

## Pattern: Environment-Specific Overrides with Profiles

When to use:

- Port, endpoint, or feature behavior differs by environment.

Example:

```properties
quarkus.http.port=9090
%dev.quarkus.http.port=8181
%test.quarkus.http.port=8182

quarkus.profile=staging,tenant-a
quarkus.config.profile.parent=common
```

Use profile-aware files when overrides become large:

- `application-staging.properties`
- `application-tenant-a.properties`

## Pattern: Layered Runtime Overrides for Deployments

When to use:

- Base config lives in the artifact but deploy-time values must stay external.

Example:

```properties
quarkus.config.locations=file:/etc/acme/app.properties
```

Deployment override examples:

```bash
java -Dacme.api.host=api.internal -jar target/quarkus-app/quarkus-run.jar
export ACME_API_HOST=api.internal
```

## Pattern: Fallback Chains with Property Expressions

When to use:

- A key should resolve from runtime env first, then app defaults.

Example:

```properties
remote.host=quarkus.io
application.host=${HOST:${remote.host}}
application.url=https://${application.host}
```

This keeps defaults in one place while allowing external override.

## Pattern: Local Secret Handling with `.env`

When to use:

- Developers need local credentials without committing secrets.

Example (`.env`):

```properties
ACME_API_KEY=dev-secret
QUARKUS_DATASOURCE_PASSWORD=dev-password
```

Add `.env` to `.gitignore` and use real secret stores in production.

## Pattern: Validate Configuration at Startup

When to use:

- Invalid config should fail fast before serving traffic.

Example:

```java
import io.smallrye.config.ConfigMapping;
import jakarta.validation.constraints.Max;
import jakarta.validation.constraints.Min;

@ConfigMapping(prefix = "api")
interface ApiConfig {
    @Min(500)
    @Max(3000)
    int requestTimeoutMillis();
}
```

Requires `quarkus-hibernate-validator`.

## Pattern: Track Build-Time Configuration Drift

When to use:

- You need to detect build-time config changes between CI builds.

Example:

```properties
quarkus.config-tracking.enabled=true
```

```bash
./mvnw quarkus:track-config-changes
```

Use this when build-time-fixed settings impact artifacts.
