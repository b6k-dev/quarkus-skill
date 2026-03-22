# Quarkus Security OIDC Usage Patterns

Use these patterns for repeatable bearer-token and authorization-code-flow implementation workflows.

## Pattern: Protect a service API with bearer tokens

When to use:

- A frontend, CLI, or another service calls your API with `Authorization: Bearer ...`.

Configuration:

```properties
quarkus.oidc.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc.client-id=orders-api
quarkus.oidc.application-type=service
quarkus.oidc.token.audience=${quarkus.oidc.client-id}
```

Endpoint:

```java
@Path("/orders")
@Authenticated
class OrderResource {
    @GET
    @RolesAllowed("user")
    public String list() {
        return "orders";
    }
}
```

## Pattern: Protect a server-rendered web app with code flow

When to use:

- Quarkus serves HTML pages and should redirect users to log in.

Configuration:

```properties
quarkus.oidc.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc.client-id=frontend
quarkus.oidc.credentials.secret=${OIDC_CLIENT_SECRET}
quarkus.oidc.application-type=web-app
quarkus.oidc.authentication.pkce-required=true
```

Use `@Authenticated` on protected endpoints and inject `@IdToken JsonWebToken` when you need the logged-in user identity.

## Pattern: Combine browser login and bearer-token APIs

When to use:

- One Quarkus app serves browser pages and also exposes token-authenticated APIs.

Configuration:

```properties
quarkus.oidc.application-type=hybrid
```

Requests with a bearer token use bearer authentication first; browser requests without it trigger code flow.

## Pattern: Map roles from non-standard claims

When to use:

- The provider does not expose roles in `groups` or Keycloak-style claims.

Configuration:

```properties
quarkus.oidc.roles.source=accesstoken
quarkus.oidc.roles.role-claim-path=groups/roles
```

If claims are still not enough, add roles in a `SecurityIdentityAugmentor`.

## Pattern: Use UserInfo only when it adds value

When to use:

- ID/access tokens do not contain enough profile data or roles.

Configuration:

```properties
quarkus.oidc.authentication.user-info-required=true
quarkus.oidc.roles.source=userinfo
quarkus.oidc.token-cache.max-size=1000
quarkus.oidc.token-cache.time-to-live=5M
```

Cache `UserInfo` or introspection results when every request would otherwise cause remote calls.

## Pattern: Stabilize redirect URIs for strict providers

When to use:

- The provider requires one fixed callback URL.

Configuration:

```properties
quarkus.oidc.authentication.redirect-path=/callback
quarkus.oidc.authentication.restore-path-after-redirect=true
```

Register the callback URL with the provider and let Quarkus restore the original requested path afterward.

## Pattern: Make web-app sessions survive token expiry

When to use:

- Short-lived ID tokens should not force frequent full re-login.

Configuration:

```properties
quarkus.oidc.token.refresh-expired=true
quarkus.oidc.authentication.session-age-extension=30M
quarkus.oidc.token.refresh-token-time-skew=1M
```

Pair this with a session cookie lifetime that is longer than the ID token lifetime.

## Pattern: Keep session cookies smaller

When to use:

- JWT-heavy ID, access, and refresh tokens make cookies too large.

Configuration:

```properties
quarkus.oidc.token-state-manager.strategy=id-refresh-tokens
quarkus.oidc.token-state-manager.split-tokens=true
quarkus.oidc.token-state-manager.encryption-secret=${OIDC_STATE_SECRET}
```

Use DB or Redis token state management when cookie size or multi-node state becomes problematic.

## Pattern: Add logout to a web app intentionally

When to use:

- Users need either provider logout or application-local logout.

Configuration:

```properties
quarkus.oidc.logout.path=/logout
quarkus.oidc.logout.post-logout-path=/welcome.html
quarkus.oidc.authentication.cookie-path=/
```

Use `OidcSession.logout()` for local logout when you do not want single sign-out at the provider.

## See Also

- [`./api.md`](./api.md) - code snippets for these workflows
- [`./configuration.md`](./configuration.md) - key properties used by these patterns
