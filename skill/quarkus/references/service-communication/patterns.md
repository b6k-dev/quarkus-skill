# Service Communication Usage Patterns

Use these patterns to choose the simplest transport that fits the service boundary.

## Pattern: Call a standard HTTP service with a typed client

When to use:

- The downstream system already exposes a REST or HTTP API.
- You care about URLs, headers, status codes, and JSON payloads.

Route to `../service-communication-rest/patterns.md`.

## Pattern: Use gRPC for internal strongly typed RPC

When to use:

- Producer and consumer are internal services under coordinated control.
- Shared `.proto` files and streaming are acceptable or desirable.

Route to `../service-communication-grpc/patterns.md`.

## Pattern: Do not use synchronous RPC for integration events

When to use:

- The caller should not wait for the downstream side effect to finish.
- Retries, replay, consumer scaling, or buffer tolerance matter.

Example:

```java
@ApplicationScoped
class OrderService {
    @Channel("orders-out")
    Emitter<OrderPlaced> emitter;

    void submit(OrderPlaced order) {
        emitter.send(order);
    }
}
```

Route to `../messaging/README.md` instead of forcing REST or gRPC into an eventing role.

## Pattern: Keep transport details at the boundary

When to use:

- A service layer should stay focused on domain logic rather than transport mechanics.

Example:

```java
@ApplicationScoped
class ShippingService {
    private final InventoryGateway inventory;

    ShippingService(InventoryGateway inventory) {
        this.inventory = inventory;
    }
}
```

Hide REST clients or gRPC stubs behind a small gateway or adapter when the rest of the code should not care about transport choice.
