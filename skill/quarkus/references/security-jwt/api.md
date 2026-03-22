# Quarkus Security JWT API Reference

Use this file for SmallRye JWT verification and JWT build APIs with short, copy-ready examples.

## Extension entry points

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-jwt</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-jwt-build</artifactId>
</dependency>
```

## Protect an endpoint with JWT roles

```java
import jakarta.annotation.security.RolesAllowed;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.jwt.JsonWebToken;

@Path("/secured")
class SecuredResource {
    @Inject
    JsonWebToken jwt;

    @GET
    @RolesAllowed({ "User", "Admin" })
    String me() {
        return jwt.getName();
    }
}
```

`JsonWebToken` is the current authenticated principal and exposes standard and custom claims.

## Inject a specific claim

```java
import jakarta.enterprise.context.RequestScoped;
import jakarta.inject.Inject;
import org.eclipse.microprofile.jwt.Claim;
import org.eclipse.microprofile.jwt.Claims;

@RequestScoped
class BirthdateView {
    @Inject
    @Claim(standard = Claims.birthdate)
    String birthdate;
}
```

Use `@RequestScoped` when injecting claims as simple types.

## Read claims from `SecurityContext`

```java
import jakarta.ws.rs.core.Context;
import jakarta.ws.rs.core.SecurityContext;
import org.eclipse.microprofile.jwt.JsonWebToken;

String caller(@Context SecurityContext ctx) {
    JsonWebToken jwt = (JsonWebToken) ctx.getUserPrincipal();
    return jwt.getClaim("email");
}
```

## Parse or verify a token manually

```java
import jakarta.inject.Inject;
import org.eclipse.microprofile.jwt.JsonWebToken;
import io.smallrye.jwt.auth.principal.JWTParser;

class TokenService {
    @Inject
    JWTParser parser;

    JsonWebToken parse(String token) throws Exception {
        return parser.parse(token);
    }
}
```

Use `JWTParser` when the token is not coming from the standard HTTP bearer header flow.

## Verify with a custom secret

```java
JsonWebToken jwt = parser.verify(token, "AyM1SysPpbyDfgZld3umj1qzKObwVMko");
```

This keeps normal JWT validation active while overriding the verification key for that parse operation.

## Build a signed token

```java
import java.util.Set;
import io.smallrye.jwt.build.Jwt;

String token = Jwt.issuer("https://example.com/issuer")
        .upn("jdoe@quarkus.io")
        .groups(Set.of("User", "Admin"))
        .claim("birthdate", "2001-07-13")
        .sign();
```

Default claims include `iat`, `exp`, and `jti` when not already set.

## Build from existing claims

```java
String token = Jwt.claims("/tokenClaims.json").sign();
String encrypted = Jwt.claims("/tokenClaims.json").jwe().encrypt();
String nested = Jwt.claims("/tokenClaims.json").innerSign().encrypt();
```

## Customize signing or encryption algorithms

```java
import io.smallrye.jwt.ContentEncryptionAlgorithm;
import io.smallrye.jwt.KeyEncryptionAlgorithm;
import io.smallrye.jwt.SignatureAlgorithm;

String signed = Jwt.upn("alice")
        .jws()
        .algorithm(SignatureAlgorithm.PS256)
        .sign();

String encrypted = Jwt.subject("bob")
        .jwe()
        .keyAlgorithm(KeyEncryptionAlgorithm.RSA_OAEP_256)
        .contentAlgorithm(ContentEncryptionAlgorithm.A256CBC_HS512)
        .encrypt();
```

## Fast helpers

```java
String signed = Jwt.sign("/claims.json");
String encrypted = Jwt.encrypt("/claims.json");
String nested = Jwt.signAndEncrypt("/claims.json");
```
