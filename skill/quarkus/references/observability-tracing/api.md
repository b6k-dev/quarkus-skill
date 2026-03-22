# Quarkus Observability Tracing API Reference

Use this file for runtime tracing APIs with short, copy-ready examples.

## Extension entry point

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-opentelemetry</artifactId>
</dependency>
```

Tracing is on by default once the extension is present.

## Automatic server span

```java
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/hello")
class GreetingResource {
    @GET
    String hello() {
        return "hello";
    }
}
```

Requests to supported HTTP endpoints are traced automatically; no tracing annotation is required.

## Create a span on a CDI method

```java
import io.opentelemetry.instrumentation.annotations.SpanAttribute;
import io.opentelemetry.instrumentation.annotations.WithSpan;
import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
class CheckoutService {
    @WithSpan("checkout.calculate")
    void calculate(@SpanAttribute("cart.id") String cartId) {
    }
}
```

Use `@WithSpan` on internal CDI methods, not on already-instrumented JAX-RS endpoints.

## Add attributes to the current span

```java
import io.opentelemetry.instrumentation.annotations.AddingSpanAttributes;
import io.opentelemetry.instrumentation.annotations.SpanAttribute;

@AddingSpanAttributes
void enrich(@SpanAttribute("tenant.id") String tenantId) {
}
```

This reuses the current span instead of creating a child span.

## Inject tracing primitives

```java
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import jakarta.inject.Inject;

class TraceSupport {
    @Inject
    OpenTelemetry openTelemetry;

    @Inject
    Tracer tracer;

    Span current() {
        return Span.current();
    }
}
```

Common CDI injections are `OpenTelemetry`, `Tracer`, `Span`, and `Baggage`.

## Manual child span

```java
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.StatusCode;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Scope;
import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
class PaymentService {
    private final Tracer tracer;

    PaymentService(Tracer tracer) {
        this.tracer = tracer;
    }

    String charge() {
        Span span = tracer.spanBuilder("payment.charge").startSpan();
        try (Scope ignored = span.makeCurrent()) {
            span.setAttribute("payment.provider", "stripe");
            return "ok";
        } catch (RuntimeException e) {
            span.setStatus(StatusCode.ERROR);
            span.recordException(e);
            throw e;
        } finally {
            span.end();
        }
    }
}
```

Always end spans and make the span current around the work being measured.

## Manual propagation with custom carriers

```java
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.context.Context;
import io.opentelemetry.context.propagation.TextMapSetter;
import jakarta.inject.Inject;
import java.util.HashMap;
import java.util.Map;

class HeaderPropagation {
    private static final TextMapSetter<Map<String, String>> SETTER = Map::put;

    @Inject
    OpenTelemetry openTelemetry;

    Map<String, String> headers() {
        Map<String, String> carrier = new HashMap<>();
        openTelemetry.getPropagators().getTextMapPropagator()
                .inject(Context.current(), carrier, SETTER);
        return carrier;
    }
}
```

Use this only when the client or transport does not already propagate trace context.

## Mutiny pipeline span

```java
import static io.quarkus.opentelemetry.runtime.tracing.mutiny.MutinyTracingHelper.wrapWithSpan;

import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Context;
import io.smallrye.mutiny.Uni;
import jakarta.inject.Inject;
import java.util.Optional;

class ReactiveWork {
    @Inject
    Tracer tracer;

    Uni<String> traced() {
        Context context = Context.current();
        return Uni.createFrom().item("hello")
                .transformToUni(item -> wrapWithSpan(tracer, Optional.of(context), "pipeline.step",
                        Uni.createFrom().item(item + " world")));
    }
}
```

## Messaging propagation

```java
import io.opentelemetry.context.Context;
import io.quarkus.messaging.metadata.TracingMetadata;
import org.eclipse.microprofile.reactive.messaging.Message;

Message<String> outgoing(String payload) {
    return Message.of(payload)
            .withMetadata(TracingMetadata.withPrevious(Context.current()));
}
```

Use this when sending messages through Quarkus Messaging and you need downstream spans linked to the current trace.

## OpenTracing shim during migration

```java
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.opentracingshim.OpenTracingShim;
import io.opentracing.Tracer;
import jakarta.inject.Inject;

class LegacyBridge {
    @Inject
    OpenTelemetry openTelemetry;

    Tracer legacyTracer() {
        return OpenTracingShim.createTracerShim(openTelemetry);
    }
}
```

Prefer native OpenTelemetry APIs for new code.
