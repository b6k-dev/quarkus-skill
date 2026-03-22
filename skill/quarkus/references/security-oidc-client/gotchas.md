# Quarkus Security OIDC Client Gotchas

Common outbound token acquisition and propagation pitfalls, symptoms, and fixes.

## Propagation vs acquisition mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Downstream receives no `Authorization` header | The REST client has no `@OidcClientFilter`, `@AccessToken`, or manual header wiring | Put token behavior on the outbound client boundary explicitly |
| A service forwards the caller token through multiple internal hops | Raw bearer propagation was used where token exchange was safer | Exchange the token before forwarding it to downstream services |
| One REST client sometimes sends a service token and sometimes a propagated user token | Acquisition and propagation filters were mixed on the same client | Split into separate client interfaces with one auth strategy each |

## Refresh and expiry surprises

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| First call fails because the token is already stale | Early token acquisition happened long before first use | Set `quarkus.oidc-client.early-tokens-acquisition=false` |
| Sporadic downstream `401` errors happen near token expiry | Refresh waits until the token is effectively dead | Add `quarkus.oidc-client.refresh-token-time-skew` and consider filter `refresh-on-unauthorized` |
| Requests pause while refreshing under load | Refresh happens on the request path only | Consider `quarkus.oidc-client.refresh-interval` for filter-managed tokens |

## Named-client confusion

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| The wrong credentials or scopes reach a downstream | A default OIDC client is being used accidentally | Use named clients explicitly and disable the default with `client-enabled=false` when that is safer |
| Annotation and config seem to disagree about client choice | Client name is set in both places | Treat the annotation value as the local override and keep one obvious source of truth |

## Grant and boundary mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Service-to-service calls act as a hard-coded user | `password` grant was used for a workload identity problem | Prefer `client` grant for workload-to-workload communication |
| Token exchange is enabled but the provider rejects the request | Required exchange grant options such as audience or requested token use are missing | Configure the right `quarkus.oidc-client.grant-options.exchange.*` or `grant-options.jwt.*` values for that provider |
| Discovery fails in restricted environments | The provider metadata endpoint is unavailable | Disable discovery and configure `token-path` directly |
