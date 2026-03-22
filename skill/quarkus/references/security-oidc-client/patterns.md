# Security OIDC Client Usage Patterns

Use these patterns to keep outbound token acquisition and propagation explicit at the service boundary.

## Pattern: Service calls another service as itself

When to use:

- The downstream API authorizes the caller as a workload or backend service.
- No end-user identity should be forwarded.

Example:

```properties
quarkus.oidc-client.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc-client.client-id=orders-service
quarkus.oidc-client.credentials.secret=${orders-secret}
quarkus.oidc-client.grant.type=client
```

Prefer `client_credentials` here. It produces a token that clearly represents the calling service rather than the original user.

## Pattern: One REST client always uses one outbound token source

When to use:

- Every call from one client interface should carry the same OIDC client identity.

Example:

```java
@Path("/catalog")
@RegisterRestClient(configKey = "catalog-api")
@OidcClientFilter("catalog")
public interface CatalogClient {
    @GET
    Uni<String> get();
}
```

Prefer `@OidcClientFilter` over manual header assembly when the choice is static.

## Pattern: Different downstreams need different clients

When to use:

- Downstreams require different scopes, audiences, or credentials.

Example:

```properties
quarkus.oidc-client.inventory.client-id=inventory-service
quarkus.oidc-client.payments.client-id=payments-service
```

Give each downstream a named OIDC client and keep that mapping explicit in code with `@NamedOidcClient` or `@OidcClientFilter("name")`.

## Pattern: Forward the current user token to one downstream hop

When to use:

- The downstream must act on behalf of the currently authenticated caller.
- The propagated token is meant for that downstream boundary.

Example:

```java
@Path("/reports")
@RegisterRestClient(configKey = "reports-api")
@AccessToken
public interface ReportsClient {
    @GET
    Uni<String> currentUserReport();
}
```

This fits delegation scenarios better than acquiring a service token, but keep the hop count intentional.

## Pattern: Exchange before forwarding in multi-hop service chains

When to use:

- `Client -> Service A -> Service B` should not reuse the exact same bearer token end to end.
- Service B must receive a token with a new audience, issuer context, or reduced scope.

Example:

```java
@Path("/ledger")
@RegisterRestClient(configKey = "ledger-api")
@AccessToken(exchangeTokenClient = "ledger-exchange")
public interface LedgerClient {
    @GET
    Uni<String> entries();
}
```

Prefer exchange over raw propagation for internal service chains where trust boundaries matter.

## Pattern: Keep token logic in the integration adapter

When to use:

- Application services should not know how tokens are fetched, refreshed, or forwarded.

Example:

```java
@ApplicationScoped
public class LedgerGateway {
    @Inject
    @RestClient
    LedgerClient client;

    public Uni<String> entries() {
        return client.entries();
    }
}
```

Hide `Tokens`, `OidcClient`, and propagation annotations inside the gateway layer so domain code stays transport- and auth-light.

## Pattern: Use manual token passing only when token choice is per call

When to use:

- The token depends on runtime data such as tenant, user, or an explicit exchange result.

Example:

```java
return tokensHelper.getTokens(client)
        .onItem().transformToUni(tokens -> downstream.get("Bearer " + tokens.getAccessToken()));
```

Manual header passing is a deliberate escape hatch, not the default for a stable client boundary.
