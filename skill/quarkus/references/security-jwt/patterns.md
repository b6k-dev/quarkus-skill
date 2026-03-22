# Quarkus Security JWT Usage Patterns

Use these patterns for repeatable bearer-token verification and token-building workflows.

## Pattern: Secure a service with locally verified bearer tokens

When to use:

- The service receives JWT bearer tokens and only needs local signature and claim verification.
- Browser login redirects and token acquisition are out of scope.

Command:

```bash
quarkus create app com.acme:orders --extension='rest,smallrye-jwt' --no-code
```

Configuration:

```properties
mp.jwt.verify.publickey.location=publicKey.pem
mp.jwt.verify.issuer=https://example.com/issuer
```

Endpoint:

```java
@Path("/orders")
class OrderResource {
    @Inject
    JsonWebToken jwt;

    @GET
    @RolesAllowed("User")
    String list() {
        return jwt.getName();
    }
}
```

## Pattern: Inject claims directly in request scope

When to use:

- Endpoint code only needs a small number of claims as typed fields.

Example:

```java
@Path("/profile")
@RequestScoped
class ProfileResource {
    @Inject
    @Claim(standard = Claims.birthdate)
    String birthdate;

    @GET
    String get() {
        return birthdate;
    }
}
```

Use `JsonWebToken` instead when request scope is inconvenient or many claims are needed.

## Pattern: Parse a token that is outside standard bearer auth

When to use:

- The token is in a payload, cookie, message, or custom integration boundary.

Example:

```java
@ApplicationScoped
class CookieTokenService {
    @Inject
    JWTParser parser;

    String subject(String token) throws Exception {
        return parser.parse(token).getName();
    }
}
```

If the token uses a per-request secret or key, call `parser.verify(...)` or `parser.decrypt(...)` with that key.

## Pattern: Issue signed service tokens

When to use:

- Tests, internal tools, or token-issuing services need to mint JWTs directly.

Configuration:

```properties
smallrye.jwt.sign.key.location=privateKey.pem
smallrye.jwt.new-token.issuer=https://example.com/issuer
smallrye.jwt.new-token.lifespan=900
```

Example:

```java
String token = Jwt.upn("jdoe@quarkus.io")
        .groups(Set.of("User"))
        .claim("tenant", "acme")
        .sign();
```

## Pattern: Sign first, then encrypt sensitive claims

When to use:

- Claims are sensitive and the receiver must validate both integrity and confidentiality.

Configuration:

```properties
smallrye.jwt.sign.key.location=privateKey.pem
smallrye.jwt.encrypt.key.location=service-public-key.pem
```

Example:

```java
String token = Jwt.claim("ssn", "123-45-6789")
        .issuer("https://example.com/issuer")
        .innerSign()
        .encrypt();
```

Prefer nested signed+encrypted tokens over direct public-key encryption when the receiver must know who produced the token.

## Pattern: Stay on JWT, move to OIDC only when needed

When to use:

- You are deciding between local JWT verification and a full identity-provider integration.

Choose SmallRye JWT when:

- The service validates self-contained JWTs with configured keys and claims.
- Service-to-service bearer auth is enough.

Move to OIDC when:

- The app needs authorization code flow, provider discovery, opaque token introspection, logout, or richer IdP-managed lifecycle behavior.
