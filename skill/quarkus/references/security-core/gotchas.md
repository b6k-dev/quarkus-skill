# Quarkus Security Core Gotchas

Common security pitfalls, symptoms, and fixes.

## Annotation vs HTTP permission rules

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `@PermitAll` does not open an endpoint | HTTP permission config already restricts the path | Relax or remove the matching `quarkus.http.auth.permission.*` rule |
| A new REST endpoint is accidentally public | No annotation and no default-deny setting | Set `quarkus.security.jaxrs.deny-unannotated-endpoints=true` or `quarkus.security.jaxrs.default-roles-allowed` |
| CDI method is callable without checks | Unannotated member lives in a bean that mixes secured and unsecured methods | Set `quarkus.security.deny-unannotated-members=true` |

## Path matching and policy selection

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| A broader rule does not apply | Longest matching path wins | Verify which permission path is more specific |
| `POST` is denied even though the path matches | A permission set matched the path but not the method | Add the method to the winning rule or define a fallback rule |
| Permission paths break after changing `quarkus.http.root-path` | Some paths use a leading slash and others do not | Standardize path style and test effective URLs |
| Wildcard rules behave unexpectedly | `*` at the end matches subpaths; `*` in the middle matches one segment | Keep patterns simple and test all secured paths |

## Authentication mechanism setup

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Basic or form auth never authenticates users | No matching `IdentityProvider` verifies username/password credentials | Add JPA, JDBC, LDAP, properties-file, or custom identity provider |
| Form auth works on one node but not another | Session cookie encryption key differs or clocks drift | Share the same `quarkus.http.auth.session.encryption-key` and keep clocks synchronized |
| SPA login flow keeps redirecting | Default form auth pages are still enabled | Clear `login-page`, `error-page`, and `landing-page` for SPA-style behavior |
| mTLS authenticates but roles are missing | Certificate subject is not mapped to roles | Configure `quarkus.http.auth.certificate-role-properties` or use an augmentor |

## Custom policies and permissions

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `@RequestScoped` data is unavailable in `HttpSecurityPolicy` | Authorization ran before Jakarta REST request setup | Apply the policy to JAX-RS endpoints with `@AuthorizationPolicy` or `applies-to=jaxrs` |
| `@PermissionsAllowed` fails in native mode with custom permission classes | Custom `Permission` type is not registered for reflection | Register the permission class for reflection when needed |
| Permission checks work in `SecurityIdentity` but not in a custom JAX-RS `SecurityContext` | Permissions are not exposed through Jakarta REST `SecurityContext` | Use `SecurityIdentity` for permission-aware authorization |
| Secured interface subresource locator is ignored | Default interface method security is not supported there | Put security annotations on the implementation method |

## Threading and request timing

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Annotation-based checks on IO-thread endpoints fail unexpectedly | Authentication is not available early enough for that execution model | Review proactive authentication and endpoint threading model |

## See Also

- [../web-rest/gotchas.md](../web-rest/gotchas.md) - REST path, filter, and threading issues that often surface with security
