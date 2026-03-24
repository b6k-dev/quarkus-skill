# Quarkus Native Image Gotchas

Common native-image pitfalls, symptoms, and fixes.

## Build environment mismatches

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Native build works locally but the binary does not run in the target container | The binary was built against a different OS or UBI level than the runtime image | Use a Linux container build for Linux targets and align the runtime base image with the builder image family |
| Container native build fails with `NoSuchFileException: /project/...-runner.jar` | `quarkus.native.container-build=true` is talking to a remote daemon | Switch to `quarkus.native.remote-container-build=true` |
| Native app fails at startup on older machines with missing CPU features | The binary was built for a newer CPU baseline | Set `quarkus.native.march=compatibility` or an explicit supported target |

## Closed-world surprises

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| JSON serialization returns empty objects or constructor errors only in native mode | Types used reflectively were removed from the image | Register the classes for reflection with `@RegisterForReflection` or `reflect-config.json` |
| `ClassLoader.getResource*()` or similar works in JVM mode but fails in native mode | The resource was not included in the native binary | Add it via `quarkus.native.resources.includes` or `resource-config.json` |
| Dynamic proxy creation fails at runtime | Proxy interfaces were not registered at image build time | Register them with `@RegisterForProxy` or `proxy-config.json` |
| `ServiceLoader`-style discovery breaks only in native mode | `META-INF/services` entries were not registered | Enable `quarkus.native.auto-service-loader-registration` or provide explicit metadata |

## Class initialization traps

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Native build fails with `Random/SplittableRandom` image-heap errors | Static initialization created runtime-sensitive state at build time | Move the work to runtime code or add targeted `--initialize-at-run-time` settings |
| A timestamp, random value, or environment-derived value never changes between restarts | Static field initialization ran at image build time | Avoid static initialization for mutable runtime state; prefer CDI-managed instances |
| Crypto, DNS, or security code triggers opaque native build failures | A library static initializer is doing runtime-only work too early | Trace object instantiation, then target the exact class or move initialization into a method executed at runtime |

## SSL and certificate assumptions

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| HTTPS calls fail with a message saying SSL support is disabled | Native SSL was not enabled for the binary | Set `quarkus.ssl.native=true` or add/use an extension that enables it automatically |
| Runtime `javax.net.ssl.trustStore` changes do nothing | Certificates were baked in at native build time | Rebuild with the new trust store or prefer runtime TLS configuration for environment-specific certs |
| Native binary is unexpectedly much larger | SSL, extra charsets, or broad metadata inclusion increased the image size | Recheck `quarkus.ssl.native`, `quarkus.native.add-all-charsets`, and broad reflection/resource registration |

## Troubleshooting mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Native build uses huge memory or gets killed in CI | The `native-image` heap is too large for the host | Set `quarkus.native.native-image-xmx` to a realistic limit and expect slower builds |
| A class-init or reachability issue is still unclear after reading the top error | Only the final failure is visible | Enable `quarkus.native.enable-reports`, consider `-H:+PrintClassInitialization`, and use tracing flags for the specific object or class |
| `gdb` or `perf` gives weak stack traces | The binary lacks debug symbols or local symbols | Rebuild with `quarkus.native.debug.enabled=true`; add extra debug flags only for diagnosis builds |
| Native tests time out even though the app eventually starts | Native startup under test is slower than the default wait window | Increase `quarkus.test.wait-time` |

## Metadata file placement

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Manual `resource-config.json` seems ignored or gets overwritten | The file was placed directly under `META-INF/native-image/` | Put it under `src/main/resources/META-INF/native-image/<group-id>/<artifact-id>/` |
| Generated native-image agent files make later builds unstable or surprising | Generated config was applied blindly | Treat agent output as informative first; review and check in only the minimal stable metadata |
