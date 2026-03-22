# Quarkus Security JWT Configuration Reference

Use this file for high-value SmallRye JWT verification, decryption, signing, and encryption settings.

## High-value verification properties

| Property | Default | Use when |
|----------|---------|----------|
| `mp.jwt.verify.publickey.location` | - | The verification key should come from classpath, filesystem, or remote JWKS/JWK location |
| `mp.jwt.verify.publickey` | - | The public key should be provided inline instead of by location |
| `mp.jwt.verify.publickey.algorithm` | `RS256` | Verification uses EC signatures such as `ES256` |
| `mp.jwt.verify.issuer` | - | Only tokens from a specific issuer should be accepted |
| `mp.jwt.verify.audiences` | - | The token audience must match one of the allowed values |
| `mp.jwt.verify.clock.skew` | `60` | Validation should tolerate bounded clock drift |
| `mp.jwt.verify.token.age` | - | Tokens older than a fixed age must be rejected |
| `mp.jwt.decrypt.key.location` | - | The service accepts encrypted JWTs or nested signed+encrypted JWTs |
| `mp.jwt.token.header` | `Authorization` | The token arrives in a non-default header or via cookies |
| `mp.jwt.token.cookie` | - | The token is carried in a named cookie |

## High-value SmallRye verification properties

| Property | Default | Use when |
|----------|---------|----------|
| `smallrye.jwt.verify.key.location` | `NONE` | A single location should cover either public or symmetric verification keys |
| `smallrye.jwt.verify.algorithm` | - | Symmetric verification must use `HS256`, `HS384`, or `HS512` |
| `smallrye.jwt.verify.key-format` | `ANY` | Key loading should be constrained to PEM, JWK, or another known format |
| `smallrye.jwt.verify.key-provider` | `DEFAULT` | Keys are resolved dynamically, for example from AWS ALB |
| `smallrye.jwt.verify.secretkey` | - | A symmetric key is supplied directly as config |
| `quarkus.smallrye-jwt.blocking-authentication` | auto | Remote key resolution can block and should move off the event loop |
| `quarkus.smallrye-jwt.silent` | `false` | JWT auth should avoid forcing a 401 challenge in mixed browser flows |

## High-value token build properties

| Property | Default | Use when |
|----------|---------|----------|
| `smallrye.jwt.sign.key.location` | - | `sign()` or `innerSign()` should load a signing key automatically |
| `smallrye.jwt.sign.key` | - | The signing key value comes from a secret manager or config source |
| `smallrye.jwt.encrypt.key.location` | - | `encrypt()` should load the encryption key automatically |
| `smallrye.jwt.encrypt.key` | - | The encryption key value comes from a secret manager or config source |
| `smallrye.jwt.new-token.signature-algorithm` | `RS256` | New tokens should default to a different signing algorithm |
| `smallrye.jwt.new-token.key-encryption-algorithm` | `RSA-OAEP` | New encrypted tokens should use a different key management algorithm |
| `smallrye.jwt.new-token.content-encryption-algorithm` | `A256GCM` | New encrypted tokens should use a different content encryption algorithm |
| `smallrye.jwt.new-token.lifespan` | `300` | Default token lifetime should change |
| `smallrye.jwt.new-token.issuer` | - | New tokens should get a default `iss` claim |
| `smallrye.jwt.new-token.audience` | - | New tokens should get a default `aud` claim |
| `smallrye.jwt.new-token.add-default-claims` | `true` | Automatic `iat`, `exp`, and `jti` claims should be disabled |

## Typical local verification setup

```properties
mp.jwt.verify.publickey.location=publicKey.pem
mp.jwt.verify.issuer=https://example.com/issuer
quarkus.native.resources.includes=publicKey.pem
```

## Typical remote JWKS verification setup

```properties
mp.jwt.verify.publickey.location=https://idp.example.com/realms/app/protocol/openid-connect/certs
mp.jwt.verify.issuer=https://idp.example.com/realms/app
quarkus.smallrye-jwt.blocking-authentication=true
```

## Accept encrypted or nested tokens

```properties
mp.jwt.verify.publickey.location=publicKey.pem
mp.jwt.decrypt.key.location=privateKey.pem
```

With both verification and decryption configured, Quarkus expects nested signed-then-encrypted tokens.

## Configure token building

```properties
smallrye.jwt.sign.key.location=privateKey.pem
smallrye.jwt.encrypt.key.location=publicKey.pem
smallrye.jwt.new-token.issuer=https://example.com/issuer
smallrye.jwt.new-token.lifespan=900
```

## Dev and test behavior

Quarkus generates a temporary RSA key pair in dev and test when no verification, decryption, signing, or encryption key is configured. This is convenient for tests but should not be relied on for production behavior.
