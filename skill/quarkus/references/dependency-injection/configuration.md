# Quarkus Dependency Injection Configuration Reference (ArC)

Use this file for ArC configuration and bean discovery tuning.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.arc.remove-unused-beans` | `all` | You need to tune bean removal: `all` removes all unused beans, `none` disables removal, `fwk` keeps unused application beans but still removes unused non-application beans |
| `quarkus.arc.unremovable-types` | - | Programmatic lookup or framework integration causes false-positive removals |
| `quarkus.arc.exclude-types` | - | A discovered type should not become a bean/observer |
| `quarkus.arc.exclude-dependency."name".*` | - | An entire dependency should be excluded from discovery |
| `quarkus.arc.selected-alternatives` | - | Select alternatives globally through config |
| `quarkus.arc.strict-compatibility` | `false` | You need behavior closer to the CDI specification |
| `quarkus.arc.transform-unproxyable-classes` | `true` | Final/unproxyable class transformations should be controlled |
| `quarkus.arc.transform-private-injected-fields` | `true` | Private injected fields should stay private (reflection fallback) |
| `quarkus.arc.fail-on-intercepted-private-method` | `true` | Private intercepted methods should fail build vs warn |
| `quarkus.arc.auto-inject-fields` | `true` | Auto-adding `@Inject` to known qualifier annotations should be disabled |
| `quarkus.arc.auto-producer-methods` | `true` | Auto-detection of producer methods should be disabled |
| `quarkus.arc.dev-mode.monitoring-enabled` | `false` | You want method/event monitoring in Dev UI |
| `quarkus.arc.dev-mode.generate-dependency-graphs` | `auto` | You need explicit dependency graph behavior in dev mode |
| `quarkus.arc.test.disable-application-lifecycle-observers` | `false` | Startup/shutdown observers should not run during tests |
| `quarkus.arc.context-propagation.enabled` | `true` | CDI context propagation with SmallRye Context Propagation must be toggled |

## Bean discovery and indexing

If a dependency is not being discovered as expected, add indexing hints:

```properties
quarkus.index-dependency.acme.group-id=org.acme
quarkus.index-dependency.acme.artifact-id=acme-api
```

Use excludes when discovery pulls in incompatible beans:

```properties
quarkus.arc.exclude-types=org.acme.LegacyBean,org.acme.internal.*,BadBean
quarkus.arc.exclude-dependency.legacy.group-id=org.acme
quarkus.arc.exclude-dependency.legacy.artifact-id=legacy-services
```

## Unused bean removal tuning

Default behavior removes beans considered unused at build time.

Common controls:

```properties
quarkus.arc.remove-unused-beans=all
quarkus.arc.unremovable-types=org.acme.ImportantBean,org.acme.plugin.**
```

Use `none` to disable removal completely if diagnosis is still in progress.

## Strict mode

Enable stricter CDI compatibility checks:

```properties
quarkus.arc.strict-compatibility=true
```

When moving toward strict behavior, also review:

- `quarkus.arc.transform-unproxyable-classes`
- `quarkus.arc.remove-unused-beans`

## Dev mode diagnostics settings

ArC diagnostics and monitoring:

```properties
quarkus.arc.dev-mode.monitoring-enabled=true
quarkus.arc.dev-mode.generate-dependency-graphs=auto
```

Useful logging categories:

```properties
quarkus.log.category."io.quarkus.arc.processor".level=DEBUG
quarkus.log.category."io.quarkus.arc.requestContext".min-level=TRACE
quarkus.log.category."io.quarkus.arc.requestContext".level=TRACE
```

## Pattern syntax for type lists

Properties such as `selected-alternatives`, `exclude-types`, and `unremovable-types` accept:

- Fully qualified class names (`org.acme.Foo`)
- Simple class names (`Foo`)
- Package match (`org.acme.*`)
- Package prefix match (`org.acme.**`)
