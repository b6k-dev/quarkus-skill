# Quarkus Vert.x Event Bus Usage Patterns

Use these patterns for repeatable local asynchronous messaging workflows.

## Pattern: Bridge HTTP to Request/Reply Messaging

When to use:

- A REST endpoint delegates work to another bean asynchronously and waits for a reply.

Example:

```java
@Path("/async")
@ApplicationScoped
class EventResource {
    @Inject
    EventBus bus;

    @GET
    @Path("/{name}")
    Uni<String> greeting(String name) {
        return bus.<String>request("greeting", name)
                .onItem().transform(Message::body);
    }
}

@ApplicationScoped
class GreetingService {
    @ConsumeEvent("greeting")
    String greeting(String name) {
        return "Hello " + name;
    }
}
```

Use this when the caller needs a direct response but you still want loose coupling inside the app.

## Pattern: Publish to All Local Consumers

When to use:

- One event should notify several local listeners.

Example:

```java
@ApplicationScoped
class NotificationPublisher {
    @Inject
    EventBus bus;

    void publish(Notification notice) {
        bus.publish("notifications", notice);
    }
}
```

Use `publish`, not `send`, when every consumer on the address should receive the message.

## Pattern: Offload Blocking Work from the Event Loop

When to use:

- A consumer calls a blocking library or performs synchronous I/O.

Example:

```java
@ApplicationScoped
class PdfConsumer {
    @ConsumeEvent("pdf-jobs")
    @Blocking
    void render(String jobId) {
        renderer.render(jobId);
    }
}
```

Prefer a normal non-blocking handler until you know blocking work is unavoidable.

## Pattern: Externalize Event Bus Addresses

When to use:

- Address names differ across environments or should stay configurable.

Example:

```java
@ApplicationScoped
class GreetingService {
    @ConsumeEvent("${app.bus.greeting-address:greeting}")
    String greeting(String name) {
        return name.toUpperCase();
    }
}
```

This keeps address literals out of code while preserving a safe default.

## Pattern: Exchange Rich Local Types with a Custom Codec

When to use:

- The local payload is not covered by the default codec or must use explicit serialization rules.

Example:

```java
DeliveryOptions options = new DeliveryOptions().setCodecName(MyCommandCodec.class.getName());
bus.request("commands", new MyCommand("sync"), options);

@ConsumeEvent(value = "commands", codec = MyCommandCodec.class)
String handle(MyCommand command) {
    return command.name();
}
```

Apply the same codec choice on both sending and receiving sides.
