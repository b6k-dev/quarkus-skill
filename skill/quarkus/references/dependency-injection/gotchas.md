# Quarkus Dependency Injection Gotchas (CDI / ArC)

Common CDI/ArC pitfalls, symptoms, and fixes.

## Injection Resolution Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `UnsatisfiedResolutionException` | No bean matches required type/qualifiers | Add a bean-defining scope, correct qualifiers, or verify discovery/indexing |
| `AmbiguousResolutionException` | Multiple beans match the same injection point | Add a qualifier, select alternatives, or use `Instance<T>` |
| Injection unexpectedly ambiguous with `@Named` | `@Named` can interact with `@Default` unexpectedly | Prefer `@Identifier` for internal string-based selection |

## Scope and Lifecycle Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `@ApplicationScoped` bean not created at startup | Normal scopes are lazy by default | Observe `StartupEvent` or use `@Startup` |
| Stateful `@ApplicationScoped` bean behaves inconsistently | Shared bean accessed concurrently without protection | Make state thread-safe or use `@Lock` |
| Unexpected behavior when accessing injected bean fields directly | Normal-scoped beans are client proxies | Invoke methods; do not rely on direct field reads/writes |

## Build-Time and Native Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Native image grows or fails due to reflection | Private injected members force reflective access | Use package-private constructor/fields/methods where possible |
| Bean available in code but missing at runtime | Bean removed as unused during build | Mark bean unremovable (`@Unremovable` or `quarkus.arc.unremovable-types`) |
| Build fails on intercepted private method | Interceptor binding on private method and fail-on-private enabled | Make method non-private or adjust `quarkus.arc.fail-on-intercepted-private-method` |

## Discovery and Integration Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Third-party beans are not discovered | Dependency not indexed and no `beans.xml` | Add Jandex index or configure `quarkus.index-dependency.*` |
| Third-party beans break startup | Problematic beans discovered automatically | Exclude with `quarkus.arc.exclude-types` or `quarkus.arc.exclude-dependency.*` |
| CDI Portable Extension does not work | Portable extensions are not supported in Quarkus build-time model | Replace with Quarkus extension/build-step approach |

## Reactive and Async Observer Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `@ObservesAsync` used as reactive pipeline stage | CDI async observers are thread-offloaded, not reactive stream stages | Keep observers `void`, use reactive APIs for stream composition |
| Async observer exception seems "lost" | `fireAsync()` completion not checked | Observe returned `CompletionStage` and handle exceptional completion |

## Dev Mode Debugging Pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `/q/arc/*` endpoints unavailable | Not running in dev mode | Run with `quarkus dev` / `./mvnw quarkus:dev` / `./gradlew quarkusDev` |
| Hard to understand removals/resolution | Diagnostics not enabled | Use ArC endpoints, enable processor DEBUG logs, and enable monitoring if needed |
