# JAR Packaging Reference

Use this module when the task is about Quarkus JVM package shape: choosing between `fast-jar`, `uber-jar`, `mutable-jar`, or `legacy-jar`, understanding the generated layout, or deciding how that artifact should be launched and delivered.

## Overview

Quarkus can package JVM applications into several JAR-oriented layouts with different operational tradeoffs.

- `fast-jar` is the default and the best general-purpose choice.
- `uber-jar` trades layered dependency layout for a single runnable file.
- `mutable-jar` keeps the `fast-jar` layout but adds deployment artifacts needed for re-augmentation and remote dev workflows.
- `legacy-jar` exists mainly for compatibility with older expectations about output shape.

The important question is usually not "which command builds a jar" but "what file or directory am I actually shipping, and what changes are still allowed after build time?"

## General guidelines

- Default to `fast-jar` unless there is a concrete distribution reason not to.
- Choose `uber-jar` only when a single-file handoff matters more than layered layout and dependency separation.
- Choose `mutable-jar` only when post-build re-augmentation or remote dev is a real requirement.
- Treat `legacy-jar` as a compatibility escape hatch, not a modern default.

## Quick Routing

1. Package type commands, output shapes, runner names, and re-augmentation commands -> `api.md`
2. High-value jar packaging properties and output customization -> `configuration.md`
3. How to choose a package type for real delivery workflows -> `patterns.md`
4. Common packaging mistakes, especially around mutable jars and launch assumptions -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Package types, commands, and generated artifact layout
[configuration.md](./configuration.md) - High-value jar packaging properties and launch-related settings
[patterns.md](./patterns.md) - Repeatable packaging decisions and delivery workflows
[gotchas.md](./gotchas.md) - Common jar packaging pitfalls and recovery guidance

## See Also

- [../tooling/README.md](../tooling/README.md) - CLI, Maven, and Gradle command selection outside jar-layout decisions
- [../configuration/README.md](../configuration/README.md) - Build-time vs runtime configuration behavior that drives re-augmentation decisions
