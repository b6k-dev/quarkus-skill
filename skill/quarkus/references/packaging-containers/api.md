# Quarkus Packaging Containers API Reference

Use this file for container-image extension entry points, image build commands, and short examples.

## Extension IDs

Add exactly one of these unless you intentionally need more than one available and will select via configuration:

```text
quarkus-container-image-jib
quarkus-container-image-docker
quarkus-container-image-podman
quarkus-container-image-openshift
quarkus-container-image-buildpack
```

CLI example:

```bash
quarkus ext add quarkus-container-image-jib
```

Build tool equivalents:

```bash
./mvnw quarkus:add-extension -Dextensions='quarkus-container-image-jib'
./gradlew addExtension --extensions='quarkus-container-image-jib'
```

## Generic build trigger

Quarkus performs container packaging when image build or push is enabled.

```properties
quarkus.container-image.build=true
```

Compact command equivalents:

```bash
quarkus build
./mvnw package -Dquarkus.container-image.build=true
./gradlew build -Dquarkus.container-image.build=true
```

## Generic push trigger

```properties
quarkus.container-image.push=true
```

With no registry configured, pushes default to `docker.io`.

## Full image naming

Use the decomposed form when CI assembles names from separate inputs:

```properties
quarkus.container-image.registry=ghcr.io
quarkus.container-image.group=acme-platform
quarkus.container-image.name=orders-service
quarkus.container-image.tag=1.2.3
quarkus.container-image.additional-tags=latest,stable
```

Use the full form when the registry path is already known:

```properties
quarkus.container-image.image=ghcr.io/acme-platform/orders-service:1.2.3
```

`quarkus.container-image.image` overrides the decomposed properties.

## Select a builder explicitly

```properties
quarkus.container-image.builder=jib
```

Useful when multiple container-image extensions exist in the project.

## Quarkus CLI image commands

```bash
quarkus image build
quarkus image build jib
quarkus image build docker
quarkus image build buildpack --builder-image paketocommunity/builder-ubi-base:latest
quarkus image push --registry=ghcr.io --registry-username=<user> --registry-password-stdin
```

Use CLI commands when you want container packaging without permanently changing project extensions.

## Jib examples

Jib is property-driven and does not rely on Dockerfiles for the application image layout.

```properties
quarkus.container-image.builder=jib
quarkus.jib.base-jvm-image=<supported-jvm-runtime-image>
quarkus.jib.base-native-image=<supported-native-runtime-image>
```

Choose base images that match your Java level, Quarkus version, and runtime requirements.

Jib copies extra files from:

```text
src/main/jib
```

## Docker and Podman examples

Docker-style builders use Dockerfiles, by default:

```text
src/main/docker/Dockerfile.jvm
src/main/docker/Dockerfile.native
```

Typical selection:

```properties
quarkus.container-image.builder=docker
quarkus.docker.dockerfile-jvm-path=src/main/docker/Dockerfile.jvm
quarkus.docker.dockerfile-native-path=src/main/docker/Dockerfile.native
```

Podman variant:

```properties
quarkus.container-image.builder=podman
quarkus.podman.dockerfile-native-path=src/main/docker/Dockerfile.native
```

## OpenShift example

```properties
quarkus.container-image.builder=openshift
quarkus.openshift.build-strategy=binary
```

Use this when the cluster should perform the image build.

## Buildpack example

```properties
quarkus.container-image.builder=buildpack
quarkus.buildpack.jvm-builder-image=<builder-image-supported-in-your-environment>
```

For native builds, set a native-capable builder image explicitly.

## Native runtime image example

Minimal native Dockerfile pattern:

```dockerfile
FROM <supported-quarkus-native-runtime-image>
WORKDIR /work/
COPY --chmod=0755 target/*-runner /work/application
EXPOSE 8080
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

Use `ubi9-minimal` instead when you need package-manager access or extra runtime utilities.

## See Also

- [./configuration.md](./configuration.md) - Property-level control for platforms, credentials, Dockerfiles, and base images
- [./patterns.md](./patterns.md) - How to choose among builders in practice
- [../tooling/api.md](../tooling/api.md) - Quarkus CLI command model and image subcommands
