# Native Image Reference

Use this module for Quarkus native executable work with GraalVM or Mandrel: choosing local vs containerized builds, handling closed-world issues, enabling native SSL, debugging build/runtime problems, and validating native behavior with tests.

## Overview

This module focuses on application-developer concerns around Quarkus native executables.

- Use it when the task is about `-Dnative`, `quarkus build --native`, Mandrel, GraalVM, builder images, reflection/resource/proxy metadata, class initialization, or native-only runtime diagnostics.
- Prefer this module when the question is "why does native fail or behave differently from JVM mode?"
- Keep build-tool-specific syntax minimal; use `tooling` for broader CLI, Maven, or Gradle workflows.

## What This Covers

- Local GraalVM/Mandrel builds versus containerized Linux-native builds.
- High-value `quarkus.native.*` and `quarkus.ssl.native` configuration.
- Reflection, resources, proxies, resource bundles, and class-init fixes.
- Native SSL defaults, trust store tradeoffs, and when native SSL must be enabled explicitly.
- Native reports, debug symbols, JFR, GC logging, heap/thread dumps, and CPU compatibility tuning.
- Native-focused test patterns with `@QuarkusIntegrationTest`.

## Quick Routing

1. Build commands, annotations, and copy-ready snippets -> `api.md`
2. High-value native properties and defaults -> `configuration.md`
3. Repeatable native build, metadata, and runtime tuning approaches -> `patterns.md`
4. Native-only failures, symptoms, and fixes -> `gotchas.md`
5. Native test setup and verification strategy -> `testing.md`

## In This Reference

- [api.md](./api.md) - Native build commands, annotations, and metadata examples
- [configuration.md](./configuration.md) - High-value native and SSL properties
- [patterns.md](./patterns.md) - Repeatable build, metadata, and runtime tuning patterns
- [gotchas.md](./gotchas.md) - Common native-image pitfalls and fixes
- [testing.md](./testing.md) - Native executable test guidance with `@QuarkusIntegrationTest`

## See Also

- [../tooling/README.md](../tooling/README.md) - CLI, Maven, and Gradle workflows outside native-specific concerns
- [../configuration/README.md](../configuration/README.md) - Profiles and config layering that affect native builds and native tests
