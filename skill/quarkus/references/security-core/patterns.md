# Quarkus Security Core Usage Patterns

Use these patterns for repeatable authentication and authorization workflows.

## Pattern: Start with annotation-based authorization

When to use:

- You are securing REST endpoints or CDI beans and do not need path-wide rules.

Example:

```java
import io.quarkus.security.Authenticated;
import jakarta.annotation.security.RolesAllowed;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/reports")
class ReportResource {
    @GET
    @Authenticated
    String me() {
        return "ok";
    }

    @GET
    @Path("/admin")
    @RolesAllowed("admin")
    String admin() {
        return "ok";
    }
}
```

Prefer this over path config when authorization follows endpoint semantics.

## Pattern: Deny by default for REST endpoints

When to use:

- You want fail-closed behavior for new endpoints.

Configuration:

```properties
quarkus.security.jaxrs.deny-unannotated-endpoints=true
```

Then add `@PermitAll`, `@Authenticated`, or `@RolesAllowed` explicitly.

## Pattern: Use HTTP permission config for path-wide rules

When to use:

- Static assets, public paths, admin prefixes, or method-based rules must be secured consistently.

Configuration:

```properties
quarkus.http.auth.policy.admin-policy.roles-allowed=admin

quarkus.http.auth.permission.public.paths=/public/*
quarkus.http.auth.permission.public.policy=permit
quarkus.http.auth.permission.public.methods=GET,HEAD

quarkus.http.auth.permission.admin.paths=/admin/*
quarkus.http.auth.permission.admin.policy=admin-policy
```

## Pattern: Add deployment-specific roles with `SecurityIdentityAugmentor`

When to use:

- Upstream identities are valid but local role mapping needs extra logic.

Example:

```java
@ApplicationScoped
class RolesAugmentor implements SecurityIdentityAugmentor {
    @Override
    public Uni<SecurityIdentity> augment(SecurityIdentity identity, AuthenticationRequestContext context) {
        if (!identity.hasRole("admin")) {
            return Uni.createFrom().item(identity);
        }
        return Uni.createFrom().item(QuarkusSecurityIdentity.builder(identity)
                .addRole("support")
                .build());
    }
}
```

## Pattern: Grant permissions through HTTP role policies

When to use:

- Endpoints use `@PermissionsAllowed` but identities naturally start as roles.

Configuration:

```properties
quarkus.http.auth.policy.editor-policy.permissions.editor=write,approve
quarkus.http.auth.permission.docs.paths=/docs/*
quarkus.http.auth.permission.docs.policy=editor-policy
```

This keeps permission mapping in config and endpoint checks in code.

## Pattern: Restrict a path to one auth mechanism

When to use:

- Different areas of the same service should use different credential types.

Configuration:

```properties
quarkus.http.auth.permission.basic-only.paths=/admin/*
quarkus.http.auth.permission.basic-only.policy=authenticated
quarkus.http.auth.permission.basic-only.auth-mechanism=basic
quarkus.http.auth.permission.basic-only.applies-to=jaxrs
```

## Pattern: Use form auth only with a real identity provider

When to use:

- You need browser login flow without external SSO.

Checklist:

- Enable `quarkus.http.auth.form.enabled=true`.
- Set `quarkus.http.auth.session.encryption-key`.
- Back credentials with JPA, JDBC, LDAP, or another provider.
- Use embedded users only for tests and examples.

## Pattern: Add request-aware authorization with a named policy

When to use:

- Access depends on headers, path content, or request context beyond plain roles.

Example:

```java
@ApplicationScoped
class ProjectPolicy implements HttpSecurityPolicy {
    @Override
    public Uni<CheckResult> checkPermission(RoutingContext ctx, Uni<SecurityIdentity> identity,
            AuthorizationRequestContext requestContext) {
        if ("internal".equals(ctx.request().getHeader("X-Network"))) {
            return CheckResult.permit();
        }
        return CheckResult.deny();
    }

    @Override
    public String name() {
        return "project-policy";
    }
}
```

Bind it with `@AuthorizationPolicy(name = "project-policy")` or HTTP permission config.
