# Quarkus Dependency Injection Usage Patterns (CDI / ArC)

Use these patterns for repeatable dependency injection and lifecycle workflows.

## Pattern: Default Service Wiring with Constructor Injection

When to use:

- Standard service-to-service wiring with required dependencies.

Example:

```java
@ApplicationScoped
class OrderService {
    private final PaymentClient paymentClient;

    OrderService(PaymentClient paymentClient) {
        this.paymentClient = paymentClient;
    }
}
```

Example (when Lombok is available):

```java
import lombok.RequiredArgsConstructor;

@ApplicationScoped
@RequiredArgsConstructor
class OrderService {
    private final PaymentClient paymentClient;
}
```

With a single generated constructor, Quarkus injects dependencies without an explicit `@Inject`.

## Pattern: Multiple Implementations with Type-Safe Qualifiers

When to use:

- More than one bean implements the same interface.

Example:

```java
@Qualifier
@Retention(RUNTIME)
@Target({ TYPE, FIELD, PARAMETER, METHOD })
@interface Fast {}

@Fast
@ApplicationScoped
class FastSearch implements SearchEngine {}

@Inject
@Fast
SearchEngine searchEngine;
```

Prefer `@Identifier("...")` over `@Named("...")` for internal selection.

## Pattern: Eager Startup Initialization

When to use:

- Warm-up, cache priming, startup validation, or one-time boot tasks.

Example:

```java
@ApplicationScoped
class Warmup {
    void onStart(@Observes StartupEvent event) {
        // perform startup work
    }
}
```

Alternative: annotate the bean with `@Startup`.

## Pattern: Conditional Defaults by Profile or Property

When to use:

- Extension-style defaults that application code may override.

Example:

```java
@Dependent
class TracerConfig {
    @Produces
    @IfBuildProfile("prod")
    Tracer realTracer() { return new RealTracer(); }

    @Produces
    @DefaultBean
    Tracer noopTracer() { return new NoopTracer(); }
}
```

## Pattern: Collect All Implementations in Priority Order

When to use:

- Pipeline, strategy chain, plugin list, or ordered processing.

Example:

```java
@Inject
@All
List<Handler> handlers;
```

`@All List<T>` is immutable and sorted by bean priority (highest first).

## Pattern: Shared Mutable State with Container-Managed Locking

When to use:

- `@ApplicationScoped` or `@Singleton` beans with concurrent access.

Example:

```java
@Lock
@ApplicationScoped
class BalanceService {
    void mutate() {}

    @Lock(Lock.Type.READ)
    BigDecimal read() { return BigDecimal.ZERO; }
}
```

Use standard Java concurrency when finer-grained control is needed.

## Pattern: Decorate Business Behavior

When to use:

- Wrap an existing implementation with business-aware behavior (not just cross-cutting interception).

Example:

```java
import jakarta.annotation.Priority;
import jakarta.decorator.Decorator;
import jakarta.decorator.Delegate;
import jakarta.enterprise.inject.Any;
import jakarta.inject.Inject;

interface Notifier {
    void send(String message);
}

@Decorator
@Priority(10)
class NotifierDecorator implements Notifier {
    @Inject
    @Delegate
    @Any
    Notifier delegate;

    @Override
    public void send(String message) {
        delegate.send("[decorated] " + message);
    }
}
```

## Pattern: Intercept External Classes with `InterceptionProxy`

When to use:

- A produced class comes from an external library and cannot be annotated directly.

Example:

```java
import io.quarkus.arc.BindingsSource;
import io.quarkus.arc.InterceptionProxy;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.inject.Produces;

abstract class ExternalApiBindings {
    @Logged
    abstract String call(String input);
}

@ApplicationScoped
class ExternalApiProducer {
    @Produces
    ExternalApiClient client(
            @BindingsSource(ExternalApiBindings.class)
            InterceptionProxy<ExternalApiClient> proxy) {
        return proxy.create(new ExternalApiClient());
    }
}
```
