# Quarkus Security Core API Reference

Use this file for core security APIs and annotations with short, copy-ready examples.

## Core types

- `SecurityIdentity` - current principal, roles, credentials, attributes, permissions
- `IdentityProvider<T>` - verifies a request and builds a `SecurityIdentity`
- `SecurityIdentityAugmentor` - adds roles, permissions, or attributes after authentication
- `HttpAuthenticationMechanism` - extracts credentials from HTTP requests
- `HttpSecurityPolicy` - performs request-time authorization checks

## Inject the current identity

```java
import io.quarkus.security.identity.SecurityIdentity;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/me")
class MeResource {
    @Inject
    SecurityIdentity identity;

    @GET
    String me() {
        return identity.isAnonymous() ? "anonymous" : identity.getPrincipal().getName();
    }
}
```

Use `SecurityIdentity` to read principal name, roles, credentials, attributes, and permissions.

## Standard authorization annotations

```java
import io.quarkus.security.Authenticated;
import jakarta.annotation.security.DenyAll;
import jakarta.annotation.security.PermitAll;
import jakarta.annotation.security.RolesAllowed;

@RolesAllowed("admin")
class AdminService {
    @PermitAll
    String ping() {
        return "ok";
    }

    @Authenticated
    String profile() {
        return "user";
    }

    @DenyAll
    String disabled() {
        return "no";
    }
}
```

`@Authenticated` is equivalent to `@RolesAllowed("**")`.

## Read the Jakarta REST security context

```java
import java.security.Principal;

import jakarta.annotation.security.RolesAllowed;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.core.Context;
import jakarta.ws.rs.core.SecurityContext;

@Path("/subject")
class SubjectResource {
    @GET
    @RolesAllowed("user")
    String subject(@Context SecurityContext securityContext) {
        Principal principal = securityContext.getUserPrincipal();
        return principal.getName();
    }
}
```

## Require permissions instead of roles

```java
import io.quarkus.security.PermissionsAllowed;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;

@Path("/documents")
class DocumentResource {
    @POST
    @PermissionsAllowed(value = { "write", "approve" }, inclusive = true)
    void publish() {
    }
}
```

Use `@PermissionsAllowed` when role checks are too coarse.

## Apply a named HTTP security policy

```java
import io.quarkus.vertx.http.security.AuthorizationPolicy;
import jakarta.annotation.security.DenyAll;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/reports")
@DenyAll
class ReportResource {
    @GET
    @AuthorizationPolicy(name = "internal-only")
    String list() {
        return "ok";
    }
}
```

## Custom `HttpSecurityPolicy`

```java
import jakarta.enterprise.context.ApplicationScoped;

import io.quarkus.security.identity.SecurityIdentity;
import io.quarkus.vertx.http.runtime.security.HttpSecurityPolicy;
import io.smallrye.mutiny.Uni;
import io.vertx.ext.web.RoutingContext;

@ApplicationScoped
class InternalOnlyPolicy implements HttpSecurityPolicy {
    @Override
    public Uni<CheckResult> checkPermission(RoutingContext ctx, Uni<SecurityIdentity> identity,
            AuthorizationRequestContext requestContext) {
        if ("true".equals(ctx.request().getHeader("X-Internal"))) {
            return CheckResult.permit();
        }
        return CheckResult.deny();
    }

    @Override
    public String name() {
        return "internal-only";
    }
}
```

Bind it with config or `@AuthorizationPolicy`.

## Augment a `SecurityIdentity`

```java
import io.quarkus.security.identity.AuthenticationRequestContext;
import io.quarkus.security.identity.SecurityIdentity;
import io.quarkus.security.identity.SecurityIdentityAugmentor;
import io.quarkus.security.runtime.QuarkusSecurityIdentity;
import io.smallrye.mutiny.Uni;
import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
class RolesAugmentor implements SecurityIdentityAugmentor {
    @Override
    public Uni<SecurityIdentity> augment(SecurityIdentity identity, AuthenticationRequestContext context) {
        if (identity.isAnonymous() || !identity.hasRole("admin")) {
            return Uni.createFrom().item(identity);
        }
        return Uni.createFrom().item(QuarkusSecurityIdentity.builder(identity)
                .addRole("support")
                .build());
    }
}
```

Use an augmentor to add roles, permissions, or attributes after authentication.

## Implement an `IdentityProvider`

```java
import io.quarkus.security.identity.AuthenticationRequestContext;
import io.quarkus.security.identity.IdentityProvider;
import io.quarkus.security.identity.SecurityIdentity;
import io.quarkus.security.identity.request.UsernamePasswordAuthenticationRequest;
import io.smallrye.mutiny.Uni;
import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
class MyIdentityProvider implements IdentityProvider<UsernamePasswordAuthenticationRequest> {
    @Override
    public Class<UsernamePasswordAuthenticationRequest> getRequestType() {
        return UsernamePasswordAuthenticationRequest.class;
    }

    @Override
    public Uni<SecurityIdentity> authenticate(UsernamePasswordAuthenticationRequest request,
            AuthenticationRequestContext context) {
        return Uni.createFrom().nullItem();
    }
}
```

Use an identity provider when a mechanism supplies credentials but no matching provider verifies them.

## Implement an `HttpAuthenticationMechanism`

```java
import io.quarkus.security.identity.IdentityProviderManager;
import io.quarkus.vertx.http.runtime.security.HttpAuthenticationMechanism;
import io.smallrye.mutiny.Uni;
import io.vertx.ext.web.RoutingContext;

class HeaderAuthenticationMechanism implements HttpAuthenticationMechanism {
    @Override
    public Uni<SecurityIdentity> authenticate(RoutingContext context,
            IdentityProviderManager identityProviderManager) {
        return Uni.createFrom().nullItem();
    }
}
```

Use a custom mechanism only when built-in Basic, Form, mTLS, OIDC, JWT, or other extensions do not fit.

## Programmatic HTTP security setup

```java
import io.quarkus.vertx.http.security.HttpSecurity;
import jakarta.enterprise.event.Observes;

class HttpSecurityConfiguration {
    void configure(@Observes HttpSecurity httpSecurity) {
        httpSecurity
                .get("/public/*").permit()
                .path("/admin/*").roles("admin")
                .path("/api/*").authenticated();
    }
}
```
