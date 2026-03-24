# Quarkus Packaging Containers Usage Patterns

Use these patterns for repeatable container packaging workflows in Quarkus.

## Pattern: Choose the lightest viable builder

When to use:

- You are starting container packaging for a service and need a default.

Chooser:

1. Use `jib` when you want registry-oriented image builds without maintaining Dockerfiles.
2. Use `docker` when the image build should be Dockerfile-first or rely on buildx.
3. Use `podman` when Podman is the standard runtime and you need Podman-specific platform behavior.
4. Use `openshift` when the cluster must own the image build process.
5. Use `buildpack` when buildpack detection and curated run images matter more than Dockerfile control.

Default recommendation:

- JVM service in normal CI: start with `jib`.
- Custom OS package or filesystem setup required: switch to `docker` or `podman`.

## Pattern: Build a JVM image with Jib for fast iterative pushes

When to use:

- The app runs on the JVM and CI should push efficiently with stable dependency layers.

Setup:

```properties
quarkus.container-image.builder=jib
quarkus.container-image.registry=ghcr.io
quarkus.container-image.group=acme
quarkus.container-image.name=orders-service
quarkus.container-image.tag=${quarkus.application.version}
```

Why this works well:

- Jib separates dependencies from application content, so small code changes usually push fewer bytes.
- No Docker daemon is required when pushing directly to a registry.

Use a Dockerfile-based builder instead when startup scripts, package installation, or OS-level customization belong in the image definition.

## Pattern: Keep Docker or Podman packaging Dockerfile-first

When to use:

- The image needs explicit OS package installation, custom users, extra copied assets, or multi-stage assembly.

Workflow:

1. Keep Quarkus-generated `src/main/docker/Dockerfile.jvm` and `src/main/docker/Dockerfile.native` as the starting point.
2. Choose `docker` or `podman` as the builder.
3. Evolve the Dockerfile for base image, package installation, copied files, and entrypoint behavior.
4. Build through Quarkus so image naming and tag properties stay centralized.

Reasoning:

- Dockerfile builders are the better fit when image construction itself is a first-class artifact.

## Pattern: Package native apps with an explicit runtime-base decision

When to use:

- The service is built as a native executable and container size vs runtime convenience matters.

Recommended decision:

1. Start with the current supported Quarkus micro runtime image for a small, purpose-built runtime base.
2. Move to a minimal-but-less-restricted base image if you need extra tools, package installation, or easier runtime library management.
3. If the native binary depends on additional shared objects, use a multi-stage Dockerfile or a less minimal base image.

Example native Dockerfile direction:

```dockerfile
FROM <supported-minimal-runtime-base-image>
WORKDIR /work/
RUN chown 1001 /work && chmod g+rwX /work && chown 1001:root /work
COPY --chown=1001:root --chmod=0755 target/*-runner /work/application
USER 1001
ENTRYPOINT ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

## Pattern: Treat multi-platform builds as publish-time builds

When to use:

- You need `linux/amd64` and `linux/arm64` images from one pipeline.

Workflow:

1. Confirm the chosen base image publishes all target architectures.
2. Configure the builder-specific platform property.
3. Build and push in the same pipeline step.
4. Verify the resulting manifest in the registry, not just local image listings.

Reasoning:

- Docker buildx cannot load a multi-platform result into the local daemon the same way it can for a single-platform build.
- Jib multi-platform support is for JVM images only.
- Native packaging still depends on how the binary itself is produced for each target architecture.

## Pattern: Use OpenShift when the cluster owns the build

When to use:

- Source artifacts can be uploaded, but image builds must run inside OpenShift.

Setup:

```properties
quarkus.container-image.builder=openshift
quarkus.container-image.build=true
quarkus.openshift.build-strategy=binary
```

Notes:

- The Quarkus Kubernetes/OpenShift integration creates the needed `BuildConfig` and `ImageStream` resources.
- This is a deployment-platform pattern, not just a local packaging choice.

## Pattern: Use Buildpacks for standardized platform-curated images

When to use:

- The platform team standardizes on builder and run images, and Dockerfile ownership should stay low.

Setup:

```properties
quarkus.container-image.builder=buildpack
quarkus.buildpack.jvm-builder-image=<builder-image-approved-for-your-platform>
```

Guidelines:

- Pass `quarkus.container-image.build=true` at build time instead of hardcoding it in application properties.
- Treat builder and run image selection as part of platform policy.

## Pattern: Podman as a Docker-compatible runtime with a few extra checks

When to use:

- The team runs Podman locally or in CI and wants Quarkus image builds to behave predictably.

Guidance:

1. If Docker-compatible behavior is enough, the Docker extension can still work against Podman.
2. Use the Podman extension when you need Podman-specific platform flags or Podman-specific settings.
3. On Linux rootless setups, export `DOCKER_HOST` to Podman's socket.
4. Prefer fully qualified image names to avoid short-name registry prompts in non-interactive tooling.

## Pattern: Make CI image names explicit

When to use:

- The same project builds in multiple branches, registries, or environments.

Example:

```properties
quarkus.container-image.image=${CI_REGISTRY_IMAGE}
quarkus.container-image.additional-tags=${CI_COMMIT_REF_SLUG},latest
```

Benefits:

- CI decides the final repository path.
- Quarkus stays responsible for build orchestration.
- The project avoids hardcoding environment-specific registry paths.

## See Also

- [./configuration.md](./configuration.md) - Property keys used by these workflows
- [./gotchas.md](./gotchas.md) - Failure modes to check before adopting a pattern
- [../tooling/patterns.md](../tooling/patterns.md) - CLI-oriented build and push workflows
