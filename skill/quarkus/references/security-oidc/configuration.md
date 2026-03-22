# Quarkus Security OIDC Configuration Reference

Use this file for high-value OIDC provider, application type, role mapping, token validation, and session settings.

## High-value properties

| Property | Typical value | Use when |
|----------|---------------|----------|
| `quarkus.oidc.auth-server-url` | provider realm/base URL | You need Quarkus to discover or resolve OIDC endpoints |
| `quarkus.oidc.discovery-enabled` | `true` | Provider metadata should be auto-discovered |
| `quarkus.oidc.application-type` | `service`, `web-app`, `hybrid` | You must choose bearer mode, browser login mode, or both |
| `quarkus.oidc.client-id` | provider client id | Audience checks or provider client authentication should be explicit |
| `quarkus.oidc.credentials.secret` | client secret | The provider requires confidential client authentication |
| `quarkus.oidc.authorization-path` | relative path | Discovery is disabled or provider metadata is incomplete |
| `quarkus.oidc.token-path` | relative path | Discovery is disabled or provider metadata is incomplete |
| `quarkus.oidc.jwks-path` | relative path | JWT verification key endpoint must be set manually |
| `quarkus.oidc.user-info-path` | relative path | `UserInfo` must be requested explicitly |
| `quarkus.oidc.introspection-path` | relative path | Opaque tokens or remote introspection must be supported |
| `quarkus.oidc.end-session-path` | relative path | RP-initiated logout endpoint must be set or overridden |
| `quarkus.oidc.roles.source` | `idtoken`, `accesstoken`, `userinfo` | Roles are not in the default token source |
| `quarkus.oidc.roles.role-claim-path` | `groups/roles` | Roles live in a nested or custom claim |
| `quarkus.oidc.authentication.user-info-required` | `true` | UserInfo is needed for claims or roles |
| `quarkus.oidc.token.issuer` | issuer URL | Issuer verification must be pinned or overridden |
| `quarkus.oidc.token.audience` | expected audience | Audience claim must be verified |
| `quarkus.oidc.token.allow-jwt-introspection` | `false` | JWT tokens should never fall back to introspection |
| `quarkus.oidc.token.allow-opaque-token-introspection` | `false` | Opaque token introspection must be disabled |
| `quarkus.oidc.token.require-jwt-introspection-only` | `true` | JWTs should always be validated remotely |
| `quarkus.oidc.token-cache.max-size` | positive number | Introspection or `UserInfo` calls should be cached |
| `quarkus.oidc.authentication.redirect-path` | `/callback` | Provider requires one stable redirect URI |
| `quarkus.oidc.authentication.restore-path-after-redirect` | `true` | Original requested path should be restored after callback |
| `quarkus.oidc.authentication.pkce-required` | `true` | Code flow should use PKCE |
| `quarkus.oidc.authentication.error-path` | `/error` | Provider redirect failures should go to a public error page |
| `quarkus.oidc.token.refresh-expired` | `true` | Web-app sessions should refresh instead of forcing re-login |
| `quarkus.oidc.authentication.session-age-extension` | duration | Session cookie should outlive the ID token |
| `quarkus.oidc.token.refresh-token-time-skew` | duration | Refresh should happen before token expiry |
| `quarkus.oidc.authentication.session-expired-page` | `/session-expired` | Users should see a public page before re-authentication |
| `quarkus.oidc.token-state-manager.strategy` | `keep-all-tokens`, `id-refresh-tokens`, `id-token` | Web-app session cookies should store fewer tokens |
| `quarkus.oidc.token-state-manager.split-tokens` | `true` | Cookie size might exceed browser limits |
| `quarkus.oidc.token-state-manager.encryption-secret` | 32+ char secret | Session cookie encryption must be stable across nodes |
| `quarkus.oidc.logout.path` | `/logout` | User-initiated logout should be enabled |
| `quarkus.oidc.logout.post-logout-path` | `/welcome.html` | User should return to a local page after logout |

## Minimal bearer-token service

```properties
quarkus.oidc.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc.client-id=inventory-service
quarkus.oidc.application-type=service
quarkus.oidc.token.audience=${quarkus.oidc.client-id}
```

## Minimal code-flow web app

```properties
quarkus.oidc.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc.client-id=frontend
quarkus.oidc.credentials.secret=${OIDC_CLIENT_SECRET}
quarkus.oidc.application-type=web-app
quarkus.oidc.authentication.pkce-required=true
```

## Manual endpoint configuration

```properties
quarkus.oidc.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc.discovery-enabled=false
quarkus.oidc.authorization-path=/protocol/openid-connect/auth
quarkus.oidc.token-path=/protocol/openid-connect/token
quarkus.oidc.jwks-path=/protocol/openid-connect/certs
quarkus.oidc.user-info-path=/protocol/openid-connect/userinfo
quarkus.oidc.introspection-path=/protocol/openid-connect/token/introspect
quarkus.oidc.end-session-path=/protocol/openid-connect/logout
```

## Role mapping from custom claims

```properties
quarkus.oidc.roles.source=accesstoken
quarkus.oidc.roles.role-claim-path=realm_access/roles
```

For nested custom claims, set `quarkus.oidc.roles.role-claim-path` explicitly.

## UserInfo-backed roles

```properties
quarkus.oidc.authentication.user-info-required=true
quarkus.oidc.roles.source=userinfo
quarkus.oidc.roles.role-claim-path=groups/roles
```

## Session refresh and cookie sizing

```properties
quarkus.oidc.token.refresh-expired=true
quarkus.oidc.authentication.session-age-extension=30M
quarkus.oidc.token.refresh-token-time-skew=1M
quarkus.oidc.token-state-manager.strategy=id-refresh-tokens
quarkus.oidc.token-state-manager.split-tokens=true
quarkus.oidc.token-state-manager.encryption-secret=${OIDC_STATE_SECRET}
```

## Logout flow

```properties
quarkus.oidc.logout.path=/logout
quarkus.oidc.logout.post-logout-path=/welcome.html
quarkus.oidc.authentication.cookie-path=/
```

## Diagnostics logging

```properties
quarkus.log.category."io.quarkus.oidc.runtime.OidcProvider".level=TRACE
quarkus.log.category."io.quarkus.oidc.runtime.OidcProvider".min-level=TRACE
quarkus.log.category."io.quarkus.oidc.runtime.OidcRecorder".level=TRACE
quarkus.log.category."io.quarkus.oidc.runtime.OidcRecorder".min-level=TRACE
```

## See Also

- [`./api.md`](./api.md) - runtime APIs tied to these properties
- [`./gotchas.md`](./gotchas.md) - common property mistakes
- [`../configuration/configuration.md`](../configuration/configuration.md) - profile and property organization guidance
