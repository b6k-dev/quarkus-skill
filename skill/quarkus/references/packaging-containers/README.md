# Packaging Containers Reference

Use this module when the task is about packaging a Quarkus application as a container image for JVM or native deployment: choosing a container-image builder, naming and tagging images, pushing to registries, selecting base images, or understanding how Quarkus packaging layout affects the final image.

## Overview

Quarkus supports container packaging through multiple extensions and through the Quarkus CLI.

- `jib` builds images without writing or running a Dockerfile build locally.
- `docker` and `podman` build from generated or custom Dockerfiles under `src/main/docker`.
- `openshift` runs the image build inside the cluster.
- `buildpack` delegates image creation to Cloud Native Buildpacks.

Container packaging choices depend on two separate decisions:

1. Application mode: JVM or native
2. Builder model: Jib, Docker, Podman, OpenShift, or Buildpacks

## General guidelines

- Pick the builder based on environment constraints first, then tune image details.
- Prefer one container-image extension per project; if multiple are present, set `quarkus.container-image.builder` explicitly.
- Keep image naming explicit in CI with registry, repository path, and tag controlled by build inputs.
- Treat multi-platform builds as a registry-oriented workflow, not a local-image workflow.
- Keep Podman guidance as an operational variant of Docker-style builds, not a separate packaging strategy.

## Builder chooser

| Use this builder | When it fits best | Watch for |
|---|---|---|
| `jib` | No local daemon requirement, fast layered JVM rebuilds, direct registry push | Customization is property-driven, not Dockerfile-first |
| `docker` | You want explicit Dockerfile control or Docker buildx features | Multi-platform builds usually need direct push |
| `podman` | Podman is the standard runtime or you need Podman-specific platform flags | Socket and host setup differs across Linux vs Mac/Windows |
| `openshift` | Builds must happen inside an OpenShift cluster | Requires OpenShift/Kubernetes resource generation |
| `buildpack` | You want buildpack detection and curated run images | Needs a builder image and usually a Docker-compatible daemon |

## Quick Routing

1. Extension IDs, CLI/build entry points, and image naming examples -> `api.md`
2. High-value properties for builder selection, tagging, pushing, platforms, Dockerfiles, and base images -> `configuration.md`
3. Builder selection, CI flows, JVM vs native image patterns, and Podman subtopics -> `patterns.md`
4. Multi-platform, registry, layout, and base-image pitfalls -> `gotchas.md`

## In This Reference

[api.md](./api.md) - Builder entry points and copy-ready image build examples
[configuration.md](./configuration.md) - High-value container image properties by concern
[patterns.md](./patterns.md) - Repeatable packaging workflows and chooser guidance
[gotchas.md](./gotchas.md) - Common packaging failures and caveats

## See Also

- [../tooling/README.md](../tooling/README.md) - Quarkus CLI and build-tool command equivalents
- [../configuration/README.md](../configuration/README.md) - Profiles and environment-specific property management
