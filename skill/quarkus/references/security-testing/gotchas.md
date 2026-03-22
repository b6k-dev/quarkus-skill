# Quarkus Security Testing Gotchas

Common security-testing pitfalls, symptoms, and fixes.

## `@TestSecurity` scope

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `@TestSecurity` appears to do nothing in an integration test | The annotation only works with `@QuarkusTest` | Use `@QuarkusTest` for shortcut identities, or send real credentials in `@QuarkusIntegrationTest` |
| Test passes authorization but misses token-dependent bugs | `@TestSecurity` bypasses real token parsing and provider interaction | Use bearer token tests when claims, token format, or OIDC behavior matter |

## Mixed authentication mechanisms

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Basic auth tests fail while `@TestSecurity` tests pass | Basic auth fallback was never enabled in `%test` | Set `%test.quarkus.http.auth.basic=true` |
| Wrong identity is used in a mixed bearer and `@TestSecurity` test | An `Authorization` header takes precedence over the shortcut test identity | Remove the header when you want the `@TestSecurity` identity, keep it when you want real bearer auth |
| Path-based auth tests behave unexpectedly | `authMechanism` was not selected or does not match the protected path | Set `@TestSecurity(authMechanism = "...")` and verify the path policy |

## OIDC and Dev Services

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Keycloak Dev Services does not start | `quarkus.oidc.auth-server-url` is already configured or the tenant is disabled | Remove the explicit server URL for Dev Services flows, or configure the intended external provider fully |
| Roles expected by `@RolesAllowed` are missing in tests | The token or imported realm does not expose the expected role claim | Verify realm mappers, scopes, and whether the endpoint expects access-token or ID-token roles |
| Dev Services startup is flaky in CI | Auth container startup exceeds the default timeout | Increase `quarkus.devservices.timeout` and keep realm imports small |
| Changes to users or clients are ignored | A `realm-path` import overrides default synthesized users and clients | Update the imported realm instead of only changing `quarkus.keycloak.devservices.users.*` |

## Dev UI and manual verification

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Dev UI login returns redirect or client errors | Client redirect URIs or flow settings do not match the chosen grant type | Fix the client configuration in the imported realm or provider admin console |
| Manual Dev UI test works but automated test fails | The automated test does not acquire or send the same token type | Align the test with the same flow, token, and endpoint assumptions used in Dev UI |

## See Also

- [./patterns.md](./patterns.md) - Preferred workflow choices before debugging failures
- [../security-oidc/README.md](../security-oidc/README.md) - OIDC runtime troubleshooting and provider setup
- [../security-jwt/README.md](../security-jwt/README.md) - JWT claim and token-shape issues that often surface in tests
