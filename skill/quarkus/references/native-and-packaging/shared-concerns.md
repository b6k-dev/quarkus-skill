# Shared Packaging Concerns

Use this file for concerns that cut across native builds, JVM package types, and container packaging.

## Output-Target-First Rule

- Decide the final deployment target first: native executable, JVM artifact, or container image.
- Do not choose `uber-jar`, `mutable-jar`, or a container builder before clarifying what the deployment environment actually needs.
- Keep build commands secondary to artifact and runtime requirements.

## Build-Time vs Runtime Boundary

- Quarkus packaging decisions often interact with build-time properties.
- If the problem is "can this still change after build?", route to `packaging-jars` for mutable-jar and re-augmentation, or `native-image` for native build-time metadata issues.
- Treat native-image failures caused by missing reflection/resources as build-time metadata problems, not runtime configuration problems.

## Packaging Shape vs Delivery Mechanism

- `packaging-jars` answers what JVM artifact shape you should produce.
- `packaging-containers` answers how that artifact or native binary becomes a container image.
- Avoid mixing these decisions unless the layout directly affects entrypoints or working directories.

## Native-to-Container Handoff

- Building a native executable and packaging it into a container is a two-step concern.
- Use `native-image` for how the binary is produced and validated.
- Use `packaging-containers` for base image choice, image construction, registry push, and runtime container concerns.
