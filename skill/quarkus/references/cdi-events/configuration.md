# Quarkus CDI Events Configuration Reference

Use this file for the small set of settings that commonly affect CDI event behavior in Quarkus.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.arc.test.disable-application-lifecycle-observers` | `false` | Startup and shutdown observers should not run during tests |
| `quarkus.arc.exclude-types` | - | A discovered observer type should be excluded from CDI discovery |
| `quarkus.arc.exclude-dependency."name".*` | - | An entire dependency brings in unwanted observers |
| `quarkus.arc.remove-unused-beans` | `all` | You are diagnosing build-time bean removal that may affect observer discovery |
| `quarkus.arc.unremovable-types` | - | Programmatic lookup or framework integration makes an observer bean look unused |
| `quarkus.arc.dev-mode.monitoring-enabled` | `false` | You want extra dev-mode visibility for CDI behavior |

## Test-focused lifecycle control

```properties
quarkus.arc.test.disable-application-lifecycle-observers=true
```

This affects lifecycle observers such as startup or shutdown events, not normal domain-event observers fired by your own code.

## Discovery exclusions

```properties
quarkus.arc.exclude-types=org.acme.legacy.LegacyObserver,org.acme.internal.*
quarkus.arc.exclude-dependency.legacy.group-id=org.acme
quarkus.arc.exclude-dependency.legacy.artifact-id=legacy-events
```

Use exclusions when third-party observers should not become active in the application.
