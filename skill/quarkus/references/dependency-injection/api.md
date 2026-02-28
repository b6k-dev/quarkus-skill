# Quarkus Dependency Injection API Reference (CDI / ArC)

Use this file for DI-focused APIs with compact, copy-ready examples.

## Core model

```java
@ApplicationScoped
class GreetingService {
    String hello(String name) {
        return "Hello " + name;
    }
}

@Path("/hello")
class GreetingResource {
    private final GreetingService service;

    GreetingResource(GreetingService service) {
        this.service = service;
    }

    @GET
    String hello() {
        return service.hello("quarkus");
    }
}
```

## Bean discovery

Class beans need a bean-defining annotation:

```java
@ApplicationScoped
class InventoryService {
}
```

Producer and observer methods are discovered even if declaring class has no scope:

```java
class DiscoverySupport {
    @Produces
    Clock clock() {
        return Clock.systemUTC();
    }

    void onStart(@Observes StartupEvent event) {
    }
}
```

Index dependencies explicitly when needed:

```properties
quarkus.index-dependency.acme.group-id=org.acme
quarkus.index-dependency.acme.artifact-id=acme-api
```

## Injection API

Constructor + field + initializer method injection:

```java
@ApplicationScoped
class CheckoutService {
    @Inject
    InventoryService inventory;

    private final PricingService pricing;

    CheckoutService(PricingService pricing) {
        this.pricing = pricing;
    }

    @Inject
    void init(AuditService audit) {
        audit.register("checkout");
    }
}
```

Programmatic lookup:

```java
@ApplicationScoped
class StrategySelector {
    @Inject
    Instance<PaymentStrategy> strategies;

    PaymentStrategy pickFirst() {
        for (PaymentStrategy strategy : strategies) {
            return strategy;
        }
        throw new IllegalStateException("No strategy available");
    }
}
```

## Qualifiers

Custom qualifier for multiple implementations:

```java
@Qualifier
@Retention(RUNTIME)
@Target({ TYPE, FIELD, PARAMETER, METHOD })
@interface Fast {
}

interface SearchService {
}

@Fast
@ApplicationScoped
class FastSearchService implements SearchService {
}

@ApplicationScoped
class SearchResource {
    @Inject
    @Fast
    SearchService searchService;
}
```

Internal string-based selection with `@Identifier`:

```java
class PaymentProducers {
    @Produces
    @Identifier("stripe")
    PaymentClient stripe() {
        return new StripeClient();
    }
}

class BillingService {
    @Inject
    @Identifier("stripe")
    PaymentClient client;
}
```

## Scopes

```java
@ApplicationScoped
class AppCache {
}

@RequestScoped
class RequestContext {
}

@Dependent
class IdGenerator {
}

@Singleton
class GlobalCounter {
}
```

## Lifecycle callbacks

```java
@ApplicationScoped
class WarmupService {
    @PostConstruct
    void init() {
    }

    void onStart(@Observes StartupEvent event) {
    }

    @PreDestroy
    void shutdown() {
    }
}
```

## Producers

```java
@ApplicationScoped
class ClientProducer {
    @Produces
    @ApplicationScoped
    HttpClient httpClient(@ConfigProperty(name = "remote.timeout-ms") int timeoutMs) {
        return HttpClient.newBuilder()
                .connectTimeout(Duration.ofMillis(timeoutMs))
                .build();
    }
}
```

## Interceptors

```java
@InterceptorBinding
@Retention(RUNTIME)
@Target({ TYPE, METHOD })
@interface Logged {
}

@Logged
@Priority(Interceptor.Priority.APPLICATION)
@Interceptor
class LoggingInterceptor {
    @AroundInvoke
    Object log(InvocationContext ctx) throws Exception {
        return ctx.proceed();
    }
}

@Logged
@ApplicationScoped
class BillingService {
    void bill() {
    }
}
```

For decorators, `@Lock`, and `InterceptionProxy`, see `patterns.md`.

## Events and observers

```java
record OrderPlaced(String id) {
}

@ApplicationScoped
class Orders {
    @Inject
    Event<OrderPlaced> orderPlaced;

    void place(String id) {
        orderPlaced.fire(new OrderPlaced(id));
    }

    CompletionStage<OrderPlaced> placeAsync(String id) {
        return orderPlaced.fireAsync(new OrderPlaced(id));
    }
}

@ApplicationScoped
class AuditLog {
    void onOrder(@Observes OrderPlaced event) {
    }

    void onOrderAsync(@ObservesAsync OrderPlaced event) {
    }
}
```

## Quarkus-specific CDI APIs

Default/conditional wiring:

```java
@Dependent
class TracerProducer {
    @Produces
    @IfBuildProfile("prod")
    Tracer realTracer() {
        return new RealTracer();
    }

    @Produces
    @DefaultBean
    Tracer noopTracer() {
        return new NoopTracer();
    }
}
```

Lookup enhancements:

```java
@ApplicationScoped
class HandlerRunner {
    @Inject
    @All
    List<Handler> handlers;

    @Inject
    @WithCaching
    Instance<Formatter> formatter;
}
```

Method-level interceptor control:

```java
@Logged
@ApplicationScoped
class ReportService {
    void export() {
    }

    @NoClassInterceptors
    void exportWithoutClassInterceptors() {
    }
}
```

## Dev mode diagnostics endpoints

```bash
quarkus dev
curl http://localhost:8080/q/arc
curl http://localhost:8080/q/arc/beans
curl "http://localhost:8080/q/arc/beans?scope=ApplicationScoped"
curl http://localhost:8080/q/arc/removed-beans
curl http://localhost:8080/q/arc/observers
```
