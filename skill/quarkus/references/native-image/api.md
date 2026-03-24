# Quarkus Native Image API Reference

Use this file for native build entry points, annotations, and copy-ready native metadata examples.

## Local native build

```bash
quarkus build --native
./mvnw package -Dnative
./gradlew build -Dquarkus.native.enabled=true
```

Use a local build when the target OS matches the machine doing the build and GraalVM or Mandrel is installed.

## Containerized Linux-native build

```bash
quarkus build --native -Dquarkus.native.container-build=true
./mvnw package -Dnative -Dquarkus.native.container-build=true
./gradlew build -Dquarkus.native.enabled=true -Dquarkus.native.container-build=true
```

Use this when you need a Linux native executable without installing GraalVM locally, or when the host OS is not Linux.

## Select the container runtime explicitly

```properties
quarkus.native.container-build=true
quarkus.native.container-runtime=podman
```

Set the runtime only when auto-detection is wrong or you want stable CI behavior.

## Switch the builder image

```properties
quarkus.native.builder-image=mandrel
quarkus.native.builder-image.pull=always
```

Use `mandrel` for the default Quarkus path. Use `graalvm` or a full image path only when you intentionally need that distribution.

## Re-run native tests against an existing binary

```bash
./mvnw test-compile failsafe:integration-test -Dnative
./mvnw test-compile failsafe:integration-test -Dnative -Dnative.image.path=target/app-runner
```

Useful when a native build is expensive and you only need to rerun the test suite.

## Native integration test annotation

```java
import io.quarkus.test.junit.QuarkusIntegrationTest;

@QuarkusIntegrationTest
class GreetingResourceIT extends GreetingResourceTest {
}
```

Use `@QuarkusIntegrationTest` for tests that must exercise the produced artifact instead of the in-process JVM test runtime.

## Skip a test in integration-test modes

```java
import io.quarkus.test.junit.DisabledOnIntegrationTest;
import org.junit.jupiter.api.Test;

class SomeTest {
    @Test
    @DisabledOnIntegrationTest
    void onlyRunsInJvmMode() {
    }
}
```

This disables the test in all integration-test artifact modes, not only native.

## Register classes for reflection

```java
import io.quarkus.runtime.annotations.RegisterForReflection;

@RegisterForReflection(targets = { User.class, UserImpl.class })
public class NativeReflectionConfig {
}
```

Prefer annotations for application-owned types. Reach for JSON only when you need finer control or third-party metadata files.

## Register proxy interfaces

```java
import io.quarkus.runtime.annotations.RegisterForProxy;

@RegisterForProxy(targets = { MyInterface.class, MySecondInterface.class })
public class NativeProxyConfig {
}
```

The interface order matters for dynamic proxies.

## Register resources and bundles

```java
import io.quarkus.runtime.annotations.RegisterResourceBundle;
import io.quarkus.runtime.annotations.RegisterResources;

@RegisterResourceBundle(bundleName = "messages")
@RegisterResources(globs = { "templates/**", "certs/*.pem" })
public class NativeResourcesConfig {
}
```

Quarkus already includes `META-INF/resources`; use explicit registration for everything else.

## Include resources with configuration

```properties
quarkus.native.resources.includes=foo/**,bar/**/*.txt
```

Use this for simple resource inclusion; it is usually easier than authoring `resource-config.json`.

## Delay class initialization to runtime

```properties
quarkus.native.additional-build-args=--initialize-at-run-time=com.example.CryptoHolder\,org.acme.SomeOtherClass
```

Use this when static initialization bakes runtime-only state into the image.

## Enable native SSL explicitly

```properties
quarkus.ssl.native=true
```

Needed when no Quarkus extension turns it on for you, but the app still needs HTTPS or other native SSL support.

## Native metadata JSON location

Put manual files under:

```text
src/main/resources/META-INF/native-image/<group-id>/<artifact-id>/
```

Typical files:

- `reflect-config.json`
- `resource-config.json`
- `proxy-config.json`

Avoid placing custom files directly under `src/main/resources/META-INF/native-image/`; Quarkus also writes metadata there.
