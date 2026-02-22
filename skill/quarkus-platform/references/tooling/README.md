# Tooling Reference

Use this module when you or the user needs to know how to work with Quarkus from the command line: bootstrap, extension management, dev loop, build/test, container images, deployment, and CLI plugins.

## Overview

There are two ways to work with quarkus projects from the command line:
- Quarkus CLI
- Build system (Maven/Gradle)

Treat them as complementary layers, not competing choices. Keep command intent aligned across both layers (same extensions, profiles, and build goals).


## General guidelines

- Use Quarkus CLI for local development speed and consistency.
- Use Maven/Gradle as a local fallback, directly in CI, automation scripts, and environments where reproducibility and zero extra-tool assumptions matter.
- For new projects, default to Maven unless the user explicitly asks for Gradle.
- For new projects, default to latest LTS version of Quarkus and Java unless the user explicitly asks for specifc versions.
- Prefer explicit coordinates for new projects instead of relying on defaults in team scripts.
- For reproducible automation, avoid wildcard extension adds unless intentionally broad.

## Quick Routing

1. Check CLI availability: `quarkus --version`.
2. If available, run CLI commands.
3. If unavailable, detect project build tool:
   - Maven: `pom.xml` or `mvnw`
   - Gradle: `build.gradle`, `build.gradle.kts`, or `gradlew`
4. Prefer local wrappers over global tools.

## Essential Commands 

| Task | Quarkus CLI | Maven fallback | Gradle fallback |
|------|-------------|----------------|-----------------|
| Create app | `quarkus create app org.acme:my-app` | Quarkus Maven plugin create flow | Prefer installing CLI, then use `quarkus create ... --gradle` |
| Run dev mode | `quarkus dev` | `./mvnw quarkus:dev` | `./gradlew quarkusDev` |
| Run tests | `quarkus test` | `./mvnw test` | `./gradlew test` |
| Build package | `quarkus build` | `./mvnw package` | `./gradlew build` |
| Add extension | `quarkus ext add rest` | `./mvnw quarkus:add-extension -Dextension=rest` | `./gradlew addExtension --extensions="rest"` |
| Build image | `quarkus image build <builder>` | Build-tool-specific container-image flow | Build-tool-specific container-image flow |
| Deploy | `quarkus deploy <target>` | Build-tool-specific deploy flow | Build-tool-specific deploy flow |

## In This Reference

[api.md](./api.md) - Quarkus CLI commands reference
[configuration.md](./configuration.md) - Quarkus CLI Installation and setup choices
[patterns.md](./patterns.md) - Common workflows and development patterns
[gotchas.md](./gotchas.md) - Comonn pitfalls and troubleshooting instructions

## Notes

- Built-in `--help` is the authoritative source for the installed CLI version.
- Command names and flags can evolve between Quarkus releases; verify with `quarkus <command> --help`.
