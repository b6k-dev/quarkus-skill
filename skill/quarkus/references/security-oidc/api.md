# Quarkus Security OIDC API Reference

Use this file for runtime OIDC APIs with short, copy-ready examples.

## Extension entry points

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-oidc</artifactId>
</dependency>
```

Optional stateful session storage extensions:

- `io.quarkus:quarkus-oidc-db-token-state-manager`
- `io.quarkus:quarkus-oidc-redis-token-state-manager`

## Protect a bearer-token service

```java
import io.quarkus.security.Authenticated;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/api/user")
@Authenticated
class UserResource {
    @GET
    String me() {
        return "ok";
    }
}
```

Use `@Authenticated` for any authenticated principal and `@RolesAllowed` for role checks.

## Role-based access

```java
import jakarta.annotation.security.RolesAllowed;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/api/admin")
class AdminResource {
    @GET
    @RolesAllowed("admin")
    String admin() {
        return "granted";
    }
}
```

## Access bearer token claims

```java
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.jwt.JsonWebToken;

@Path("/api/claims")
class ClaimsResource {
    @Inject
    JsonWebToken jwt;

    @GET
    String subject() {
        return jwt.getSubject();
    }
}
```

## Use scopes as permissions

```java
import io.quarkus.security.PermissionsAllowed;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/orders")
class OrderResource {
    @GET
    @PermissionsAllowed("orders_read")
    String list() {
        return "orders";
    }
}
```

## Access the current identity

```java
import io.quarkus.security.identity.SecurityIdentity;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/api/me")
class IdentityResource {
    @Inject
    SecurityIdentity identity;

    @GET
    String me() {
        return identity.getPrincipal().getName();
    }
}
```

## Access code-flow ID token

```java
import io.quarkus.oidc.IdToken;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.jwt.JsonWebToken;

@Path("/web")
class WebResource {
    @Inject
    @IdToken
    JsonWebToken idToken;

    @GET
    String user() {
        return idToken.getName();
    }
}
```

## Access code-flow access token

```java
import io.quarkus.oidc.AccessTokenCredential;
import jakarta.inject.Inject;

class DownstreamClient {
    @Inject
    AccessTokenCredential accessToken;

    String bearer() {
        return accessToken.getToken();
    }
}
```

Inject `JsonWebToken` instead when the access token is JWT and you need claims.

## Access UserInfo and provider metadata

```java
import io.quarkus.oidc.OidcConfigurationMetadata;
import io.quarkus.oidc.UserInfo;
import jakarta.inject.Inject;

class OidcDetails {
    @Inject
    UserInfo userInfo;

    @Inject
    OidcConfigurationMetadata metadata;
}
```

`UserInfo` is available when Quarkus fetches it explicitly or auto-enables it.

## Local logout for web apps

```java
import io.quarkus.oidc.OidcSession;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/session")
class SessionResource {
    @Inject
    OidcSession oidcSession;

    @GET
    @Path("logout")
    String logout() {
        oidcSession.logout().await().indefinitely();
        return "logged out";
    }
}
```

## Identity augmentation

Use `SecurityIdentityAugmentor` when provider claims are not enough to build application roles or permissions.

## Flow-specific OIDC filters

Use these when requests to the OIDC provider need custom headers, body changes, or logging:

- `OidcRequestFilter`
- `OidcResponseFilter`
- `OidcRedirectFilter`
- `@AuthorizationCodeFlow`
- `@BearerTokenAuthentication`

## See Also

- [`./configuration.md`](./configuration.md) - properties that activate these patterns
- [`./patterns.md`](./patterns.md) - end-to-end implementation workflows
- [`../web-rest/api.md`](../web-rest/api.md) - endpoint annotations and response APIs
