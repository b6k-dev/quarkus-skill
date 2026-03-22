# Quarkus Security WebAuthn API Reference

Use this file for runtime WebAuthn APIs and short, copy-ready examples.

## Extension entry points

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-security-webauthn</artifactId>
</dependency>
```

Typical persistence companions:

- `io.quarkus:quarkus-hibernate-orm-panache`
- a JDBC or reactive datasource driver

## Built-in endpoints

When enabled, Quarkus serves these endpoints:

- `GET /q/webauthn/register-options-challenge?username=...&displayName=...`
- `POST /q/webauthn/register`
- `GET /q/webauthn/login-options-challenge?username=...`
- `POST /q/webauthn/login`
- `GET /q/webauthn/logout`

Registration and login are separate two-step ceremonies. The challenge endpoints set a challenge cookie consumed by the matching `register` or `login` call.

## Browser helper library

```html
<script src="/q/webauthn/webauthn.js" type="text/javascript"></script>
<script>
  const webAuthn = new WebAuthn();
</script>
```

Default helpers:

- `webAuthn.register({ username, displayName })`
- `webAuthn.login({ username })`
- `webAuthn.registerClientSteps(...)`
- `webAuthn.loginClientSteps(...)`

`username` is optional for discoverable credentials/passkeys during login.

## Minimal credential record mapping

Persist the data Quarkus needs to validate later assertions:

```java
import java.util.UUID;

import io.quarkus.security.webauthn.WebAuthnCredentialRecord;
import io.quarkus.security.webauthn.WebAuthnCredentialRecord.RequiredPersistedData;

class StoredCredential {
    String credentialId;
    byte[] publicKey;
    long publicKeyAlgorithm;
    long counter;
    UUID aaguid;
    String username;

    WebAuthnCredentialRecord toRecord() {
        return WebAuthnCredentialRecord.fromRequiredPersistedData(
                new RequiredPersistedData(username, credentialId, aaguid, publicKey, publicKeyAlgorithm, counter));
    }
}
```

One username may map to multiple credential IDs when users register more than one authenticator.

## `WebAuthnUserProvider`

Expose a CDI bean so Quarkus can load, store, and update credentials:

```java
import java.util.List;
import java.util.Set;

import io.quarkus.security.webauthn.WebAuthnCredentialRecord;
import io.quarkus.security.webauthn.WebAuthnUserProvider;
import io.smallrye.common.annotation.Blocking;
import io.smallrye.mutiny.Uni;
import jakarta.enterprise.context.ApplicationScoped;

@Blocking
@ApplicationScoped
class MyWebAuthnUserProvider implements WebAuthnUserProvider {
    @Override
    public Uni<List<WebAuthnCredentialRecord>> findByUsername(String username) {
        return Uni.createFrom().item(List.of());
    }

    @Override
    public Uni<WebAuthnCredentialRecord> findByCredentialId(String credentialId) {
        return Uni.createFrom().nullItem();
    }

    @Override
    public Uni<Void> store(WebAuthnCredentialRecord credentialRecord) {
        return Uni.createFrom().voidItem();
    }

    @Override
    public Uni<Void> update(String credentialId, long counter) {
        return Uni.createFrom().voidItem();
    }

    @Override
    public Set<String> getRoles(String username) {
        return Set.of("user");
    }
}
```

Implement `store` only when using Quarkus' default registration endpoint. Implement `update` only when using Quarkus' default login endpoint.

## Custom login and registration endpoints

Use the browser helper's client-only steps plus `WebAuthnSecurity` when the built-in endpoints are too limited:

```java
import jakarta.inject.Inject;
import jakarta.ws.rs.BeanParam;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.core.Response;

import io.quarkus.security.webauthn.WebAuthnLoginResponse;
import io.quarkus.security.webauthn.WebAuthnRegisterResponse;
import io.quarkus.security.webauthn.WebAuthnSecurity;
import io.vertx.ext.web.RoutingContext;

@Path("/passkeys")
class PasskeyResource {
    @Inject
    WebAuthnSecurity webAuthnSecurity;

    @POST
    @Path("/login")
    Response login(@BeanParam WebAuthnLoginResponse response, RoutingContext ctx) {
        webAuthnSecurity.login(response, ctx).await().indefinitely();
        webAuthnSecurity.rememberUser("alice", ctx);
        return Response.ok().build();
    }

    @POST
    @Path("/register")
    Response register(@BeanParam WebAuthnRegisterResponse response, RoutingContext ctx) {
        webAuthnSecurity.register("alice", response, ctx).await().indefinitely();
        webAuthnSecurity.rememberUser("alice", ctx);
        return Response.ok().build();
    }
}
```

`WebAuthnSecurity` validates ceremonies, but your endpoint still owns input validation, persistence decisions, and session or token issuance.

## Blocking and virtual-thread usage

- Add `@Blocking` to `WebAuthnUserProvider` when it uses blocking persistence.
- `@RunOnVirtualThread` is also valid when the provider does blocking work on virtual threads.
- Blocking on `webAuthnSecurity.login(...).await().indefinitely()` is acceptable in those models because the main async boundary is usually persistence.
