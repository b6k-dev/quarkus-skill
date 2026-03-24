# Quarkus JAR Packaging Configuration Reference

Use this file for the jar packaging settings that most often affect artifact shape, delivery layout, and post-build behavior.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.package.jar.type` | `fast-jar` | You need to choose `fast-jar`, `uber-jar`, `mutable-jar`, or `legacy-jar` explicitly |
| `quarkus.package.output-directory` | build-tool default | Different package variants must be written to separate output directories |
| `quarkus.package.output-name` | build-tool artifact name | The final artifact base name should differ from project coordinates |
| `quarkus.package.runner-suffix` | `-runner` | The generated runner artifact needs a custom suffix |
| `quarkus.package.jar.add-runner-suffix` | `true` | An uber jar should replace the original artifact name instead of adding a classifier-like suffix |
| `quarkus.package.ignored-entries` | signature files excluded by default for uber jars | Conflicting or unwanted entries must be removed from an uber jar |
| `quarkus.package.jar.filter-optional-dependencies` | `false` | Optional dependencies should be filtered out unless explicitly included |
| `quarkus.package.jar.included-optional-dependencies` | none | Only specific optional dependencies should be packaged |
| `quarkus.package.jar.include-dependency-list` | `true` | `fast-jar` or `mutable-jar` builds should emit dependency coordinates for scanners |
| `quarkus.package.jar.user-providers-directory` | unset | A mutable jar should expose a directory for user-supplied jars before re-augmentation |
| `quarkus.config.build-time-mismatch-at-runtime` | warn behavior by default | Runtime startup should fail instead of warn when build-time config drifts |

## Choose package type in application config

```properties
quarkus.package.jar.type=fast-jar
```

Common alternatives:

```properties
quarkus.package.jar.type=uber-jar
quarkus.package.jar.type=mutable-jar
quarkus.package.jar.type=legacy-jar
```

Prefer command-line overrides for occasional one-off builds. Prefer `application.properties` when the artifact shape is part of the service's expected delivery contract.

## Separate outputs for multiple builds

```properties
quarkus.package.output-directory=postgresql-quarkus-app
quarkus.package.output-name=orders-postgresql
```

This matters when the same module is packaged multiple times with different build-time settings and you need both outputs preserved.

## Uber-jar naming and exclusions

```properties
quarkus.package.jar.type=uber-jar
quarkus.package.runner-suffix=-runner
quarkus.package.jar.add-runner-suffix=true
quarkus.package.ignored-entries=META-INF/BC2048KE.SF,META-INF/BC2048KE.DSA
```

Set `quarkus.package.jar.add-runner-suffix=false` only when replacing the original jar name is intentional and downstream tooling expects that exact filename.

## Mutable-jar for re-augmentation

```properties
quarkus.package.jar.type=mutable-jar
quarkus.config.build-time-mismatch-at-runtime=fail
```

`mutable-jar` is required if you want to rebuild augmentation output after packaging.

`quarkus.config.build-time-mismatch-at-runtime=fail` is a useful guard when you want startup to reject drift instead of quietly running with ignored build-time changes.

## User providers directory for mutable jars

```properties
quarkus.package.jar.user-providers-directory=providers
```

This creates a directory in the packaged layout where users can place jars that become effective only after re-augmentation.

Use it only for advanced mutable-jar workflows. It has no value for ordinary immutable `fast-jar` delivery.

## Optional dependency filtering

```properties
quarkus.package.jar.filter-optional-dependencies=true
quarkus.package.jar.included-optional-dependencies=org.postgresql:postgresql::jar
```

Use this in multi-build setups where optional dependencies represent environment-specific choices and you do not want every variant to carry every optional driver.

If you have multiple entries, separate them with commas:

```properties
quarkus.package.jar.included-optional-dependencies=org.postgresql:postgresql::jar,com.oracle.database.jdbc:ojdbc11::jar
```

## Dependency list emission

```properties
quarkus.package.jar.include-dependency-list=true
```

This is primarily useful for `fast-jar` and `mutable-jar` distributions where scanners or inventory tooling benefit from a generated dependency coordinate list in `quarkus-app/`.

## Launch-time rebuild switch

Re-augmentation itself is triggered at launch time, not packaging time:

```bash
java -jar -Dquarkus.launch.rebuild=true target/quarkus-app/quarkus-run.jar
```

Treat `quarkus.launch.rebuild` as an operational flag used with a `mutable-jar`, not as a normal steady-state runtime setting.

## Container entrypoint note

If a container extension or custom Dockerfile overrides the JVM entrypoint, make sure it still points at the right packaged artifact shape.

- `fast-jar` and `mutable-jar` entrypoints usually target `quarkus-run.jar` inside the packaged directory layout.
- `uber-jar` entrypoints usually target a single runner jar file.
- `legacy-jar` may require older launch assumptions and should be treated explicitly.

## See Also

- [./api.md](./api.md) - Concrete commands and output layout examples
- [./gotchas.md](./gotchas.md) - Failure modes caused by wrong package settings or launch assumptions
- [../configuration/README.md](../configuration/README.md) - Broader build-time vs runtime config behavior
