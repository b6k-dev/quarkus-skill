# Quarkus CDI Events API Reference

Use this file for CDI event APIs and compact, copy-ready examples.

## Fire a local domain event

```java
record OrderPlaced(String id) {
}

@ApplicationScoped
class Orders {
    @Inject
    Event<OrderPlaced> orderPlaced;

    @Transactional
    void place(String id) {
        orderPlaced.fire(new OrderPlaced(id));
    }
}
```

## Observe synchronously

```java
@ApplicationScoped
class AuditLog {
    void onOrder(@Observes OrderPlaced event) {
    }
}
```

`@Observes` runs in-process and synchronously with the firing call.

## Observe asynchronously

```java
@ApplicationScoped
class Notifications {
    void onOrder(@ObservesAsync OrderPlaced event) {
    }
}

@ApplicationScoped
class OrderNotifier {
    @Inject
    Event<OrderPlaced> orderPlaced;

    CompletionStage<OrderPlaced> notifyAsync(String id) {
        return orderPlaced.fireAsync(new OrderPlaced(id));
    }
}
```

`@ObservesAsync` is still in-process. It is async dispatch, not messaging with durability.

## Transactional observers

```java
@ApplicationScoped
class AuditLog {
    void onOrder(@Observes(during = TransactionPhase.AFTER_SUCCESS) OrderPlaced event) {
    }
}
```

Useful phases:

- `IN_PROGRESS` - default, inside the current transaction
- `BEFORE_COMPLETION` - just before commit
- `AFTER_SUCCESS` - only after commit succeeds
- `AFTER_FAILURE` - only after rollback
- `AFTER_COMPLETION` - after transaction completion regardless of result

Prefer `AFTER_SUCCESS` for side effects that should happen only after a successful commit.
