# Quarkus Packaging Containers Gotchas

Common container packaging pitfalls, symptoms, and fixes.

## Builder selection and extension mixing

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Build fails because multiple container-image extensions are present | Quarkus cannot infer which builder to use | Remove unused extensions or set `quarkus.container-image.builder` explicitly |
| Image build behavior changes unexpectedly across environments | Different builders are being invoked by CLI, CI, or local config | Make the builder explicit and keep one main packaging path per project |

## Image naming and push surprises

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Image is pushed to Docker Hub unexpectedly | `quarkus.container-image.push=true` was set without `quarkus.container-image.registry` | Set the registry explicitly for any non-Docker-Hub workflow |
| `group`, `name`, or `tag` changes seem ignored | `quarkus.container-image.image` is also set | Use either the full image property or the decomposed properties, not both |
| Push fails in CI with auth errors | Registry credentials were not provided in the builder-specific environment | Supply `username` and `password`, or use the CLI's stdin-based push flow |

## Multi-platform caveats

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Docker buildx multi-platform build does not appear in `docker images` | Multi-platform buildx outputs are not loaded into the local daemon like single-platform builds | Push directly as part of the build and inspect the registry manifest instead |
| Jib multi-platform build fails for native packaging expectations | Jib platform manifests do not solve native cross-compilation | Build native binaries per target architecture and package them intentionally |
| Multi-platform build fails before packaging completes | The selected base image does not publish all requested architectures | Choose a manifest-list base image that includes every target platform |

## Layout and entrypoint mismatches

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Custom Jib entrypoint cannot find `quarkus-run.jar` or launch scripts | The entrypoint assumed the wrong jar layout | Match the entrypoint to the selected jar packaging type and its working directory |
| Custom Dockerfile copies the wrong artifact | The Dockerfile and chosen packaging mode disagree about file names or paths | Align the Dockerfile with JVM vs native output and with the generated Quarkus layout |
| Container starts but is unreachable | The process is not binding to container-facing interfaces | For native images, keep `-Dquarkus.http.host=0.0.0.0` in the container entrypoint or equivalent runtime args |

## Base image tradeoffs

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Native container is small but missing runtime libraries or utilities | `ubi9-quarkus-micro-image` is intentionally minimal | Add required libraries in a multi-stage Dockerfile or move to `ubi9-minimal` |
| OpenShift or Jib runtime image differs from Dockerfile-based builds | Different builders use different base-image configuration models | Pin base images explicitly per builder when reproducibility matters |
| JVM image works locally but not after Java target changes | The builder default base image changed with the application's Java level | Pin `base-jvm-image` when the runtime image must stay stable across upgrades |

## Podman-specific environment issues

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Quarkus or related container tooling cannot reach Podman on Linux | `DOCKER_HOST` is not pointed at Podman's user socket | Export `DOCKER_HOST=unix://$(podman info --format '{{.Host.RemoteSocket.Path}}')` |
| Non-interactive pulls stall or fail when using short image names with Podman | Podman prompts for registry resolution | Prefer fully qualified image names; for Testcontainers-heavy environments, disable short-name prompting in Podman config |
| Mac or Windows Podman setup behaves differently from Linux | Containers run through a Podman Machine VM instead of directly on the host | Treat Podman Desktop and machine configuration as part of the environment, not the Quarkus app |

## Buildpack-specific caveat

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Buildpack build recursively triggers during packaging flows | `quarkus.container-image.build=true` is baked into normal project properties | Pass build enablement at command time unless always-on behavior is truly intended |

## See Also

- [./configuration.md](./configuration.md) - The exact properties behind these behaviors
- [./patterns.md](./patterns.md) - Safer builder-selection and CI packaging workflows
- [../tooling/gotchas.md](../tooling/gotchas.md) - CLI-side image and push command pitfalls
