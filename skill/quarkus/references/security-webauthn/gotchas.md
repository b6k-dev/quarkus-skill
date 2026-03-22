# Quarkus Security WebAuthn Gotchas

Common WebAuthn and passkey pitfalls, symptoms, and fixes.

## Registration safety

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| A new passkey can be attached to an existing account by an unauthenticated user | `WebAuthnUserProvider.store` allows duplicate usernames during registration | Reject registration for existing usernames unless the current session is already authenticated as that same user |
| Only the first authenticator works for a user | The data model assumes one credential per username | Model credential IDs separately and return all matching credentials from `findByUsername` |

## Origin and browser context issues

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Browser refuses to start the ceremony | The page is not served from HTTPS and is not `localhost` | Use HTTPS outside localhost; make sure the browser sees a secure context |
| Registration works locally but fails behind a proxy or on another hostname | `quarkus.webauthn.origins` or `quarkus.webauthn.relying-party.id` do not match the public URL | Set explicit origins and RP ID based on the externally visible hostnames |
| Cross-subdomain login or embedded flows fail unexpectedly | Cookie same-site policy is too strict for the deployment shape | Review `quarkus.webauthn.cookie-same-site` and related cookie scoping |

## Endpoint ownership mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Default `POST /q/webauthn/register` fails at runtime | Built-in registration is enabled but `WebAuthnUserProvider.store` is missing or incomplete | Implement `store` or disable the default registration endpoint |
| Default `POST /q/webauthn/login` fails to update counters | Built-in login is enabled but `WebAuthnUserProvider.update` is missing or ignored | Implement `update` and persist the new signature counter |
| Custom endpoints validate WebAuthn but users do not stay logged in | `WebAuthnSecurity` was used without issuing a cookie or token | Call `rememberUser(...)` for cookie sessions or issue your own token/session explicitly |

## Credential lifecycle and policy issues

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Username-less passkey login never finds a user | Discoverable credentials are expected but resident key behavior was relaxed | Keep `quarkus.webauthn.resident-key=required` for passkey-first flows unless you know the UX tradeoff |
| Device attestation settings appear ignored or inconsistent | Browsers may reduce or override attestation for privacy | Treat attestation as an advanced policy feature; test real browsers before depending on it |
| Counter updates or reads block the event loop | Blocking persistence is used from WebAuthn callbacks without `@Blocking` or virtual-thread handoff | Mark the provider appropriately and keep blocking I/O off the event loop |
