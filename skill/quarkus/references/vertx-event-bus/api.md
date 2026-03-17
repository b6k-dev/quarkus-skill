# Quarkus Vert.x Event Bus API Reference

Use this file for event-bus APIs and compact, copy-ready examples.

## Extension

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-vertx</artifactId>
</dependency>
```

## Declarative consumers with `@ConsumeEvent`

```java
@ApplicationScoped
class GreetingService {
    @ConsumeEvent("greeting")
    String greet(String name) {
        return "Hello " + name;
    }
}
```

If no address is provided, the fully qualified bean class name is used.

## Inject and use the `EventBus`

```java
@Path("/bus")
@ApplicationScoped
class GreetingResource {
    @Inject
    EventBus bus;

    @GET
    @Path("/{name}")
    Uni<String> greet(String name) {
        return bus.<String>request("greeting", name)
                .onItem().transform(Message::body);
    }
}
```

Core sending styles:

```java
bus.send("jobs", job);
bus.publish("notifications", notice);
Uni<String> response = bus.<String>request("greeting", "quarkus")
        .onItem().transform(Message::body);
```

- `send` -> one consumer receives the message
- `publish` -> all consumers on the address receive the message
- `request` -> expect a reply

## Consume the full message

Use Vert.x `Message<T>` when you need headers, address, or manual reply access:

```java
@ConsumeEvent("jobs")
void consume(Message<Job> message) {
    Job job = message.body();
}
```

## Async and blocking consumers

Async handlers can return `Uni<T>` or `CompletionStage<T>`:

```java
@ConsumeEvent("greeting")
Uni<String> greetAsync(String name) {
    return Uni.createFrom().item(() -> name.toUpperCase());
}
```

Blocking handlers must move off the event loop:

```java
@ConsumeEvent("reports")
@Blocking
void generateReport(String id) {
}
```

Equivalent annotation attribute:

```java
@ConsumeEvent(value = "reports", blocking = true)
void generateReport(String id) {
}
```

## Virtual-thread consumers

```java
@ConsumeEvent("imports")
@RunOnVirtualThread
void importBatch(String payload) {
}
```

Use this only with blocking signatures such as `void` or a plain return type. `Uni` and `CompletionStage` return types do not run on virtual threads.

## Config-driven addresses

```java
@ConsumeEvent("${app.events.greeting-address:greeting}")
String greet(String name) {
    return name.toUpperCase();
}
```

If the property is missing and no default value is provided, startup fails.

## Codecs

Quarkus provides a default codec for local delivery. For explicit codecs:

```java
bus.request("greeting", new MyName("quarkus"),
        new DeliveryOptions().setCodecName(MyNameCodec.class.getName()));

@ConsumeEvent(value = "greeting", codec = MyNameCodec.class)
String greet(MyName name) {
    return "Hello " + name.value();
}
```
