# Quarkus Native Image Usage Patterns

Use these patterns for repeatable native build, metadata, and runtime-tuning workflows in Quarkus.

## Pattern: Default to local native builds on matching Linux hosts

When to use:

- Developers or CI already have Mandrel or GraalVM installed.
- The produced binary runs on the same OS and architecture as the build machine.

Example:

```bash
quarkus build --native
```

This keeps the workflow simple and usually gives the fastest edit-build-debug loop.

## Pattern: Use container builds for Linux targets from non-Linux hosts

When to use:

- Developers build on macOS or Windows but deploy Linux-native binaries.
- The team wants builder-version consistency without local GraalVM installs.

Configuration:

```properties
quarkus.native.container-build=true
quarkus.native.builder-image=mandrel
```

This produces a Linux executable that may not run on the host machine, which is expected.

## Pattern: Switch to remote container build only for remote daemons

When to use:

- Native container build fails with missing `/project/...-runner.jar` paths.
- Docker or Podman is talking to a remote daemon where bind mounts do not work.

Configuration:

```properties
quarkus.native.remote-container-build=true
```

Prefer plain `container-build` for local daemons because mounting is usually faster than copying.

## Pattern: Fix native resource issues with Quarkus-first metadata

When to use:

- Files load correctly in JVM mode but disappear in the native binary.

Configuration:

```properties
quarkus.native.resources.includes=templates/**,certs/*.pem,version.txt
```

Start with `quarkus.native.resources.includes`. Use manual `resource-config.json` only when globs are not precise enough.

## Pattern: Register reflection for JSON or object-mapping types

When to use:

- Serialization works in JVM mode but native returns empty JSON, constructor errors, or reflective lookup failures.

Example:

```java
@RegisterForReflection(targets = { User.class, UserImpl.class })
public class NativeReflectionConfig {
}
```

Prefer annotation-based registration for application-owned classes. Keep the target list narrow to limit image growth.

## Pattern: Delay runtime-sensitive class initialization

When to use:

- Native build fails with `Random/SplittableRandom` image-heap errors.
- Static initialization captures values that should change at runtime.
- Security or crypto code initializes too early.

Configuration:

```properties
quarkus.native.additional-build-args=--initialize-at-run-time=com.example.CryptoHolder
```

Move the problematic work into runtime code first if possible. Use runtime initialization as the targeted fallback, not the first instinct for whole packages.

## Pattern: Keep mutable singleton state out of static initialization

When to use:

- A singleton caches timestamps, random seeds, or environment-derived values.

Example:

```java
@Singleton
class StartupClock {
    final long startedAt = System.currentTimeMillis();
}
```

CDI singletons are usually safer than static fields for values that should be determined per process start.

## Pattern: Enable native SSL only when the app really needs it

When to use:

- The application makes HTTPS calls or uses extensions that require native SSL.
- You are consciously trading binary size for network capabilities.

Configuration:

```properties
quarkus.ssl.native=true
```

If HTTPS is not needed, keeping SSL disabled can reduce the native binary size substantially.

## Pattern: Bake trust material only for immutable environments

When to use:

- You intentionally want the binary to carry its own trust store.
- Runtime certificate rotation is not required.

Configuration:

```properties
quarkus.native.additional-build-args=-J-Djavax.net.ssl.trustStore=/tmp/mycerts,-J-Djavax.net.ssl.trustStorePassword=changeit
```

This is strong for tightly controlled deployments but awkward when certificates differ by environment or rotate often.

## Pattern: Turn on reports before deep native debugging

When to use:

- You need to explain why code is reachable, why the image grew, or why analysis fails.

Configuration:

```properties
quarkus.native.enable-reports=true
```

Call-tree and used-class reports are often the fastest way to turn a vague native failure into something explainable.

## Pattern: Build a debug-friendly binary on purpose

When to use:

- You need `gdb`, `perf`, or source-aware backtraces.

Configuration:

```properties
quarkus.native.debug.enabled=true
quarkus.native.additional-build-args=-H:-OmitInlinedMethodDebugLineInfo
```

Keep this for diagnosis builds rather than every production artifact.

## Pattern: Add only the runtime monitoring features you plan to use

When to use:

- Production or staging needs deeper diagnostics without switching back to JVM mode.

Configuration:

```properties
quarkus.native.monitoring=jfr,heapdump,threaddump
```

Then run with runtime flags such as `-XX:+FlightRecorder` only when collecting data.

## Pattern: Target older CPUs explicitly for broader deployment compatibility

When to use:

- Builds happen on newer hardware than production.
- Startup fails with unsupported CPU feature messages.

Configuration:

```properties
quarkus.native.march=compatibility
```

Use a specific `-march` only when you know the deployment fleet precisely and want tighter control.
