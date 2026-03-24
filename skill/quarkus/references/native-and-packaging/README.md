# Native and Packaging Routing Guide

Use this module when the task starts broadly around Quarkus build outputs and you first need to choose between native-image work, JVM JAR packaging, and container packaging.

## What this module is for

This module is an entrypoint, not a full implementation reference.

- Start here when the question is "which packaging or native module do I need?"
- Use it to separate native-image troubleshooting from artifact-shape decisions and container builder choices.
- Use it for the small set of cross-cutting concerns shared across these areas: build-time vs runtime boundaries, output-target-first decisions, and where `tooling` takes over.
- Move into the leaf modules once the primary concern is clear.

## Boundary-First Chooser

```text
What are you actually trying to produce or debug?
├── A native executable or native-only runtime behavior
│   └── native-image
├── A JVM artifact shape to ship or run
│   └── packaging-jars
└── A container image to build, tag, push, or tune
    └── packaging-containers
```

## In This Reference

- [routing.md](./routing.md) - Task-to-module routing and boundary decisions
- [shared-concerns.md](./shared-concerns.md) - Cross-cutting packaging concerns and escalation rules

## See Also

- [../native-image/README.md](../native-image/README.md) - Native executable builds, diagnostics, and testing
- [../packaging-jars/README.md](../packaging-jars/README.md) - JVM JAR package shapes and re-augmentation
- [../packaging-containers/README.md](../packaging-containers/README.md) - Container image packaging and builder selection
- [../tooling/README.md](../tooling/README.md) - CLI, Maven, and Gradle workflows
