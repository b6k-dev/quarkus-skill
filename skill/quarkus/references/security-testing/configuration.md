# Quarkus Security Testing Configuration Reference

Use this file for high-value test security, OIDC, and Dev Services settings.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `%test.quarkus.http.auth.basic=true` | `false` | Tests mix `@TestSecurity` with fallback Basic authentication |
| `quarkus.oidc.auth-server-url` | - | Tests should use an already running OIDC provider instead of Dev Services |
| `quarkus.oidc.client-id` | - | OIDC tests need a non-default client ID |
| `quarkus.oidc.credentials.secret` | - | Confidential client tests need a client secret |
| `quarkus.oidc.devui.grant.type` | `code` | Dev mode token acquisition should use `code`, `implicit`, `password`, or `client` |
| `quarkus.oidc.devservices.enabled` | auto | You want the built-in OIDC dev service instead of a real external provider |
| `quarkus.oidc.devservices.roles.alice` | built-in roles | Built-in OIDC dev service users need custom roles |
| `quarkus.keycloak.devservices.enabled` | `true` | Keycloak Dev Services should be disabled or forced off in tests |
| `quarkus.keycloak.devservices.realm-path` | - | Tests should import one or more realm files instead of using the default realm |
| `quarkus.keycloak.devservices.realm-name` | `quarkus` | The test realm name should be explicit without parsing the realm file |
| `quarkus.keycloak.devservices.users.*` | built-in users | Test users and passwords should be customized |
| `quarkus.keycloak.devservices.roles.*` | built-in roles | Test user roles should match application authorization rules |
| `quarkus.keycloak.devservices.shared` | `true` in dev | Dev mode should not reuse an existing shared Keycloak container |
| `quarkus.keycloak.devservices.port` | random | Tests need a stable Keycloak port |
| `quarkus.devservices.timeout` | `60S` | Slow CI or local machines need more startup time for auth containers |

## Profile-based fallback auth for tests

```properties
quarkus.oauth2.enabled=true
%test.quarkus.oauth2.enabled=false
%test.quarkus.security.users.embedded.enabled=true
%test.quarkus.security.users.embedded.plain-text=true
%test.quarkus.security.users.embedded.users.scott=reader
%test.quarkus.security.users.embedded.roles.scott=READER
```

Use this to keep production auth enabled while giving tests a simpler local identity source.

## Enable Basic auth for mixed tests

```properties
%test.quarkus.http.auth.basic=true
```

Required when some tests rely on `@TestSecurity` and others send real Basic auth requests.

## Import a test realm into Keycloak Dev Services

```properties
quarkus.keycloak.devservices.realm-path=quarkus-realm.json
quarkus.keycloak.devservices.realm-name=quarkus
quarkus.oidc.client-id=quarkus-app
quarkus.oidc.credentials.secret=secret
```

If `realm-path` is set, Dev Services uses that realm instead of synthesizing default users and clients.

## Customize Dev Services test users

```properties
%test.quarkus.keycloak.devservices.users.duke=dukePassword
%test.quarkus.keycloak.devservices.roles.duke=reader
%test.quarkus.keycloak.devservices.users.john=johnPassword
%test.quarkus.keycloak.devservices.roles.john=reader,writer
```

## Enable the lightweight OIDC dev service

```properties
quarkus.oidc.devservices.enabled=true
quarkus.oidc.devservices.roles.alice=admin,user
quarkus.oidc.devservices.roles.bob=user
```

Use this for non-Keycloak providers or lighter-weight OIDC testing.

## Dev UI token acquisition

```properties
quarkus.oidc.client-id=quarkus-app
quarkus.oidc.credentials.secret=secret
quarkus.oidc.devui.grant.type=password
```

Useful in dev mode when you want the Dev UI to fetch tokens and drive manual secured-endpoint tests.

## See Also

- [./patterns.md](./patterns.md) - How these properties fit typical secured test workflows
- [../configuration/README.md](../configuration/README.md) - Profiles and property scoping across dev, test, and prod
- [../security-oidc/README.md](../security-oidc/README.md) - OIDC runtime properties beyond test-specific setup
