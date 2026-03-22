# Quarkus Security OIDC Gotchas

Common OIDC pitfalls, symptoms, and fixes.

## Application type and flow selection

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Browser request gets `401` instead of login redirect | `quarkus.oidc.application-type=service` is used for a web app | Switch to `web-app` or `hybrid` |
| Bearer-token API starts redirecting to the login page | `web-app` mode is used for endpoints that should accept bearer tokens | Use `service` or `hybrid` for mixed apps |

## Provider and endpoint configuration

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Startup or first auth request fails against the provider | Discovery is disabled but required endpoint paths were not set | Configure `authorization-path`, `token-path`, `jwks-path`, and other needed paths |
| Logout works locally but not at the provider | Provider does not expose or accept the default end-session contract | Configure `quarkus.oidc.end-session-path` and provider-specific logout params |
| Redirect URI mismatch at login | Provider expects one fixed callback URL | Set `quarkus.oidc.authentication.redirect-path` and register it with the provider |

## Token validation and claims

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Valid-looking token is rejected with issuer errors | Discovered issuer URL differs from token `iss` | Align provider external URL or set `quarkus.oidc.token.issuer` explicitly |
| Token from another client is accepted unexpectedly | Audience is never checked | Set `quarkus.oidc.client-id` and `quarkus.oidc.token.audience` |
| Role-based checks always fail | Roles are not in `groups` or default Keycloak claims | Set `quarkus.oidc.roles.source` and `quarkus.oidc.roles.role-claim-path` |
| Web app can log in but role checks still fail | Roles are in the access token, not the ID token | Set `quarkus.oidc.roles.source=accesstoken` |

## UserInfo and introspection costs

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Every request becomes slower after enabling profile lookups | `UserInfo` or introspection adds remote calls on each request | Enable token cache or avoid `UserInfo` unless needed |
| Opaque tokens always fail | Introspection endpoint is missing or not authenticated correctly | Configure `quarkus.oidc.introspection-path` and any required introspection credentials |
| JWT validation unexpectedly depends on remote introspection | Default fallback to introspection is still enabled | Disable it with `quarkus.oidc.token.allow-jwt-introspection=false` if appropriate |

## Web-app sessions and cookies

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Users are asked to log in again too often | ID token expires and session refresh is disabled | Set `quarkus.oidc.token.refresh-expired=true` and tune session age |
| Users return after inactivity and are treated as brand new | Session cookie expired before refresh could happen | Increase `quarkus.oidc.authentication.session-age-extension` reasonably |
| Login works on one pod but not another | Session-cookie encryption key differs across nodes | Set a stable `quarkus.oidc.token-state-manager.encryption-secret` |
| Browser drops the session cookie | Stored tokens make the cookie too large | Use `id-refresh-tokens`, `split-tokens`, or DB/Redis token state |

## Web-app logout behavior

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Logout endpoint loops back into authentication | Logout or post-logout page is still protected | Make logout target pages public where required |
| Logout removes the user from the whole identity provider unexpectedly | RP-initiated logout was used but local app-only logout was intended | Use `OidcSession.logout()` for local logout |
| Post-logout redirect is rejected by the provider | Redirect URI is not registered as valid logout callback | Register `post_logout_redirect_uri` and keep it aligned with `post-logout-path` |

## See Also

- [`./configuration.md`](./configuration.md) - property fixes for these failures
- [`./patterns.md`](./patterns.md) - recommended implementation baselines
- [`../web-rest/gotchas.md`](../web-rest/gotchas.md) - HTTP and endpoint-level troubleshooting
