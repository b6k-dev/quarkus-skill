# Quarkus Security OIDC Client Configuration Reference

Use this file for high-value OIDC client, refresh, and token propagation settings that matter in outbound service calls.

## High-value OIDC client properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.oidc-client.auth-server-url` | none | The token endpoint should be discovered from the provider metadata |
| `quarkus.oidc-client.discovery-enabled` | `true` | Discovery is unavailable or you want to configure the token endpoint directly |
| `quarkus.oidc-client.token-path` | discovered | The token endpoint must be given explicitly as a relative path or full URL |
| `quarkus.oidc-client.client-id` | none | The outbound client identity must be declared |
| `quarkus.oidc-client.credentials.secret` | none | Shared-secret client authentication is used |
| `quarkus.oidc-client.grant.type` | `client` | You need `password`, `exchange`, `jwt`, `refresh`, or another non-default grant |
| `quarkus.oidc-client.scopes` | none | The token must include explicit scopes |
| `quarkus.oidc-client.refresh-token-time-skew` | disabled | Quarkus should refresh shortly before expiry instead of waiting for a near-dead token |
| `quarkus.oidc-client.refresh-interval` | disabled | Filter-managed tokens should refresh asynchronously in the background |
| `quarkus.oidc-client.early-tokens-acquisition` | `true` | Initial token acquisition must be delayed until first use |

## Minimal service-to-service configuration

```properties
quarkus.oidc-client.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc-client.client-id=inventory-service
quarkus.oidc-client.credentials.secret=${inventory-service-secret}
quarkus.oidc-client.grant.type=client
quarkus.oidc-client.scopes=inventory.read
```

Prefer `grant.type=client` for service identity calls that should not impersonate an end user.

## Disable discovery and set the token endpoint directly

```properties
quarkus.oidc-client.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc-client.discovery-enabled=false
quarkus.oidc-client.token-path=/protocol/openid-connect/token
```

You can also set `quarkus.oidc-client.token-path` to a full URL when discovery and `auth-server-url` are not useful.

## Named clients

```properties
quarkus.oidc-client.client-enabled=false

quarkus.oidc-client.inventory.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc-client.inventory.client-id=inventory-service
quarkus.oidc-client.inventory.credentials.secret=${inventory-secret}
quarkus.oidc-client.inventory.grant.type=client
quarkus.oidc-client.inventory.scopes=inventory.read

quarkus.oidc-client.payments.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc-client.payments.client-id=payments-service
quarkus.oidc-client.payments.credentials.secret=${payments-secret}
quarkus.oidc-client.payments.grant.type=client
quarkus.oidc-client.payments.scopes=payments.write
```

- `quarkus.oidc-client.<name>.*` defines a named outbound client.
- `client-enabled=false` disables the unnamed default client when every caller should be explicit.
- Named clients are a good fit when downstream systems have different scopes, audiences, or credentials.

## Password grant for user-bound outbound calls

```properties
quarkus.oidc-client.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc-client.client-id=frontend-service
quarkus.oidc-client.credentials.secret=${frontend-secret}
quarkus.oidc-client.grant.type=password
quarkus.oidc-client.grant-options.password.username=alice
quarkus.oidc-client.grant-options.password.password=${alice-password}
```

Use this only for narrowly justified legacy or machine-driven flows; it is not the normal service-to-service default.

## Token refresh behavior

```properties
quarkus.oidc-client.refresh-token-time-skew=30S
quarkus.oidc-client.refresh-interval=1M
quarkus.oidc-client.early-tokens-acquisition=false
```

- `refresh-token-time-skew` refreshes slightly early to reduce avoidable `401` races.
- `refresh-interval` helps filter-managed tokens refresh off the request path.
- `early-tokens-acquisition=false` avoids grabbing a token at startup or filter init when it may expire before first use.

## REST client OIDC filter settings

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.rest-client-oidc-filter.client-name` | default client | A reactive REST client should use a named OIDC client |
| `quarkus.rest-client-oidc-filter.refresh-on-unauthorized` | `false` | The filter should retry with a refreshed token after a downstream `401` |
| `quarkus.resteasy-client-oidc-filter.client-name` | default client | A classic REST client should use a named OIDC client |
| `quarkus.resteasy-client-oidc-filter.refresh-on-unauthorized` | `false` | The classic filter should retry after a downstream `401` |

Reactive REST clients should generally use the `quarkus.rest-client-oidc-filter.*` settings.

## Token propagation and exchange settings

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.rest-client-oidc-token-propagation.exchange-token` | `false` | The incoming token should be exchanged before forwarding |
| `quarkus.rest-client-oidc-token-propagation.client-name` | default client | Token exchange should use a named OIDC client |
| `quarkus.rest-client-oidc-token-propagation.enabled-during-authentication` | `false` | Propagation is needed during `SecurityIdentity` augmentation |
| `quarkus.resteasy-client-oidc-token-propagation.exchange-token` | `false` | A classic REST client should exchange instead of forwarding the token |
| `quarkus.resteasy-client-oidc-token-propagation.client-name` | default client | Classic token exchange should use a named OIDC client |

## Exchange-token setup for multi-hop calls

```properties
quarkus.oidc-client.ledger-exchange.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc-client.ledger-exchange.client-id=service-a
quarkus.oidc-client.ledger-exchange.credentials.secret=${service-a-secret}
quarkus.oidc-client.ledger-exchange.grant.type=exchange
quarkus.oidc-client.ledger-exchange.grant-options.exchange.audience=ledger-api

quarkus.rest-client-oidc-token-propagation.client-name=ledger-exchange
quarkus.rest-client-oidc-token-propagation.exchange-token=true
```

Use this when service A must call service B on behalf of a user, but service B should receive a token minted for service B rather than the original caller token.
