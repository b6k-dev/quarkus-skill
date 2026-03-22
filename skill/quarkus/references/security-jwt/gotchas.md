# Quarkus Security JWT Gotchas

Common JWT verification, claim injection, and token-building pitfalls.

## Verification and token source

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Valid-looking token is rejected with 401 | `iss` does not match `mp.jwt.verify.issuer` | Align issuer in both token generation and server config |
| Token is never seen by JWT auth | Token was sent in a cookie or custom header but `mp.jwt.token.header` and `mp.jwt.token.cookie` were not configured | Configure the actual token source |
| HS256 token never verifies | Symmetric verification was configured like an RSA/EC public key flow | Use `smallrye.jwt.verify.key.location` or `smallrye.jwt.verify.secretkey` with `smallrye.jwt.verify.algorithm=HS256` |

## Keys and crypto

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Remote JWKS lookup causes event-loop blocking warnings | Key refresh performs network IO on the authentication path | Enable `quarkus.smallrye-jwt.blocking-authentication=true` |
| Encrypted token can be decrypted but sender identity is unclear | Claims were encrypted directly with a public key and not signed first | Prefer `innerSign().encrypt()` for sender authenticity plus confidentiality |
| Native executable cannot load HTTPS JWKS | Remote HTTPS key retrieval needs native SSL support | Enable the relevant native SSL support and verify the remote cert setup |

## CDI and claims

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Claim injection as `String` or another simple type fails | Bean is not `@RequestScoped` | Use `@RequestScoped` for direct claim injection |
| Public endpoint stops seeing injected JWT after disabling proactive auth | Token verification no longer runs for anonymous/public requests | Re-enable proactive auth or avoid expecting injected JWT on public methods |

## Dev, test, and operational surprises

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Tests pass without configured keys but prod fails | Dev/test generated temporary RSA keys automatically | Configure real verification/signing keys in production-like tests |
| Verification behavior differs after moving from local key to remote JWKS | Runtime now depends on remote availability, key rotation, and issuer alignment | Test the full remote-key path, not only token contents |
| JWT build output is missing expected issuer or audience | Defaults were not configured and claims were not set on the builder | Set claims explicitly or configure `smallrye.jwt.new-token.*` defaults |
