# Quarkus Messaging Configuration Reference

Use this file for high-value Reactive Messaging properties and channel wiring rules.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `mp.messaging.incoming.<channel>.connector` | auto-selected if only one connector is present | Bind an incoming channel to a specific connector |
| `mp.messaging.outgoing.<channel>.connector` | auto-selected if only one connector is present | Bind an outgoing channel to a specific connector |
| `mp.messaging.incoming.<channel>.<connector-attr>` | connector-specific | Configure broker-specific inbound settings such as `topic`, `address`, deserializers, or consumer group |
| `mp.messaging.outgoing.<channel>.<connector-attr>` | connector-specific | Configure broker-specific outbound settings such as `topic`, serializers, keys, or acknowledgements |
| `mp.messaging.connector.<connector>.<attr>` | connector-specific | Apply connector-wide defaults such as Kafka bootstrap servers |
| `mp.messaging.incoming.<channel>.enabled` | `true` | Disable a channel for a profile or build slice |
| `mp.messaging.incoming.<channel>.concurrency` | `1` | Increase parallel consumption when the connector supports it |
| `mp.messaging.incoming.<channel>.pausable` | `false` | Allow runtime pause/resume through `ChannelRegistry` |
| `mp.messaging.incoming.<channel>.tracing-enabled` | connector-specific | Turn OpenTelemetry propagation off for a noisy or special-case channel |
| `mp.messaging.incoming.<channel>.tls-configuration-name` | - | Attach a named Quarkus TLS configuration to one channel |
| `mp.messaging.connector.<connector>.tls-configuration-name` | - | Attach the same TLS configuration to all channels of a connector |
| `quarkus.messaging.connector-context-propagation` | none | Explicitly propagate selected contexts such as `CDI` through connectors |
| `quarkus.messaging.request-scoped.enabled` | `false` | Activate request scope for each incoming message |
| `quarkus.messaging.health.enabled` | `true` | Disable all messaging health checks |
| `quarkus.messaging.health.<channel>.enabled` | `true` | Disable health checks for one channel |
| `quarkus.messaging.health.<channel>.liveness.enabled` | `true` | Disable only one health-check type for a channel |
| `smallrye.messaging.observation.enabled` | `false` | Enable per-channel Micrometer observation metrics |

## Channel naming and prefixes

Channel properties always use one of these prefixes:

```properties
mp.messaging.incoming.orders.connector=smallrye-kafka
mp.messaging.outgoing.shipments.connector=smallrye-kafka
```

The channel name in config must match the annotation value exactly.

## Example: Kafka channel configuration

```properties
mp.messaging.connector.smallrye-kafka.bootstrap.servers=localhost:9092

mp.messaging.incoming.orders.connector=smallrye-kafka
mp.messaging.incoming.orders.topic=orders
mp.messaging.incoming.orders.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
mp.messaging.incoming.orders.auto.offset.reset=earliest

mp.messaging.outgoing.shipments.connector=smallrye-kafka
mp.messaging.outgoing.shipments.topic=shipments
mp.messaging.outgoing.shipments.value.serializer=org.apache.kafka.common.serialization.StringSerializer
```

The actual destination property varies by connector. Kafka commonly uses `topic`; other connectors may use properties such as `address` or `queue`.

## Profile-based channel disablement

Disable a channel only when the bean graph still makes sense without it:

```properties
%test.mp.messaging.incoming.orders.enabled=false
```

Pair disabled channels with build-time bean filtering when necessary:

```java
@ApplicationScoped
@IfBuildProfile("prod")
class OrderIngestion {
    @Incoming("orders")
    void consume(String payload) {
    }
}
```

## Concurrency

Increase inbound concurrency only when the connector and workload support it:

```properties
mp.messaging.incoming.orders.concurrency=4
```

This creates multiple logical copies of the incoming channel and can improve throughput, especially on partitioned brokers such as Kafka.

## TLS integration

Use the Quarkus TLS registry instead of connector-specific truststore flags when the connector supports it:

```properties
quarkus.tls.messaging.trust-store=truststore.jks
quarkus.tls.messaging.trust-store-password=secret

mp.messaging.incoming.orders.tls-configuration-name=messaging
```

## Context and health controls

```properties
quarkus.messaging.connector-context-propagation=CDI
quarkus.messaging.request-scoped.enabled=true
quarkus.messaging.health.enabled=true
smallrye.messaging.observation.enabled=true
```

Leave connector context propagation off unless you have a clear need; broad propagation can hide lifecycle mistakes.
