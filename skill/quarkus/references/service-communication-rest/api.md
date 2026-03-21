# Quarkus Service Communication REST API Reference

Use this file for outbound REST client APIs with short, copy-ready examples.

## Extension entry points

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-client</artifactId>
</dependency>
```

JSON variants:

- `io.quarkus:quarkus-rest-client-jackson`
- `io.quarkus:quarkus-rest-client-jsonb`

## Minimal typed client

```java
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

@Path("/inventory")
@RegisterRestClient(configKey = "inventory-api")
public interface InventoryClient {
    @GET
    ItemAvailability get();
}
```

`configKey` keeps configuration stable if the interface package or name changes.

## Inject and use the client

```java
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.rest.client.inject.RestClient;

@ApplicationScoped
public class PricingService {
    @RestClient
    InventoryClient inventory;

    public ItemAvailability availability() {
        return inventory.get();
    }
}
```

Use `@RestClient` on the injection point; the interface type alone is not enough.

## Request parameters

```java
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.PathParam;
import jakarta.ws.rs.QueryParam;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;
import org.jboss.resteasy.reactive.RestQuery;

@Path("/catalog")
@RegisterRestClient(configKey = "catalog-api")
public interface CatalogClient {
    @GET
    @Path("/items/{id}")
    Item get(@PathParam("id") String id,
             @QueryParam("expand") String expand,
             @RestQuery String locale);
}
```

`@RestQuery` is useful when parameter names match method parameter names or when sending maps.

## Headers on every call

```java
import org.eclipse.microprofile.rest.client.annotation.ClientHeaderParam;

@Path("/partners")
@RegisterRestClient(configKey = "partner-api")
@ClientHeaderParam(name = "X-Service-Name", value = "pricing-service")
@ClientHeaderParam(name = "Authorization", value = "{bearerToken}")
public interface PartnerClient {
    @GET
    PartnerStatus status();

    default String bearerToken(String headerName) {
        return "Bearer " + Tokens.current();
    }
}
```

Use `@ClientHeaderParam` for static, config-driven, or computed headers without repeating them on every method.

## Override the target URL per call

```java
import io.quarkus.rest.client.reactive.Url;
import java.net.URI;

@Path("/tenants")
@RegisterRestClient(configKey = "tenant-api")
public interface TenantClient {
    @GET
    @Path("/health")
    HealthStatus health(@Url URI baseUri);
}
```

Use `@Url` for exceptional dynamic routing; prefer configured base URLs for normal service calls.

## Typed response with status and headers

```java
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.PathParam;
import org.jboss.resteasy.reactive.RestResponse;

@GET
@Path("/orders/{id}")
RestResponse<OrderSnapshot> getOrder(@PathParam("id") String id);
```

Prefer `RestResponse<T>` when callers need status codes or response headers alongside the payload.

## Reactive return types

```java
import io.smallrye.mutiny.Uni;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.PathParam;

@GET
@Path("/quotes/{sku}")
Uni<Quote> quote(@PathParam("sku") String sku);
```

Use `Uni<T>` for non-blocking composition. Keep blocking callers off the event loop.

## Client-side exception mapping

```java
import jakarta.ws.rs.core.Response;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;
import org.jboss.resteasy.reactive.ClientExceptionMapper;

@RegisterRestClient(configKey = "inventory-api")
public interface InventoryClient {
    @ClientExceptionMapper
    static RuntimeException map404(Response response) {
        if (response.getStatus() == 404) {
            return new UnknownItemException();
        }
        return null;
    }
}
```

Use `@ClientExceptionMapper` for client-local status mapping without a separate provider class.

## Register providers

```java
import jakarta.ws.rs.ext.Provider;
import org.eclipse.microprofile.rest.client.annotation.RegisterProvider;

@Provider
public class DownstreamCorrelationFilter implements jakarta.ws.rs.client.ClientRequestFilter {
    @Override
    public void filter(jakarta.ws.rs.client.ClientRequestContext requestContext) {
        requestContext.getHeaders().add("X-Request-Id", RequestIds.current());
    }
}

@Path("/billing")
@RegisterRestClient(configKey = "billing-api")
@RegisterProvider(DownstreamCorrelationFilter.class)
public interface BillingClient {
    @GET
    @Path("/summary")
    BillingSummary summary();
}
```

Use `@RegisterProvider` for request filters, response filters, message body providers, and mapper classes tied to one client.

## Multipart for service integrations

```java
import jakarta.ws.rs.Consumes;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.core.MediaType;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;
import org.jboss.resteasy.reactive.PartType;
import org.jboss.resteasy.reactive.RestForm;

@Path("/imports")
@RegisterRestClient(configKey = "import-api")
public interface ImportClient {
    @POST
    @Consumes(MediaType.MULTIPART_FORM_DATA)
    ImportResult upload(@RestForm("file") byte[] file,
                        @RestForm @PartType(MediaType.APPLICATION_JSON) ImportMetadata metadata);
}
```

Prefer JSON for normal service calls; use multipart only when binary payloads are part of the contract.