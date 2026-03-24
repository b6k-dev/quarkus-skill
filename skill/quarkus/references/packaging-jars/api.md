# Quarkus JAR Packaging API Reference

Use this file for package type commands, artifact shape, and the minimum launch expectations for each layout.

## Select a package type

Maven:

```bash
./mvnw package -Dquarkus.package.jar.type=fast-jar
./mvnw package -Dquarkus.package.jar.type=uber-jar
./mvnw package -Dquarkus.package.jar.type=mutable-jar
```

Gradle:

```bash
./gradlew build -Dquarkus.package.jar.type=fast-jar
./gradlew build -Dquarkus.package.jar.type=uber-jar
./gradlew build -Dquarkus.package.jar.type=mutable-jar
```

You can also set `quarkus.package.jar.type` in application configuration when the package choice is part of the project's normal build contract.

## Package type outputs

| Type | Output shape | Typical launch | Best fit |
|------|--------------|----------------|----------|
| `fast-jar` | `target/quarkus-app/` or `build/quarkus-app/` directory | `java -jar quarkus-run.jar` from that directory layout | Default production JVM packaging |
| `mutable-jar` | Same as `fast-jar` plus `lib/deployment/` | Same as `fast-jar` | Re-augmentation or remote dev |
| `uber-jar` | Single runnable jar in build output directory | `java -jar <name>-runner.jar` by default | Single-file delivery |
| `legacy-jar` | Older Quarkus jar layout | Depends on produced legacy runner jar layout | Compatibility with older tooling/scripts |

## Fast-jar layout

Maven output root:

```text
target/quarkus-app/
  app/
  lib/
  quarkus/
  quarkus-app-dependencies.txt
  quarkus-run.jar
```

Gradle output root:

```text
build/quarkus-app/
  app/
  lib/
  quarkus/
  quarkus-app-dependencies.txt
  quarkus-run.jar
```

Launch:

```bash
java -jar target/quarkus-app/quarkus-run.jar
java -jar build/quarkus-app/quarkus-run.jar
```

The full `quarkus-app/` directory is the deliverable. Do not copy only `quarkus-run.jar`.

## Mutable-jar delta

`mutable-jar` keeps the `fast-jar` layout and adds deployment artifacts:

```text
quarkus-app/
  lib/
    deployment/
```

That `lib/deployment/` directory is what enables re-augmentation and remote dev style rebuild workflows after packaging.

## Uber-jar behavior

Basic build:

```bash
./mvnw package -Dquarkus.package.jar.type=uber-jar
./gradlew build -Dquarkus.package.jar.type=uber-jar
```

By default the produced file uses the runner suffix:

```text
target/<final-name>-runner.jar
build/libs/<final-name>-runner.jar
```

The original non-uber jar is kept as the original artifact or renamed with an `.original` suffix depending on build tool behavior and suffix settings.

Exclude conflicting entries when needed:

```properties
quarkus.package.jar.type=uber-jar
quarkus.package.ignored-entries=META-INF/BC2048KE.SF,META-INF/BC2048KE.DSA
```

Gradle also supports repeated CLI options:

```bash
./gradlew quarkusBuild -Dquarkus.package.jar.type=uber-jar --ignored-entry=META-INF/file1.txt
```

## Re-augmentation command

Build as mutable jar first, then rebuild augmentation output with new build-time config:

```bash
./mvnw package -Dquarkus.package.jar.type=mutable-jar
java -jar -Dquarkus.launch.rebuild=true -Dquarkus.datasource.db-kind=mysql target/quarkus-app/quarkus-run.jar
```

```bash
./gradlew build -Dquarkus.package.jar.type=mutable-jar
java -jar -Dquarkus.launch.rebuild=true -Dquarkus.datasource.db-kind=mysql build/quarkus-app/quarkus-run.jar
```

This command rebuilds augmentation output and exits. It does not start the application normally.

## Optional dependency packaging filter

Use this only when a module intentionally declares optional dependencies and you want the final package to include only a selected subset.

```properties
quarkus.package.jar.filter-optional-dependencies=true
quarkus.package.jar.included-optional-dependencies=org.postgresql:postgresql::jar
```

Each dependency uses this shape:

```text
groupId:artifactId[:classifier[:type]]
```

## Output naming

Change the output name or directory when multiple package variants must coexist:

```properties
quarkus.package.output-name=orders-postgresql
quarkus.package.output-directory=postgresql-quarkus-app
```

For uber jars, control the runner suffix separately:

```properties
quarkus.package.runner-suffix=-runner
quarkus.package.jar.add-runner-suffix=false
```

## Container layout note

- `fast-jar` and `mutable-jar` container entrypoints usually assume the application files live together under the image working directory and launch `quarkus-run.jar`.
- `legacy-jar` and `uber-jar` image layouts use different working-directory expectations in some generated/container-extension defaults.
- If you override a container entrypoint manually, align it with the selected jar type instead of hard-coding one layout for all builds.

## See Also

- [./configuration.md](./configuration.md) - The properties behind these commands and output shapes
- [./patterns.md](./patterns.md) - How to choose among these layouts in practice
