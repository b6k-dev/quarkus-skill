# Quarkus Native Image Configuration Reference

Use this file for the native-image properties that change build strategy, binary contents, or native runtime diagnostics in meaningful ways.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.native.enabled` | `false` | The build should produce a native executable |
| `quarkus.native.container-build` | auto/off | Native build should run in a container runtime |
| `quarkus.native.remote-container-build` | `false` | The build uses a remote Docker daemon and volume mounts fail |
| `quarkus.native.builder-image` | `mandrel` | You need a specific Mandrel/GraalVM builder image or UBI level |
| `quarkus.native.builder-image.pull` | `always` | Local development should avoid pulling on every build |
| `quarkus.native.additional-build-args` | none | You need raw `native-image` flags such as class-init, tracing, or static build args |
| `quarkus.native.additional-build-args-append` | none | You want command-line overrides without replacing checked-in defaults |
| `quarkus.native.resources.includes` | none | Classpath resources outside `META-INF/resources` must be packaged |
| `quarkus.native.auto-service-loader-registration` | `false` | The app depends on `META-INF/services` discovery that native analysis misses |
| `quarkus.ssl.native` | extension-driven | HTTPS or native SSL is needed but no extension auto-enables it |
| `quarkus.native.native-image-xmx` | unset | Native builds run out of memory or need tighter CI memory limits |
| `quarkus.native.debug.enabled` | `false` | You need debug symbols and source cache for `gdb` or profiling |
| `quarkus.native.monitoring` | none | JFR, heap dumps, thread dumps, or other VM-style diagnostics are needed |
| `quarkus.native.enable-reports` | `false` | You need call-tree and used-class reports to explain native reachability |
| `quarkus.native.march` | host-biased | The binary must run on older CPUs or on a known target architecture |
| `quarkus.test.wait-time` | `60` seconds | Native tests need more startup time before failing |
| `quarkus.test.integration-test-profile` | `prod` runtime | Native tests should run the executable with a different profile |

## Local versus container build

```properties
quarkus.native.container-build=true
quarkus.native.container-runtime=podman
quarkus.native.builder-image=mandrel
```

- Prefer local builds when the build host matches the deployment target and GraalVM or Mandrel is already installed.
- Prefer container builds when you need a Linux binary from macOS or Windows, or want tighter builder-version consistency.
- Use `quarkus.native.remote-container-build=true` only for remote daemons; local volume mounts are faster.

## Builder image compatibility boundary

```properties
quarkus.native.builder-image=<builder-image-matching-your-quarkus-version-and-runtime-base>
```

Use a builder image line that matches both your Quarkus version and the runtime base image family you plan to ship. If the builder and runtime base image families diverge, the produced Linux binary may not run in the final container as expected.

## Additional native-image arguments

```properties
quarkus.native.additional-build-args=--initialize-at-run-time=com.example.CryptoHolder\,-H:+PrintClassInitialization
```

- Put stable project defaults in `quarkus.native.additional-build-args`.
- Put temporary troubleshooting flags at the command line with `quarkus.native.additional-build-args-append`.
- Escape commas in property values with `\,` when they belong inside a single native-image argument.

## Resource inclusion

```properties
quarkus.native.resources.includes=version.txt,certs/*.pem,templates/**
```

Use globs first. Reach for `resource-config.json` only when you need regex precision or metadata not covered well by globs.

## Native SSL

```properties
quarkus.ssl.native=true
```

- Many Quarkus client extensions enable native SSL automatically.
- Set this explicitly when the application still needs HTTPS support and no extension turns it on.
- Turning it off can noticeably reduce binary size, but HTTPS calls will fail.

## Build-time trust store baking

```properties
quarkus.native.additional-build-args=-J-Djavax.net.ssl.trustStore=/tmp/mycerts,-J-Djavax.net.ssl.trustStorePassword=changeit
```

Use this only when you intentionally want immutable certificates baked into the binary. Prefer runtime TLS configuration when certificates differ across environments.

## Build memory tuning

```properties
quarkus.native.native-image-xmx=5g
```

Lower this only when builds fail from host memory pressure. Smaller heaps can make native builds much slower.

## Debug and reports

```properties
quarkus.native.debug.enabled=true
quarkus.native.enable-reports=true
```

- `debug.enabled` adds debug symbols and source cache for debugging and profiling.
- `enable-reports` adds call-tree and used-class reports that help explain why code is included.

## Runtime diagnostics support

```properties
quarkus.native.monitoring=jfr,heapdump,threaddump
```

Add only the monitoring features you actually plan to use; native binaries do not include all JVM-style diagnostics by default.

## CPU compatibility

```properties
quarkus.native.march=compatibility
```

Use this when building on newer CPUs but deploying to older hosts. Leave the default when build and run targets are tightly controlled and performance matters more.
