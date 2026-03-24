# Quarkus Native Image Testing Reference

Use this file for testing the produced native executable rather than only validating JVM-mode behavior.

## Why this gets its own page

Native tests answer a different question than `@QuarkusTest`: does the produced artifact behave correctly once Quarkus and GraalVM or Mandrel have finished ahead-of-time processing?

- They catch reflection, resource, proxy, and class-init issues that JVM tests often miss.
- They exercise the real native executable or produced artifact instead of the in-process test harness.
- They are slower and narrower than normal JVM tests, so they need deliberate scope.

## Pattern: Share assertions, specialize the native launcher

When to use:

- You want one set of endpoint assertions in JVM mode and native mode.

Example:

```java
import io.quarkus.test.junit.QuarkusIntegrationTest;
import io.quarkus.test.junit.QuarkusTest;

@QuarkusTest
class GreetingResourceTest {
}

@QuarkusIntegrationTest
class GreetingResourceIT extends GreetingResourceTest {
}
```

Keep the JVM test as the main home of the assertions. Let the `@QuarkusIntegrationTest` subclass reuse them against the produced artifact.

## Pattern: Run native tests as part of the native build flow

Typical commands:

```bash
./mvnw verify -Dnative
./gradlew testNative
```

- In Maven, the common pattern is a `native` profile that flips `quarkus.native.enabled` on and runs integration tests.
- In Gradle, `testNative` builds the native image first and then runs the native tests.

## Pattern: Keep native tests HTTP-level and artifact-focused

When to use:

- The value is proving the packaged app works externally.

Good native test targets:

- HTTP endpoints and serialization.
- Database schema/resources needed by the packaged app.
- TLS/client connectivity that depends on native SSL.
- Startup behavior and production wiring.

Poor native test targets:

- Pure unit logic with no artifact boundary.
- Tests that inspect implementation details unavailable from outside the process.

## Pattern: Exclude tests that do not make sense in integration-test modes

When to use:

- A shared test should run in JVM mode but not against the packaged artifact.

Example:

```java
import io.quarkus.test.junit.DisabledOnIntegrationTest;
import org.junit.jupiter.api.Test;

class SomeTest {
    @Test
    @DisabledOnIntegrationTest
    void jvmOnlyCheck() {
    }
}
```

Remember that this disables the test for all `@QuarkusIntegrationTest` executions, including non-native artifact modes.

## Pattern: Adjust startup wait time for slower native test environments

When to use:

- CI runners or larger apps take longer than the default 60 seconds to start.

Example:

```bash
./mvnw verify -Dnative -Dquarkus.test.wait-time=300
```

Increase this sparingly; persistent slowness can still point to a real startup issue.

## Pattern: Use profile overrides intentionally in native tests

When to use:

- The executable should run with test-only runtime settings.

Example:

```bash
./mvnw verify -Dnative -Dquarkus.test.integration-test-profile=test
```

This changes the profile used when the native executable runs during the test. If you instead build the executable with `-Dquarkus.profile=test`, you are also changing what gets packaged into the binary.

## Pattern: Include test-only native resources without polluting production

When to use:

- Native tests need extra seed files or fixtures that production should not ship.

Example:

```properties
quarkus.native.resources.includes=version.txt
%test.quarkus.native.resources.includes=version.txt,import-dev.sql
%test.quarkus.hibernate-orm.schema-management.strategy=drop-and-create
%test.quarkus.hibernate-orm.sql-load-script=import-dev.sql
```

This keeps the production binary lean while still letting native tests package what they need.

## Pattern: Re-run tests against an existing binary

When to use:

- You already paid the native build cost and only need another test pass.

Example:

```bash
./mvnw test-compile failsafe:integration-test -Dnative
./mvnw test-compile failsafe:integration-test -Dnative -Dnative.image.path=target/app-runner
```

This is useful for flaky-environment diagnosis or iterative test debugging.

## Pattern: Use the native image agent carefully

When to use:

- JVM integration tests can help discover missing reflection, proxy, or resource metadata.

Guidance:

- Treat generated config as informative first.
- Review and check in only the minimal stable metadata you actually want.
- Regenerate after Mandrel upgrades if you rely on agent-produced files.

Blindly applying generated agent output can make native behavior depend on incidental test coverage.
