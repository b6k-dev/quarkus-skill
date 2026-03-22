# Quarkus Security Testing Usage Patterns

Use these patterns for repeatable secured-endpoint and auth integration-test workflows.

## Pattern: Use `@TestSecurity` for fast authorization tests

When to use:

- You are in `@QuarkusTest` and want to verify role or permission checks without booting a real identity provider.

Example:

```java
import io.quarkus.test.junit.QuarkusTest;
import io.quarkus.test.security.TestSecurity;
import io.restassured.RestAssured;
import org.junit.jupiter.api.Test;

@QuarkusTest
class AdminEndpointTest {
    @Test
    @TestSecurity(user = "alice", roles = "admin")
    void allowsAdmin() {
        RestAssured.get("/admin").then().statusCode(200);
    }

    @Test
    @TestSecurity(user = "bob", roles = "user")
    void rejectsNonAdmin() {
        RestAssured.get("/admin").then().statusCode(403);
    }
}
```

Keep these tests close to the secured resource or service and use them as the fastest feedback loop.

## Pattern: Switch to bearer tokens when claims matter

When to use:

- Endpoint code reads token claims, injects `JsonWebToken`, or depends on the real bearer mechanism.

Example:

```java
String token = tokenFor("alice");

RestAssured.given()
        .auth().oauth2(token)
        .get("/api/userinfo")
        .then()
        .statusCode(200);
```

Prefer real tokens over `@TestSecurity` when the test should prove token parsing and claim mapping, not just authorization.

## Pattern: Keep one JVM test layer and one integration-test layer

Use two complementary test layers:

- `@QuarkusTest` plus `@TestSecurity` for fast authorization and identity-driven code paths.
- HTTP integration tests with bearer tokens or a real OIDC login flow for end-to-end auth behavior.

Do not force all security checks into only one style. They answer different questions.

## Pattern: Use Keycloak Dev Services for realistic OIDC tests

When to use:

- The application is secured with `quarkus-oidc` and the test should talk to a real provider.

Example:

```properties
quarkus.keycloak.devservices.realm-path=quarkus-realm.json
quarkus.oidc.client-id=quarkus-app
quarkus.oidc.credentials.secret=secret
```

This is usually the best default for OIDC service or web-app integration tests in local development and CI.

## Pattern: Import a realm instead of rebuilding test users in code

When to use:

- Roles, clients, scopes, redirect URIs, or mappers are part of the behavior under test.

Example:

```properties
quarkus.keycloak.devservices.realm-path=quarkus-realm.json
quarkus.keycloak.devservices.realm-name=quarkus
```

Prefer a checked-in realm file when authorization behavior depends on provider-side configuration.

## Pattern: Mix `@TestSecurity` and real tokens intentionally

When to use:

- Some test methods need shortcut identities while others must send actual credentials.

Example:

```java
@Test
@TestSecurity(user = "Bob")
void choosesBearerTokenWhenPresent() {
    RestAssured.given()
            .auth().oauth2(tokenFor("Alice"))
            .get("/hello")
            .then()
            .statusCode(200);

    RestAssured.get("/hello")
            .then()
            .statusCode(200);
}
```

If an `Authorization` header is present, the real mechanism wins. Without it, the test identity is used.

## Pattern: Use `%test` profiles to simplify auth only in tests

When to use:

- Production uses OAuth2 or OIDC but tests need simpler local auth for selected suites.

Example:

```properties
quarkus.oauth2.enabled=true
%test.quarkus.oauth2.enabled=false
%test.quarkus.security.users.embedded.enabled=true
%test.quarkus.security.users.embedded.users.scott=reader
%test.quarkus.security.users.embedded.roles.scott=READER
```

This keeps production security intact while avoiding external dependencies in targeted JVM tests.

## Pattern: Run auth integration tests continuously

When to use:

- You want continuous testing or normal `test` runs to boot the auth provider automatically.

Dev Services for Keycloak starts in test mode when OIDC is present and `quarkus.oidc.auth-server-url` is not set. That makes it a good fit for CI and continuous testing without a separately managed Keycloak.

## Pattern: Use a custom test resource when Dev Services are not enough

When to use:

- The provider must be created dynamically, pre-seeded via API, or shared with nonstandard wiring.

Use `QuarkusTestResourceLifecycleManager` to return dynamic OIDC properties and keep provider startup outside the test class.

## See Also

- [./api.md](./api.md) - Copy-ready annotations and test resource snippets
- [./configuration.md](./configuration.md) - Properties for Keycloak, OIDC, and mixed auth flows
- [../security-core/README.md](../security-core/README.md) - Identity, role, and permission concepts behind these tests
