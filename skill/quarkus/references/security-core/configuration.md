# Quarkus Security Core Configuration Reference

Use this file for high-value authentication, authorization, and mechanism-selection settings.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.security.jaxrs.deny-unannotated-endpoints` | `false` | Unannotated Jakarta REST endpoints should be denied by default |
| `quarkus.security.jaxrs.default-roles-allowed` | - | Unannotated Jakarta REST endpoints should require specific roles |
| `quarkus.security.deny-unannotated-members` | `false` | Unannotated CDI methods in partially secured beans should be denied |
| `quarkus.http.auth.permission."name".paths` | required | You need path-based HTTP authorization rules |
| `quarkus.http.auth.permission."name".policy` | required | A permission set should reference `permit`, `deny`, `authenticated`, or a named role/custom policy |
| `quarkus.http.auth.permission."name".methods` | all methods | Access should differ by HTTP method |
| `quarkus.http.auth.permission."name".auth-mechanism` | all matches | A path should use only selected mechanisms such as `basic`, `form`, or `bearer` |
| `quarkus.http.auth.permission."name".applies-to` | `all` | Permission checks should be delayed to Jakarta REST endpoint resolution |
| `quarkus.http.auth.permission."name".shared` | `false` | A policy should always run together with the winning path policy |
| `quarkus.http.auth.policy."name".roles-allowed` | `**` | A named role policy should require specific roles |
| `quarkus.http.auth.policy."name".permissions."role"` | - | Roles should grant string or custom permissions for `@PermissionsAllowed` |
| `quarkus.http.auth.policy."name".permission-class` | `io.quarkus.security.StringPermission` | Role-mapped permissions should use a custom `Permission` type |
| `quarkus.http.auth.roles-mapping."role"` | - | External roles should be remapped to deployment-specific roles |
| `quarkus.http.auth.inclusive` | `false` | Multiple authentication mechanisms must all participate |
| `quarkus.http.auth.inclusive-mode` | `strict` | Inclusive auth should be `strict` or `lax` |
| `quarkus.http.auth.form.enabled` | `false` | Built-in form authentication should be enabled |
| `quarkus.http.auth.session.encryption-key` | - | Form auth sessions must work safely, especially across cluster members |
| `quarkus.http.auth.form.login-page` | `/login.html` | Form login redirects should use a custom page or be disabled |
| `quarkus.http.auth.form.error-page` | `/error.html` | Form auth failure redirect should change or be disabled |
| `quarkus.http.auth.form.landing-page` | `/index.html` | Post-login redirect target should change or be disabled |
| `quarkus.http.ssl.client-auth` | `none` | mTLS should request or require client certificates |
| `quarkus.http.auth.certificate-role-properties` | - | Client certificate attributes should map to roles |

## Secure all unannotated REST endpoints

```properties
quarkus.security.jaxrs.deny-unannotated-endpoints=true
```

Alternative:

```properties
quarkus.security.jaxrs.default-roles-allowed=**
```

## Path-based RBAC

```properties
quarkus.http.auth.policy.admin-policy.roles-allowed=admin

quarkus.http.auth.permission.admin.paths=/admin/*
quarkus.http.auth.permission.admin.policy=admin-policy
```

## Public GET, deny everything else on same path

```properties
quarkus.http.auth.permission.public.paths=/docs/*
quarkus.http.auth.permission.public.policy=permit
quarkus.http.auth.permission.public.methods=GET,HEAD
```

`POST /docs/x` is denied if no other matching method rule applies.

## Select auth mechanism per path

```properties
quarkus.http.auth.permission.admin.paths=/admin/*
quarkus.http.auth.permission.admin.policy=authenticated
quarkus.http.auth.permission.admin.auth-mechanism=basic
quarkus.http.auth.permission.admin.applies-to=jaxrs
```

## Form authentication

```properties
quarkus.http.auth.form.enabled=true
quarkus.http.auth.session.encryption-key=change-me-to-16-plus-chars
quarkus.http.auth.form.login-page=/login.html
quarkus.http.auth.form.landing-page=/app
quarkus.http.auth.form.error-page=/login-error.html
```

SPA-style setup without redirects:

```properties
quarkus.http.auth.form.login-page=
quarkus.http.auth.form.error-page=
quarkus.http.auth.form.landing-page=
quarkus.http.auth.form.http-only-cookie=false
```

## Mutual TLS

```properties
quarkus.http.ssl.client-auth=required
quarkus.http.auth.permission.cert.paths=/*
quarkus.http.auth.permission.cert.policy=authenticated
quarkus.http.auth.certificate-role-properties=cert-role-mappings.properties
```

## See Also

- [../configuration/README.md](../configuration/README.md) - General config source precedence, profiles, and overrides
