# Quarkus Security WebAuthn Configuration Reference

Use this file for high-value WebAuthn, relying-party, and cookie settings.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.webauthn.enable-registration-endpoint` | `false` | You want Quarkus to expose `POST /q/webauthn/register` and call `WebAuthnUserProvider.store` |
| `quarkus.webauthn.enable-login-endpoint` | `false` | You want Quarkus to expose `POST /q/webauthn/login` and call `WebAuthnUserProvider.update` |
| `quarkus.webauthn.origins` | deployed application origin | The browser origin differs from Quarkus' observed external URL or multiple origins must be accepted |
| `quarkus.webauthn.relying-party.id` | first allowed origin host | The RP ID must be pinned explicitly, often behind proxies or shared hostnames |
| `quarkus.webauthn.relying-party.name` | `Quarkus server` | Passkey prompts should show a user-friendly relying party name |
| `quarkus.webauthn.transports` | all common transports | You want to restrict authenticators to internal or roaming devices |
| `quarkus.webauthn.authenticator-attachment` | any | Registration should prefer platform or cross-platform authenticators |
| `quarkus.webauthn.resident-key` | `required` | You want to relax or enforce discoverable credential behavior |
| `quarkus.webauthn.user-verification` | `required` | UX or security policy requires stronger or weaker biometric/PIN verification |
| `quarkus.webauthn.attestation` | `none` | You need attestation signals during registration |
| `quarkus.webauthn.load-metadata` | `false` | Attestation verification should use FIDO metadata |
| `quarkus.webauthn.timeout` | `5m` | Ceremonies need a shorter or longer challenge lifetime |
| `quarkus.webauthn.login-page` | `/login.html` | Unauthenticated redirects should land on a custom page |
| `quarkus.webauthn.session-timeout` | `30M` | Cookie-backed login sessions should expire sooner or later |
| `quarkus.webauthn.new-cookie-interval` | `1M` | Cookie renewal cadence should be tuned relative to idle timeout |
| `quarkus.webauthn.cookie-name` | `quarkus-credential` | The main session cookie name must avoid collisions |
| `quarkus.webauthn.challenge-cookie-name` | `_quarkus_webauthn_challenge` | Challenge cookie naming must avoid collisions |
| `quarkus.webauthn.cookie-same-site` | `strict` | Cross-site embedding or redirected flows need looser same-site behavior |
| `quarkus.webauthn.cookie-path` | `/` | Session or challenge cookies should be scoped to a narrower path |
| `quarkus.webauthn.cookie-max-age` | session cookie | The browser should persist the login cookie beyond a browser restart |

## Minimal built-in endpoint setup

```properties
quarkus.webauthn.enable-registration-endpoint=true
quarkus.webauthn.enable-login-endpoint=true
```

Use this only when your `WebAuthnUserProvider` implements both `store` and `update`.

## Relying party and origin setup

```properties
quarkus.webauthn.origins=https://app.example.com,https://app.example.co.uk
quarkus.webauthn.relying-party.id=example.com
quarkus.webauthn.relying-party.name=Example Passkeys
```

Keep the effective browser origin, proxy headers, and RP ID aligned or authenticator ceremonies will fail.

## Authenticator policy example

```properties
quarkus.webauthn.transports=internal
quarkus.webauthn.authenticator-attachment=platform
quarkus.webauthn.resident-key=required
quarkus.webauthn.user-verification=required
```

This setup biases toward device-bound passkeys with local verification.

## Cookie and session controls

```properties
quarkus.webauthn.session-timeout=30M
quarkus.webauthn.new-cookie-interval=5M
quarkus.webauthn.cookie-same-site=strict
quarkus.webauthn.cookie-path=/
```

The encrypted session cookie also depends on `quarkus.http.auth.session.encryption-key` for stable, production-safe session handling.
