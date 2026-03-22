# Quarkus Security WebAuthn Usage Patterns

Use these patterns for repeatable passkey registration and login workflows.

## Pattern: Start with the default Quarkus ceremony

When to use:

- You only need username-based registration and login.
- Cookie-backed sessions are acceptable.

Setup:

```properties
quarkus.webauthn.enable-registration-endpoint=true
quarkus.webauthn.enable-login-endpoint=true
```

Also provide a `WebAuthnUserProvider` with `findByUsername`, `findByCredentialId`, `store`, and `update`.

## Pattern: Persist required credential fields exactly once

When to use:

- You are designing the server-side identity store.

Persist at least:

- username or user identifier
- credential ID
- public key bytes
- public key algorithm
- signature counter
- AAGUID

Store them durably and map them back to `WebAuthnCredentialRecord` on reads.

## Pattern: Support multiple passkeys per account safely

When to use:

- Users may enroll more than one authenticator.

Approach:

- Make credential ID unique.
- Model username-to-credentials as one-to-many.
- Only allow adding a new credential to an existing account when the current session is already authenticated as that same user.

Do not treat an unauthenticated registration for an existing username as account linking.

## Pattern: Keep built-in challenge endpoints, customize the callback endpoints

When to use:

- Registration needs extra profile data or business validation.
- Login must issue JWTs or another custom session mechanism.

Flow:

```javascript
webAuthn.registerClientSteps({ username, displayName }).then(body => {
  // submit body fields with your own form or fetch call
});
```

On the server, call `WebAuthnSecurity.register(...)` or `WebAuthnSecurity.login(...)`, then persist or update credentials and issue your own session/token.

## Pattern: Use blocking persistence explicitly

When to use:

- The credential store is JPA, Panache, JDBC, or another blocking API.

Example:

```java
import io.smallrye.common.annotation.Blocking;

@Blocking
@ApplicationScoped
class MyWebAuthnUserProvider implements WebAuthnUserProvider {
}
```

This keeps WebAuthn endpoint work off the event loop.

## Pattern: Test ceremonies without physical hardware

When to use:

- You want repeatable integration tests for registration and login.

Test dependency:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-test-security-webauthn</artifactId>
    <scope>test</scope>
</dependency>
```

Use `WebAuthnHardware` to emulate an authenticator and `WebAuthnEndpointHelper` to drive the challenge/register/login endpoints.
