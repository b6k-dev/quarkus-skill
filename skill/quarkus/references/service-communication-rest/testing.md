# Quarkus Service Communication REST Testing Reference

Use this file for outbound REST client tests that need HTTP-level verification with WireMock.

## Why WireMock fits this module

WireMock is a strong fit when you need to test the real REST client boundary without calling the real external service.

- Stub exact HTTP paths, methods, headers, query params, and payloads.
- Return the same status codes and response bodies the downstream service would return.
- Verify client-side mapping, retries, exception translation, and timeout handling.
- Keep tests deterministic and fast enough for normal JVM test runs.

This is the right tool when mock-bean tests are too shallow and full end-to-end tests against a real dependency are too heavy.

## Pattern: Start WireMock through a Quarkus test resource

When to use:

- The REST client base URL should point at a dynamic local stub server during tests.

Example:

```java
import com.github.tomakehurst.wiremock.WireMockServer;
import io.quarkus.test.common.QuarkusTestResourceLifecycleManager;
import java.util.Map;

import static com.github.tomakehurst.wiremock.core.WireMockConfiguration.options;

public class CustomerApiWireMock implements QuarkusTestResourceLifecycleManager {
    private static WireMockServer current;
    private WireMockServer wireMock;

    public static WireMockServer server() {
        return current;
    }

    @Override
    public Map<String, String> start() {
        wireMock = new WireMockServer(options().dynamicPort());
        wireMock.start();
        current = wireMock;

        return Map.of(
                "quarkus.rest-client.customer-api.url",
                wireMock.baseUrl());
    }

    @Override
    public void stop() {
        if (wireMock != null) {
            wireMock.stop();
        }
        current = null;
    }
}
```

Use a `configKey` on the REST client so the test resource can override one stable property.

## Pattern: Exercise the real client from a `@QuarkusTest`

When to use:

- You want Quarkus to boot the real REST client bean and serialization layer.

Example:

```java
import io.quarkus.test.common.QuarkusTestResource;
import io.quarkus.test.junit.QuarkusTest;
import jakarta.inject.Inject;
import org.eclipse.microprofile.rest.client.inject.RestClient;
import org.junit.jupiter.api.Test;

import static com.github.tomakehurst.wiremock.client.WireMock.*;
import static org.junit.jupiter.api.Assertions.assertEquals;

@QuarkusTest
@QuarkusTestResource(CustomerApiWireMock.class)
class CustomerClientTest {
    @Inject
    @RestClient
    CustomerClient customers;

    @Test
    void getsCustomerFromStub() {
        CustomerApiWireMock.server().stubFor(get(urlEqualTo("/customers/42"))
                .willReturn(okJson("""
                        {"id":"42","name":"Ada"}
                        """)));

        CustomerView customer = customers.get("42");

        assertEquals("Ada", customer.name());
        CustomerApiWireMock.server().verify(getRequestedFor(urlEqualTo("/customers/42")));
    }
}
```

Assert both the returned domain value and the HTTP interaction that produced it.

## Pattern: Cover the four high-value downstream outcomes

A practical set of scenarios worth keeping together for a REST client is:

- HTTP success with a success payload.
- HTTP success with a payload that still represents business failure.
- HTTP auth or permission failure such as `401` or `403`.
- Slow downstream behavior that should trigger a timeout path.

That mix is especially valuable when the downstream API encodes some failures in the body instead of the status code.

## Pattern: Stub a business failure hidden inside `200 OK`

When to use:

- The downstream service returns `200` even when the operation-level result is unsuccessful.

Example:

```java
CustomerApiWireMock.server().stubFor(post(urlEqualTo("/messages"))
        .willReturn(okJson("""
                {"status":"FAILED","reason":"INSUFFICIENT_CREDIT"}
                """)));
```

Use this to validate body inspection and service-layer translation logic. Do not assume every failure is represented by a non-2xx status.

## Pattern: Stub authorization failures

When to use:

- The client must attach auth or tenant headers correctly.

Example:

```java
CustomerApiWireMock.server().stubFor(get(urlPathEqualTo("/reports/daily"))
        .withHeader("Authorization", matching("Bearer .+"))
        .willReturn(okJson("{" + "\"status\":\"READY\"}")));
```

Add a negative test too:

```java
CustomerApiWireMock.server().stubFor(get(urlPathEqualTo("/reports/daily"))
        .atPriority(10)
        .willReturn(aResponse().withStatus(401)));
```

This keeps header propagation and fallback behavior under test at the real HTTP boundary.

## Pattern: Simulate timeouts and retry paths

When to use:

- You need to prove timeout handling, retries, circuit breakers, or fallback behavior.

Example:

```java
CustomerApiWireMock.server().stubFor(get(urlEqualTo("/inventory/42"))
        .willReturn(aResponse()
                .withFixedDelay(3000)
                .withStatus(200)
                .withBody("{" + "\"id\":\"42\"}")));
```

Pair this with a short test timeout:

```properties
%test.quarkus.rest-client.inventory-api.read-timeout=500ms
```

Use direct client tests to validate timeout behavior at the HTTP boundary. If you need to verify service-level `@Retry`, `@CircuitBreaker`, or fallback behavior, call the CDI service method that wraps the client instead of invoking the client bean directly.

## Pattern: Separate client tests from service tests

Use two layers of tests:

- Mock-bean tests for pure service logic around the client.
- WireMock-backed Quarkus tests for the real HTTP client contract.

Do not replace one with the other. They answer different questions.

## Pattern: Reuse mappings in heavier integration tests

When to use:

- The application under test runs in a container or a broader component-test environment.

A broader component-test flow can use a WireMock container with preloaded mappings. That is a good next step once single-service Quarkus tests are not enough.

- Keep simple JVM tests with in-process WireMock for day-to-day development.
- Move stable stub files into a mounted `mappings/` directory for containerized component tests.
- Reuse the same downstream scenarios across local, CI, and broader environment tests.

## Dependency hint

One common Maven test dependency is:

```xml
<dependency>
    <groupId>org.wiremock</groupId>
    <artifactId>wiremock</artifactId>
    <scope>test</scope>
</dependency>
```

Use the WireMock artifact that matches your runtime and test stack conventions.
