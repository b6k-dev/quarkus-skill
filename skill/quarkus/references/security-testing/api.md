# Quarkus Security Testing API Reference

Use this file for security-testing APIs with short, copy-ready examples.

## Test dependency

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-test-security</artifactId>
    <scope>test</scope>
</dependency>
```

## Disable authorization checks

```java
import io.quarkus.test.security.TestSecurity;
import org.junit.jupiter.api.Test;

@Test
@TestSecurity(authorizationEnabled = false)
void allowsAccessWithoutRealAuth() {
}
```

Use this for endpoint or service tests that do not care which identity authenticated.

## Run as a specific user

```java
import io.quarkus.test.security.TestSecurity;
import io.quarkus.test.junit.QuarkusTest;
import io.restassured.RestAssured;
import org.junit.jupiter.api.Test;

@QuarkusTest
class GreetingResourceTest {
    @Test
    @TestSecurity(user = "alice", roles = { "admin", "user" })
    void callsSecuredEndpoint() {
        RestAssured.when()
                .get("/greeting")
                .then()
                .statusCode(200);
    }
}
```

`@TestSecurity` works with `@QuarkusTest`, not `@QuarkusIntegrationTest`.

## Add custom identity attributes

```java
import io.quarkus.security.identity.SecurityIdentity;
import io.quarkus.test.security.TestSecurity;
import io.quarkus.test.security.SecurityAttribute;
import io.quarkus.test.security.TestSecurity.AttributeType;
import jakarta.inject.Inject;
import org.junit.jupiter.api.Test;

class IdentityAttributeTest {
    @Inject
    SecurityIdentity identity;

    @Test
    @TestSecurity(
            user = "alice",
            roles = "admin",
            attributes = @SecurityAttribute(key = "tenant-id", value = "42", type = AttributeType.LONG))
    void readsAugmentedAttribute() {
        Long tenantId = identity.getAttribute("tenant-id");
    }
}
```

## Grant permissions

```java
import io.quarkus.test.security.TestSecurity;
import org.junit.jupiter.api.Test;

@Test
@TestSecurity(user = "alice", permissions = "see:detail")
void passesPermissionsAllowedCheck() {
}
```

Use this when code is secured with `@PermissionsAllowed` instead of only role checks.

## Reuse a common test identity

```java
import io.quarkus.test.security.TestSecurity;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@TestSecurity(user = "alice", roles = { "admin", "user" })
public @interface AdminUser {
}
```

Meta-annotations keep repeated security fixtures short and consistent.

## Select an authentication mechanism

```java
import io.quarkus.test.security.TestSecurity;
import org.junit.jupiter.api.Test;

@Test
@TestSecurity(user = "alice", roles = "user", authMechanism = "basic")
void exercisesBasicAuthPath() {
}
```

Use `authMechanism` when the app combines mechanisms or uses path-specific auth policies.

## Prefer real bearer tokens for HTTP-level auth tests

```java
import io.quarkus.test.junit.QuarkusTest;
import io.restassured.RestAssured;
import org.junit.jupiter.api.Test;

@QuarkusTest
class BearerTokenTest {
    @Test
    void callsEndpointWithToken() {
        String token = tokenFor("alice");

        RestAssured.given()
                .auth().oauth2(token)
                .when().get("/api/admin")
                .then().statusCode(200);
    }

    private String tokenFor(String user) {
        return "<token>";
    }
}
```

Use this style when the endpoint depends on claims, token parsing, or the real bearer auth mechanism.

## Start a custom auth dependency with a test resource

```java
import io.quarkus.test.common.QuarkusTestResourceLifecycleManager;
import java.util.Map;

public class KeycloakTestResource implements QuarkusTestResourceLifecycleManager {
    @Override
    public Map<String, String> start() {
        return Map.of(
                "quarkus.oidc.auth-server-url", "http://localhost:8180/realms/quarkus",
                "quarkus.oidc.client-id", "quarkus-app",
                "quarkus.oidc.credentials.secret", "secret");
    }

    @Override
    public void stop() {
    }
}
```

Use a test resource when Dev Services are not enough or the provider must be customized programmatically.

## See Also

- [./patterns.md](./patterns.md) - When to choose `@TestSecurity`, bearer tokens, or Dev Services
- [../security-jwt/README.md](../security-jwt/README.md) - Token-building approaches for JWT-backed tests
- [../security-oidc/README.md](../security-oidc/README.md) - Runtime OIDC behavior that integration tests should exercise
