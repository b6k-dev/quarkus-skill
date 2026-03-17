# Quarkus Vert.x Event Bus Configuration Reference

Use this file for the transport, clustering, and execution settings most relevant to Event Bus usage.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.vertx.cluster.clustered` | `false` | Enable Vert.x clustering so the event bus can communicate across nodes |
| `quarkus.vertx.cluster.host` | `localhost` | Bind the cluster manager to a specific host |
| `quarkus.vertx.cluster.port` | implementation-specific | Pin the cluster port instead of using defaults |
| `quarkus.vertx.cluster.public-host` | - | Advertise a reachable public host when bind and advertised addresses differ |
| `quarkus.vertx.cluster.public-port` | - | Advertise a reachable public port |
| `quarkus.vertx.cluster.ping-interval` | `20S` | Tune cluster heartbeat traffic |
| `quarkus.vertx.cluster.ping-reply-interval` | `20S` | Tune heartbeat response timing |
| `quarkus.vertx.eventbus.connect-timeout` | `60S` | Adjust how long remote event-bus connections may take |
| `quarkus.vertx.eventbus.reconnect-attempts` | `0` | Retry remote event-bus reconnection instead of failing immediately |
| `quarkus.vertx.eventbus.reconnect-interval` | `1S` | Control retry interval for reconnect attempts |
| `quarkus.vertx.eventbus.ssl` | `false` | Enable SSL for clustered event-bus transport |
| `quarkus.vertx.eventbus.client-auth` | `NONE` | Require client certificates for clustered transport |
| `quarkus.vertx.eventbus.trust-all` | `false` | Trust all peer certificates in non-production scenarios |
| `quarkus.vertx.event-loops-pool-size` | CPU-count based | Tune event-loop parallelism |
| `quarkus.vertx.max-event-loop-execute-time` | `2S` | Detect handlers that block the event loop too long |
| `quarkus.vertx.warning-exception-time` | `2S` | Control when blocked-event-loop warnings appear |

## Cluster enablement

Minimal clustered setup:

```properties
quarkus.vertx.cluster.clustered=true
quarkus.vertx.cluster.host=0.0.0.0
quarkus.vertx.cluster.public-host=app-1.internal
quarkus.vertx.cluster.port=15701
```

Clustering helps local address-based communication span nodes, but it still does not turn the event bus into a durable broker.

## Reconnect behavior

```properties
quarkus.vertx.eventbus.reconnect-attempts=10
quarkus.vertx.eventbus.reconnect-interval=2S
quarkus.vertx.eventbus.connect-timeout=10S
```

Use reconnect settings only for clustered transport issues. They do not add message durability.

## SSL families

Choose one certificate format family for key and trust material:

- PEM: `quarkus.vertx.eventbus.key-certificate-pem.*`, `quarkus.vertx.eventbus.trust-certificate-pem.*`
- JKS: `quarkus.vertx.eventbus.key-certificate-jks.*`, `quarkus.vertx.eventbus.trust-certificate-jks.*`
- PFX: `quarkus.vertx.eventbus.key-certificate-pfx.*`, `quarkus.vertx.eventbus.trust-certificate-pfx.*`

Example with JKS:

```properties
quarkus.vertx.eventbus.ssl=true
quarkus.vertx.eventbus.client-auth=REQUIRED
quarkus.vertx.eventbus.key-certificate-jks.path=eventbus-keystore.jks
quarkus.vertx.eventbus.key-certificate-jks.password=secret
quarkus.vertx.eventbus.trust-certificate-jks.path=eventbus-truststore.jks
quarkus.vertx.eventbus.trust-certificate-jks.password=secret
```

## Event-loop safety signals

```properties
quarkus.vertx.max-event-loop-execute-time=500ms
quarkus.vertx.warning-exception-time=500ms
```

Use these warnings to catch handlers that should move to `@Blocking` or other worker execution.
