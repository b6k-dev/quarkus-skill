# Quarkus gRPC API Reference

Use this file for generated gRPC types, service annotations, injected clients, and compact call examples.

## Common extension

Add Quarkus gRPC support:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-grpc</artifactId>
</dependency>
```

Quarkus generates gRPC classes automatically from `.proto` files in `src/main/proto` during the normal build.

## Proto-first contract

Define the service and messages first:

```proto
syntax = "proto3";

option java_multiple_files = true;
option java_package = "org.acme.orders";

package orders;

service OrderQuery {
    rpc GetOrder (GetOrderRequest) returns (OrderReply) {}
    rpc WatchOrders (WatchOrdersRequest) returns (stream OrderReply) {}
}

message GetOrderRequest {
    string id = 1;
}

message WatchOrdersRequest {
    string customer_id = 1;
}

message OrderReply {
    string id = 1;
    string status = 2;
    int64 total_cents = 3;
}
```

Do not enable `java_generic_services`; Quarkus code generation expects the modern generated service model.

## Generated types you use most

From a service named `OrderQuery`, Quarkus commonly generates:

- `org.acme.orders.OrderQuery` - Mutiny service interface for implementation and injection
- `org.acme.orders.MutinyOrderQueryGrpc.MutinyOrderQueryStub` - Mutiny client stub
- `org.acme.orders.OrderQueryGrpc.OrderQueryBlockingStub` - blocking client stub
- `org.acme.orders.OrderQueryGrpc.OrderQueryImplBase` - base class for `StreamObserver` style implementation
- Message classes such as `GetOrderRequest` and `OrderReply`

## Implement a service with `@GrpcService`

Prefer the generated Mutiny service interface:

```java
import io.quarkus.grpc.GrpcService;
import io.smallrye.mutiny.Uni;
import org.acme.orders.GetOrderRequest;
import org.acme.orders.OrderQuery;
import org.acme.orders.OrderReply;

@GrpcService
public class OrderQueryService implements OrderQuery {
    @Override
    public Uni<OrderReply> getOrder(GetOrderRequest request) {
        return Uni.createFrom().item(() ->
                OrderReply.newBuilder()
                        .setId(request.getId())
                        .setStatus("READY")
                        .setTotalCents(2599)
                        .build());
    }
}
```

`@GrpcService` beans are automatically exposed as gRPC services and should not add extra CDI qualifiers.

## Blocking work inside a service

Service methods run on the event loop by default. Mark truly blocking work explicitly:

```java
import io.smallrye.common.annotation.Blocking;

@Override
@Blocking
public Uni<OrderReply> getOrder(GetOrderRequest request) {
    Order order = repository.findById(request.getId());
    return Uni.createFrom().item(() -> map(order));
}
```

Use `@Blocking` for JDBC, filesystem, or remote clients that block the calling thread.

## Inject a client with `@GrpcClient`

Inject the generated service interface when you want a Mutiny-friendly boundary:

```java
import io.quarkus.grpc.GrpcClient;
import io.smallrye.mutiny.Uni;
import jakarta.enterprise.context.ApplicationScoped;
import org.acme.orders.GetOrderRequest;
import org.acme.orders.OrderQuery;

@ApplicationScoped
public class ShippingGateway {
    @GrpcClient("orders")
    OrderQuery orders;

    public Uni<String> fetchStatus(String id) {
        return orders.getOrder(GetOrderRequest.newBuilder().setId(id).build())
                .onItem().transform(reply -> reply.getStatus());
    }
}
```

If `@GrpcClient` has no value, Quarkus uses the field name as the client name.

## Use generated stubs directly

Inject a blocking stub at imperative edges only:

```java
import io.quarkus.grpc.GrpcClient;
import jakarta.enterprise.context.ApplicationScoped;
import org.acme.orders.GetOrderRequest;
import org.acme.orders.OrderQueryGrpc;

@ApplicationScoped
public class LegacyAdapter {
    @GrpcClient("orders")
    OrderQueryGrpc.OrderQueryBlockingStub orders;

    public String fetchStatus(String id) {
        return orders.getOrder(GetOrderRequest.newBuilder().setId(id).build())
                .getStatus();
    }
}
```

You can also inject `MutinyOrderQueryGrpc.MutinyOrderQueryStub` or `io.grpc.Channel` when you need lower-level stub customization.

## Streaming shapes

Unary request, unary response:

```proto
rpc GetOrder (GetOrderRequest) returns (OrderReply) {}
```

Server streaming:

```java
import io.smallrye.mutiny.Multi;

@Override
public Multi<OrderReply> watchOrders(WatchOrdersRequest request) {
    return orderEvents.streamForCustomer(request.getCustomerId())
            .onItem().transform(this::map);
}
```

Client streaming:

```proto
rpc ImportOrders (stream ImportOrderRequest) returns (ImportSummary) {}
```

```java
import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;

@Override
public Uni<ImportSummary> importOrders(Multi<ImportOrderRequest> request) {
    return request
            .onItem().transform(this::toCommand)
            .collect().asList()
            .onItem().transform(this::runImport);
}
```

Bidirectional streaming:

```proto
rpc SyncOrders (stream SyncRequest) returns (stream SyncReply) {}
```

```java
@Override
public Multi<SyncReply> syncOrders(Multi<SyncRequest> request) {
    return request.onItem().transform(this::handleSync);
}
```

Choose streaming only when the interaction is naturally incremental; unary RPC stays simpler for most service calls.

## `StreamObserver` style implementation

Still useful when integrating code written against standard gRPC Java APIs:

```java
import io.grpc.stub.StreamObserver;
import io.quarkus.grpc.GrpcService;

@GrpcService
public class OrderQueryObserverService extends OrderQueryGrpc.OrderQueryImplBase {
    @Override
    public void getOrder(GetOrderRequest request, StreamObserver<OrderReply> observer) {
        observer.onNext(OrderReply.newBuilder().setId(request.getId()).setStatus("READY").build());
        observer.onCompleted();
    }
}
```

Prefer Mutiny for new code; `StreamObserver` is mainly an interop escape hatch.

## Deadlines

Set a client-wide deadline in configuration when every call to a dependency should fail fast:

```properties
quarkus.grpc.clients.orders.deadline=2s
```

Use a short deadline for synchronous internal calls so slow dependencies fail predictably instead of tying up threads and connections.

## Metadata

Attach headers when you need auth, tenant, or trace propagation at the gRPC layer:

```java
import io.grpc.Metadata;
import io.grpc.stub.MetadataUtils;

Metadata metadata = new Metadata();
Metadata.Key<String> tenantKey = Metadata.Key.of("tenant-id", Metadata.ASCII_STRING_MARSHALLER);
metadata.put(tenantKey, tenantId);

OrderQueryGrpc.OrderQueryBlockingStub stubWithHeaders =
        MetadataUtils.attachHeaders(orders, metadata);
```

For reusable behavior across many calls, prefer client interceptors over ad hoc per-call header code.

## Interceptors

Apply cross-cutting behavior on the server:

```java
import io.grpc.ServerInterceptor;
import io.quarkus.grpc.GlobalInterceptor;
import jakarta.enterprise.context.ApplicationScoped;

@GlobalInterceptor
@ApplicationScoped
public class AuditInterceptor implements ServerInterceptor {
}
```

Register a client-specific interceptor:

```java
import io.quarkus.grpc.RegisterClientInterceptor;

@GrpcClient("orders")
@RegisterClientInterceptor(TenantHeaderInterceptor.class)
OrderQuery orders;
```

Use interceptors for auth, metadata propagation, logging, and deadline customization that should stay outside business code.

## Reflection and health

- gRPC reflection is enabled by default in dev mode and helps tools such as `grpcurl` inspect services.
- Quarkus also exposes standard gRPC health endpoints for implemented services.
