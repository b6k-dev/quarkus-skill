# Quarkus gRPC Usage Patterns

Use these patterns for repeatable gRPC contract, implementation, client, and testing workflows.

## Pattern: Start proto-first

When to use:

- Two services are owned together or share a versioned contract artifact.
- You want generated request, response, and client types instead of handwritten DTOs.

Example:

```proto
syntax = "proto3";

option java_multiple_files = true;
option java_package = "org.acme.inventory";

package inventory;

service InventoryQuery {
    rpc GetStock (GetStockRequest) returns (StockReply) {}
}

message GetStockRequest {
    string sku = 1;
}

message StockReply {
    string sku = 1;
    int32 available = 2;
}
```

Keep domain entities out of the `.proto` contract; design transport messages explicitly for cross-service use.

## Pattern: Implement a service with the generated Mutiny interface

When to use:

- You are writing new Quarkus gRPC code.
- Service logic is naturally reactive or can at least return `Uni` and `Multi`.

Example:

```java
import io.quarkus.grpc.GrpcService;
import io.smallrye.mutiny.Uni;
import org.acme.inventory.GetStockRequest;
import org.acme.inventory.InventoryQuery;
import org.acme.inventory.StockReply;

@GrpcService
public class InventoryQueryService implements InventoryQuery {
    @Override
    public Uni<StockReply> getStock(GetStockRequest request) {
        return stockService.find(request.getSku())
                .onItem().transform(stock -> StockReply.newBuilder()
                        .setSku(stock.sku())
                        .setAvailable(stock.available())
                        .build());
    }
}
```

Prefer this over `StreamObserver` for new services because the method shapes align better with Quarkus reactive APIs.

## Pattern: Consume another service through an injected client boundary

When to use:

- One Quarkus service depends on another internal gRPC service.
- You want endpoint configuration outside application code.

Example:

```java
import io.quarkus.grpc.GrpcClient;
import io.smallrye.mutiny.Uni;
import jakarta.enterprise.context.ApplicationScoped;
import org.acme.inventory.GetStockRequest;
import org.acme.inventory.InventoryQuery;

@ApplicationScoped
public class InventoryGateway {
    @GrpcClient("inventory")
    InventoryQuery inventory;

    public Uni<Integer> available(String sku) {
        return inventory.getStock(GetStockRequest.newBuilder().setSku(sku).build())
                .onItem().transform(reply -> reply.getAvailable());
    }
}
```

Keep the rest of the application talking to a gateway or service bean, not to generated stubs scattered everywhere.

## Pattern: Use unary RPC by default

When to use:

- One request produces one result.
- The call should be easy to reason about, retry, and observe.

Example:

```proto
rpc ReserveStock (ReserveStockRequest) returns (ReserveStockReply) {}
```

Start with unary RPC unless you have a real incremental stream on at least one side.

## Pattern: Use streaming only for incremental work

When to use:

- The server produces multiple results over time.
- The client needs to upload many items progressively.
- Both sides benefit from continuous bidirectional flow.

Example:

```java
import io.smallrye.mutiny.Multi;

@Override
public Multi<PriceTick> watchPrices(WatchPricesRequest request) {
    return marketFeed.stream(request.getSymbol())
            .onItem().transform(this::toReply);
}
```

Streaming adds lifecycle and back-pressure complexity, so use it for genuinely streamed data rather than as a default transport style.

## Pattern: Mark blocking boundaries explicitly

When to use:

- Service logic hits JDBC, JPA, filesystem, or any blocking client.

Example:

```java
import io.smallrye.common.annotation.Blocking;

@Override
@Blocking
public Uni<StockReply> getStock(GetStockRequest request) {
    Stock stock = repository.findBySku(request.getSku());
    return Uni.createFrom().item(() -> toReply(stock));
}
```

Do not hide blocking work behind a reactive signature; tell Quarkus where thread offloading is required.

## Pattern: Map domain models to proto DTOs at the edge

When to use:

- Your domain model does not match the transport model exactly.
- You want the protobuf contract to remain stable while internals evolve.

Example:

```java
private StockReply toReply(Stock stock) {
    return StockReply.newBuilder()
            .setSku(stock.sku())
            .setAvailable(stock.available())
            .build();
}
```

Avoid returning persistence entities, Panache models, or internal enums directly across the RPC boundary.

## Pattern: Put deadlines in configuration, not business code

When to use:

- Calls should fail fast based on environment or dependency behavior.
- Different deployments need different timeout budgets.

Example:

```properties
quarkus.grpc.clients.inventory.deadline=1500ms
```

Per-environment config keeps latency budgets adjustable without recompiling client code.

## Pattern: Propagate auth or tenant context with interceptors

When to use:

- Every call needs the same metadata headers.
- You want cross-cutting transport logic outside feature code.

Example:

```java
import io.grpc.ClientInterceptor;
import io.quarkus.grpc.GlobalInterceptor;
import jakarta.enterprise.context.ApplicationScoped;

@GlobalInterceptor
@ApplicationScoped
public class TenantHeaderInterceptor implements ClientInterceptor {
}
```

Use interceptors for repeated metadata logic; use ad hoc header attachment only for exceptional one-off calls.

## Pattern: Integration test with `@QuarkusTest`

When to use:

- You want to verify generated stubs, service registration, and config together.

Example:

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import io.quarkus.grpc.GrpcClient;
import io.quarkus.test.junit.QuarkusTest;
import jakarta.inject.Inject;
import org.junit.jupiter.api.Test;

@QuarkusTest
class InventoryQueryTest {
    @Inject
    @GrpcClient("inventory")
    InventoryQuery inventory;

    @Test
    void shouldReturnStock() {
        StockReply reply = inventory.getStock(GetStockRequest.newBuilder().setSku("A-1").build())
                .await().indefinitely();
        assertEquals(12, reply.getAvailable());
    }
}
```

Prefer full integration tests for gRPC because configuration, generated code, and transport wiring matter as much as method logic.

## Pattern: Use reflection for local inspection only

When to use:

- You want to inspect services with `grpcurl` during development or test troubleshooting.

Example:

```properties
quarkus.grpc.server.enable-reflection-service=true
```

Disable or review reflection exposure deliberately in production environments.
