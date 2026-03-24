# Quarkus Packaging Containers Configuration Reference

Use this file for high-value container image properties in Quarkus.

## Core image properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.container-image.build` | unset | The build should also produce a container image |
| `quarkus.container-image.push` | unset | The image should be pushed during the build |
| `quarkus.container-image.builder` | auto/required if ambiguous | More than one builder extension exists or you want explicit selection |
| `quarkus.container-image.registry` | unset | Images should be pushed to a specific registry |
| `quarkus.container-image.group` | unset | The image repository path needs an organization or namespace |
| `quarkus.container-image.name` | `quarkus.application.name` | The image name should differ from the application name |
| `quarkus.container-image.tag` | application version | The main tag should be explicit |
| `quarkus.container-image.additional-tags` | unset | The same image should also be published as `latest`, `stable`, or similar |
| `quarkus.container-image.image` | unset | You want to set the full image reference in one property |
| `quarkus.container-image.username` | unset | Registry authentication should be provided through Quarkus |
| `quarkus.container-image.password` | unset | Registry authentication should be provided through Quarkus |
| `quarkus.container-image.insecure` | `false` | The target registry is insecure and you accept that tradeoff |
| `quarkus.container-image.labels."name"` | unset | OCI labels or metadata should be attached to the image |

## Builder selection

```properties
quarkus.container-image.builder=jib
```

Supported builder values in this module's scope:

- `jib`
- `docker`
- `podman`
- `openshift`
- `buildpack`

Set this whenever multiple container-image extensions are present.

## Jib properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.jib.base-jvm-image` | Quarkus-selected UBI OpenJDK runtime image | The JVM runtime image must be pinned or replaced |
| `quarkus.jib.base-native-image` | `quay.io/quarkus/ubi9-quarkus-micro-image:2.0` | Native runtime image should be customized |
| `quarkus.jib.jvm-entrypoint` | generated | You need a fully custom JVM entrypoint |
| `quarkus.jib.native-entrypoint` | generated | You need a fully custom native entrypoint |
| `quarkus.jib.jvm-arguments` | generated defaults | JVM startup flags belong in the image entrypoint |
| `quarkus.jib.native-arguments` | unset | Native startup flags belong in the image entrypoint |
| `quarkus.jib.environment-variables."name"` | unset | Environment variables should be baked into the image |
| `quarkus.jib.platforms` | current platform | A JVM image manifest should target more than one platform |
| `quarkus.jib.working-directory` | `/home/jboss` | The base image layout or custom entrypoint needs a different workdir |

Layout note:

- With `fast-jar`, Jib places the application under `/work` and uses `/work` as the working directory.
- With `legacy-jar` or `uber-jar`, Jib uses `/app`.
- Native Jib images also run from `/work`.

## Docker properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.docker.dockerfile-jvm-path` | `src/main/docker/Dockerfile.jvm` | The JVM Dockerfile lives elsewhere |
| `quarkus.docker.dockerfile-native-path` | `src/main/docker/Dockerfile.native` | The native Dockerfile lives elsewhere |
| `quarkus.docker.build-args."name"` | unset | Build args should flow into `docker build` |
| `quarkus.docker.cache-from` | unset | Layer cache should reuse named images |
| `quarkus.docker.network` | unset | Build-time `RUN` steps need a specific network mode |
| `quarkus.docker.executable-name` | auto | Docker-compatible runtime detection must be overridden |
| `quarkus.docker.additional-args` | unset | Extra runtime-specific flags are needed |
| `quarkus.docker.buildx.platform` | unset | Docker buildx should target one or more platforms |
| `quarkus.docker.buildx.output` | unset | Buildx output mode should be controlled explicitly |
| `quarkus.docker.buildx.progress` | `auto` | CI logs should show plain buildx output |

## Podman properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.podman.dockerfile-jvm-path` | `src/main/docker/Dockerfile.jvm` | Podman should use a different JVM Dockerfile |
| `quarkus.podman.dockerfile-native-path` | `src/main/docker/Dockerfile.native` | Podman should use a different native Dockerfile |
| `quarkus.podman.build-args."name"` | unset | Build args should flow into `podman build` |
| `quarkus.podman.cache-from` | unset | Cache sources should be reused |
| `quarkus.podman.network` | unset | Build-time networking should be controlled |
| `quarkus.podman.executable-name` | auto | The podman binary name/path must be overridden |
| `quarkus.podman.additional-args` | unset | Extra Podman flags are needed |
| `quarkus.podman.platform` | unset | Podman should build for one or more platforms |
| `quarkus.podman.tls-verify` | `true` | Registry TLS verification must be relaxed |

## OpenShift properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.openshift.build-strategy` | `binary` | You need to switch between OpenShift binary and Docker strategies |
| `quarkus.openshift.base-jvm-image` | Quarkus-selected UBI builder image | The cluster-side JVM builder image must change |
| `quarkus.openshift.base-native-image` | `quay.io/quarkus/ubi9-quarkus-native-binary-s2i:2.0` | The cluster-side native builder image must change |
| `quarkus.openshift.jvm-dockerfile` | `src/main/docker/Dockerfile.jvm` | OpenShift should build from a custom JVM Dockerfile |
| `quarkus.openshift.native-dockerfile` | `src/main/docker/Dockerfile.native` | OpenShift should build from a custom native Dockerfile |
| `quarkus.openshift.jar-directory` | image-specific | A custom S2I image expects jars in a different directory |
| `quarkus.openshift.jar-file-name` | image-specific | The S2I image requires a fixed jar name |
| `quarkus.openshift.native-binary-directory` | image-specific | The S2I image expects the native binary in a different directory |
| `quarkus.openshift.native-binary-file-name` | image-specific | The S2I image requires a fixed native filename |
| `quarkus.openshift.build-timeout` | `5M` | Cluster builds need more time |
| `quarkus.openshift.image-push-secret` | unset | OpenShift should push to an external registry |

## Buildpack properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.buildpack.jvm-builder-image` | `paketocommunity/builder-ubi-base:latest` | JVM images should use a specific builder |
| `quarkus.buildpack.native-builder-image` | unset | Native packaging needs a native-capable builder |
| `quarkus.buildpack.run-image` | builder-selected | The runtime image must be pinned |
| `quarkus.buildpack.lifecycle-image` | unset | The lifecycle image should be overridden |
| `quarkus.buildpack.builder-env."name"` | unset | Buildpack environment variables must be passed in |
| `quarkus.buildpack.use-daemon` | `true` | The build should avoid daemon mode and write layers directly to a registry |
| `quarkus.buildpack.docker-host` | auto | Docker or Podman socket detection must be overridden |
| `quarkus.buildpack.docker-socket` | auto | The mounted socket path must be specified |
| `quarkus.buildpack.docker-network` | unset | Build containers need host or custom network access |

Avoid keeping `quarkus.container-image.build=true` permanently in properties with buildpacks unless that nesting behavior is intentional.

## Base image guidance

For JVM packaging:

- Jib and OpenShift default to Quarkus-selected UBI OpenJDK images that follow the application's Java target.
- Docker and Podman use whatever the chosen Dockerfile declares.

For native packaging:

- Default to `quay.io/quarkus/ubi9-quarkus-micro-image:2.0` for small runtime images.
- Use `registry.access.redhat.com/ubi9/ubi-minimal` when the runtime image needs package installation or more shell-level utilities.
- If the native binary needs extra shared libraries, solve that in the Dockerfile or by choosing a less minimal base image.

## Multi-platform settings

Use one platform property family per builder:

```properties
quarkus.docker.buildx.platform=linux/amd64,linux/arm64
quarkus.podman.platform=linux/amd64,linux/arm64
quarkus.jib.platforms=linux/amd64,linux/arm64
```

Important limits:

- Jib multi-platform support applies to JVM images, not native cross-compilation.
- Docker buildx multi-platform builds are normally push-oriented.
- Multi-platform base images must themselves publish all requested platforms.

## Podman environment note

On Linux rootless setups, Quarkus and related tooling commonly need:

```bash
export DOCKER_HOST=unix://$(podman info --format '{{.Host.RemoteSocket.Path}}')
```

This matters even when using Docker-compatible APIs against Podman.

## See Also

- [./patterns.md](./patterns.md) - Recommended property combinations by builder and deployment style
- [./gotchas.md](./gotchas.md) - Platform, registry, and layout caveats behind these settings
- [../configuration/README.md](../configuration/README.md) - Broader configuration and profile organization guidance
