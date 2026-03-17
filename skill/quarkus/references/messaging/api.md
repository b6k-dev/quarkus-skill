# Quarkus Messaging API Reference

Use this file for Reactive Messaging APIs and compact, copy-ready examples.

## Common extensions

Choose the connector that matches the broker:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-messaging-kafka</artifactId>
</dependency>
```

Other common choices are `quarkus-messaging-rabbitmq`, `quarkus-messaging-amqp`, and `quarkus-messaging-pulsar`.

## Core channel annotations

Transform a stream from one channel to another:

```java
@ApplicationScoped
class PriceProcessor {
    @Incoming("prices-in")
    @Outgoing("prices-out")
    String normalize(String payload) {
        return payload.trim().toUpperCase();
    }
}
```

Consume only:

```java
@ApplicationScoped
class AuditConsumer {
    @Incoming("audit")
    void onMessage(String payload) {
    }
}
```

Produce only:

```java
@ApplicationScoped
class SeedProducer {
    @Outgoing("seed")
    Multi<String> seed() {
        return Multi.createFrom().items("a", "b", "c");
    }
}
```

Do not call `@Incoming` or `@Outgoing` methods directly from user code; Quarkus invokes them.

## Imperative publishing with `@Channel`

Send from REST, scheduled jobs, or other imperative entry points:

```java
@Path("/orders")
@ApplicationScoped
class OrderResource {
    @Channel("orders-out")
    Emitter<OrderPlaced> emitter;

    @POST
    CompletionStage<Void> create(OrderPlaced order) {
        return emitter.send(order);
    }
}
```

Use `MutinyEmitter<T>` when you want Mutiny-style APIs such as `sendAndAwait`.

## Inject channel streams

Expose a channel as a reactive stream:

```java
@Path("/prices")
@ApplicationScoped
class PriceStreamResource {
    @Channel("prices")
    Multi<Double> prices;

    @GET
    Multi<Double> stream() {
        return prices;
    }
}
```

When injecting `Multi<T>` with `@Channel`, your code is responsible for subscribing to it.

## Messages and metadata

Use `Message<T>` when you need the envelope rather than only the payload:

```java
@Incoming("orders-in")
@Outgoing("orders-out")
Message<String> process(Message<String> message) {
    return message.withPayload(message.getPayload().toUpperCase());
}
```

Connector metadata can be injected directly as a parameter after the payload:

```java
@Incoming("orders-in")
String process(String payload, MyConnectorMetadata metadata) {
    return payload + ":" + metadata.partition();
}
```

## Fan-out and fan-in

Broadcast to multiple downstream consumers:

```java
@Incoming("raw")
@Outgoing("enriched")
@Broadcast
String enrich(String payload) {
    return payload.toUpperCase();
}
```

Allow multiple producers into one channel:

```java
@Incoming("a")
@Outgoing("merged")
String fromA(String payload) {
    return payload;
}

@Incoming("b")
@Outgoing("merged")
String fromB(String payload) {
    return payload;
}

@Incoming("merged")
@Merge
void consume(String payload) {
}
```

## Stream processing signatures

Operate on the whole stream when per-message methods are too limiting:

```java
@Incoming("source")
@Outgoing("sink")
Multi<String> process(Multi<String> input) {
    return input.map(String::toUpperCase);
}
```

## Execution control

Quarkus chooses event-loop vs worker execution from the method signature, but you can override it:

```java
@Incoming("jobs")
@Blocking
void runBlocking(Job job) {
}

@Incoming("events")
@NonBlocking
Uni<Void> runAsync(Event event) {
    return Uni.createFrom().voidItem();
}

@Incoming("imports")
@RunOnVirtualThread
void importBatch(Batch batch) {
}
```

`@Transactional` implies blocking execution.

## Pausable channels

Control an incoming channel at runtime:

```java
@ApplicationScoped
class ChannelController {
    @Inject
    ChannelRegistry registry;

    void pause() {
        registry.getPausable("orders").pause();
    }

    void resume() {
        registry.getPausable("orders").resume();
    }
}
```

Enable pausability in configuration before using `ChannelRegistry#getPausable`.
