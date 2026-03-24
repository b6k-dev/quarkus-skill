# Quarkus JAR Packaging Usage Patterns

Use these patterns to choose a jar layout based on operational needs instead of habit.

## Pattern: Default service packaging with `fast-jar`

When to use:

- Standard JVM service deployment.
- Container images or ZIP bundles can carry a directory, not just one file.

Example:

```bash
./mvnw package
java -jar target/quarkus-app/quarkus-run.jar
```

```bash
./gradlew build
java -jar build/quarkus-app/quarkus-run.jar
```

Why this pattern:

- `fast-jar` is the default.
- It keeps dependencies separate and matches Quarkus' modern runner layout.
- It is the safest default for container layering and ordinary JVM operations.

## Pattern: Single-file handoff with `uber-jar`

When to use:

- The deployment target or operator strongly prefers one runnable file.
- You are handing off artifacts to environments that do not want a `quarkus-app/` directory.

Example:

```bash
./mvnw package -Dquarkus.package.jar.type=uber-jar
java -jar target/app-runner.jar
```

```bash
./gradlew build -Dquarkus.package.jar.type=uber-jar
java -jar build/libs/app-runner.jar
```

Tradeoff:

- Distribution is simpler.
- Dependency layering and file-level separation are gone.
- Resource/signature conflicts are more likely to matter.

## Pattern: Ship one mutable build for environment-specific build-time choices

When to use:

- Users must change build-time-fixed behavior after packaging.
- One binary distribution must support multiple environment variants.

Example:

```properties
quarkus.package.jar.type=mutable-jar
quarkus.config.build-time-mismatch-at-runtime=fail
```

```bash
java -jar -Dquarkus.launch.rebuild=true -Dquarkus.datasource.db-kind=mysql target/quarkus-app/quarkus-run.jar
```

Use this for cases like switching database kind or enabling build-time extensions that were already present during the original build.

## Pattern: Prepare a mutable jar for remote dev

When to use:

- The app runs remotely in a dev environment and local changes must sync into that environment.

Example:

```properties
quarkus.package.jar.type=mutable-jar
quarkus.live-reload.password=changeit
```

Remote process:

```bash
export QUARKUS_LAUNCH_DEVMODE=true
java -jar target/quarkus-app/quarkus-run.jar
```

Local side:

```bash
./mvnw quarkus:remote-dev -Dquarkus.live-reload.url=http://my-remote-host:8080
./gradlew quarkusRemoteDev -Dquarkus.live-reload.url=http://my-remote-host:8080
```

This is a development workflow only, not a production packaging pattern.

## Pattern: Multi-build outputs with isolated directories

When to use:

- One module is packaged several times with different build-time config.
- You need to preserve each resulting `quarkus-app/` directory.

Example:

```properties
quarkus.package.output-directory=oracle-quarkus-app
quarkus.package.output-name=orders-oracle
```

Create a separate output directory per variant instead of letting later builds overwrite `quarkus-app`.

## Pattern: Package only the optional dependencies a variant actually needs

When to use:

- A project declares several optional drivers/providers.
- Each packaged variant should include only one selected subset.

Example:

```properties
quarkus.package.jar.filter-optional-dependencies=true
quarkus.package.jar.included-optional-dependencies=org.postgresql:postgresql::jar
```

This keeps environment-specific builds smaller and avoids shipping unused optional jars.

## Pattern: Keep container entrypoints aligned with jar type

When to use:

- A Dockerfile or container-image extension overrides the default JVM command.

Example:

```properties
quarkus.jib.jvm-entrypoint=java,-jar,quarkus-run.jar
```

Use this only when the packaged layout is actually `fast-jar` or `mutable-jar` and the working directory matches that expectation.

If you later switch to `uber-jar`, revisit the entrypoint instead of assuming the same file layout still exists.

## See Also

- [./configuration.md](./configuration.md) - Properties that implement these workflows
- [./gotchas.md](./gotchas.md) - Typical ways these patterns break in practice
