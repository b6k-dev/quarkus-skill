# Quarkus Security OIDC Client API Reference

Use this file for outbound token acquisition and propagation APIs with short, copy-ready examples.

## Extension entry points

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-oidc-client</artifactId>
</dependency>

<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-client-oidc-filter</artifactId>
</dependency>

<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-client-oidc-token-propagation</artifactId>
</dependency>
```

Use the RESTEasy Classic filter variants only when the application is still built around classic REST clients.

## Minimal client-credentials setup

```properties
quarkus.oidc-client.auth-server-url=https://idp.example.com/realms/acme
quarkus.oidc-client.client-id=inventory-service
quarkus.oidc-client.credentials.secret=${inventory-service-secret}
quarkus.oidc-client.grant.type=client
```

`client` is the normal starting point for service-to-service calls made under the service's own identity.

## Inject `Tokens`

```java
import io.quarkus.oidc.client.Tokens;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;

@ApplicationScoped
public class DownstreamAuthHeaderFactory {
    @Inject
    Tokens tokens;

    public String authorization() {
        return "Bearer " + tokens.getAccessToken();
    }
}
```

`Tokens` is the simplest API when you need the current outbound access token and want Quarkus to handle refresh.

## Use `OidcClient` directly

```java
import io.quarkus.oidc.client.OidcClient;
import io.quarkus.oidc.client.runtime.TokensHelper;
import io.smallrye.mutiny.Uni;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;

@ApplicationScoped
public class TokenService {
    @Inject
    OidcClient client;

    private final TokensHelper tokensHelper = new TokensHelper();

    public Uni<String> accessToken() {
        return tokensHelper.getTokens(client)
                .map(tokens -> tokens.getAccessToken());
    }
}
```

Use `OidcClient` when you need reactive composition or non-default grants such as token exchange.

## Attach a token to a REST client manually

```java
import io.smallrye.mutiny.Uni;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.HeaderParam;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

@Path("/inventory")
@RegisterRestClient(configKey = "inventory-api")
public interface InventoryClient {
    @GET
    Uni<String> get(@HeaderParam("Authorization") String authorization);
}
```

Manual header passing is useful when token choice is dynamic per call.

## Use the OIDC client filter on a reactive REST client

```java
import io.quarkus.oidc.client.filter.OidcClientFilter;
import io.smallrye.mutiny.Uni;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

@Path("/inventory")
@RegisterRestClient(configKey = "inventory-api")
@OidcClientFilter
public interface InventoryClient {
    @GET
    Uni<String> get();
}
```

`@OidcClientFilter` is the usual choice when one outbound client should always send a token from one OIDC client.

## Select a named OIDC client

```java
import io.quarkus.oidc.client.NamedOidcClient;
import io.quarkus.oidc.client.Tokens;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;

@ApplicationScoped
public class PaymentsAuthHeaderFactory {
    @Inject
    @NamedOidcClient("payments")
    Tokens tokens;

    public String authorization() {
        return "Bearer " + tokens.getAccessToken();
    }
}
```

Use named clients when different downstreams need different credentials, scopes, or grant options.

## Propagate the incoming access token

```java
import io.quarkus.oidc.token.propagation.common.AccessToken;
import io.smallrye.mutiny.Uni;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

@Path("/reports")
@RegisterRestClient(configKey = "reports-api")
@AccessToken
public interface ReportsClient {
    @GET
    Uni<String> currentUserReport();
}
```

Use propagation only when the downstream should act on behalf of the current caller.

## Exchange a propagated token before forwarding it

```java
import io.quarkus.oidc.token.propagation.common.AccessToken;
import io.smallrye.mutiny.Uni;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

@Path("/ledger")
@RegisterRestClient(configKey = "ledger-api")
@AccessToken(exchangeTokenClient = "ledger-exchange")
public interface LedgerClient {
    @GET
    Uni<String> entries();
}
```

This is the safer pattern for multi-hop service calls when the downstream should receive a token minted for that service boundary, not the original caller token.

## Create a client programmatically

```java
import io.quarkus.oidc.client.OidcClient;
import io.quarkus.oidc.client.OidcClients;
import io.quarkus.oidc.client.runtime.OidcClientConfig;
import io.smallrye.mutiny.Uni;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;

@ApplicationScoped
public class DynamicOidcClientFactory {
    @Inject
    OidcClients clients;

    public Uni<OidcClient> create(String authServerUrl, String clientId, String secret) {
        OidcClientConfig config = OidcClientConfig
                .authServerUrl(authServerUrl)
                .id("dynamic")
                .clientId(clientId)
                .credentials(secret)
                .build();
        return clients.newClient(config);
    }
}
```

Programmatic creation is an edge-case tool for dynamic tenants or credentials that cannot be declared statically.
