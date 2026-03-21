# Quarkus Service Communication REST Usage Patterns

Use these patterns for repeatable outbound HTTP integration workflows.

## Pattern: Define a typed downstream client

When to use:

- One service calls another service with a stable HTTP contract.
- You want config-driven base URLs and typed payload mapping.

Example:

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.PathParam;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;
import org.eclipse.microprofile.rest.client.inject.RestClient;

@Path("/customers")
@RegisterRestClient(configKey = "customer-api")
public interface CustomerClient {
    @GET
    @Path("/{id}")
    CustomerView get(@PathParam("id") String id);
}

@ApplicationScoped
public class CustomerGateway {
    @Inject
    @RestClient
    CustomerClient customers;

    public CustomerView load(String id) {
        return customers.get(id);
    }
}
```

## Pattern: Propagate auth or correlation headers

When to use:

- Downstream services need caller identity, bearer tokens, or request IDs.

Example:

```java
import org.eclipse.microprofile.rest.client.annotation.ClientHeaderParam;

@Path("/reports")
@RegisterRestClient(configKey = "report-api")
@ClientHeaderParam(name = "Authorization", value = "{authorization}")
@ClientHeaderParam(name = "X-Request-Id", value = "{requestId}")
public interface ReportClient {
    @GET
    ReportSummary summary();

    default String authorization(String name) {
        return SecurityContextHolder.bearerToken();
    }

    default String requestId(String name) {
        return RequestIds.current();
    }
}
```

Keep propagation logic at the client edge instead of scattering header code through services.

## Pattern: Combine REST client with fault tolerance

When to use:

- A downstream dependency is important but can fail transiently.

Example:

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import org.eclipse.microprofile.faulttolerance.CircuitBreaker;
import org.eclipse.microprofile.faulttolerance.Fallback;
import org.eclipse.microprofile.faulttolerance.Retry;
import org.eclipse.microprofile.rest.client.inject.RestClient;

@ApplicationScoped
public class QuoteService {
    @Inject
    @RestClient
    QuoteClient quotes;

    @Retry(maxRetries = 2, delay = 200)
    @CircuitBreaker(requestVolumeThreshold = 4, failureRatio = 0.5, delay = 1000)
    @Fallback(fallbackMethod = "cachedQuote")
    public Quote load(String sku) {
        return quotes.quote(sku);
    }

    Quote cachedQuote(String sku) {
        return Quote.unavailable(sku);
    }
}
```

Put retries and fallbacks on the service method that owns the business decision, not blindly on every client method.

## Pattern: Compose non-blocking downstream calls

When to use:

- The request flow is already reactive.
- You need to combine multiple remote calls without blocking threads.

Example:

```java
import io.smallrye.mutiny.Uni;

public Uni<OrderView> loadOrder(String id) {
    return orders.get(id)
            .onItem().transformToUni(order -> shipments.get(order.shipmentId())
                    .map(shipment -> new OrderView(order, shipment)));
}
```

Keep the full chain reactive; avoid `.await().indefinitely()` in request paths.

## Pattern: Choose JSON unless the contract needs multipart

When to use:

- Most service calls exchange structured data only.
- A subset of calls must send files plus metadata.

Example:

```java
import jakarta.ws.rs.Consumes;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.core.MediaType;
import org.jboss.resteasy.reactive.PartType;
import org.jboss.resteasy.reactive.RestForm;

@POST
@Path("/imports/json")
ImportRequest submit(ImportRequest request);

@POST
@Path("/imports/upload")
@Consumes(MediaType.MULTIPART_FORM_DATA)
ImportResult upload(@RestForm("file") byte[] file,
                    @RestForm @PartType(MediaType.APPLICATION_JSON) ImportMetadata metadata);
```

Default to JSON for ordinary RPC-style calls; reserve multipart for contracts that truly include binary parts.
