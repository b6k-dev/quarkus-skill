# Quarkus Service Communication REST Configuration Reference

Use this file for high-value outbound REST client settings.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.rest-client.<client>.url` | - | A client needs a configured base URL |
| `quarkus.rest-client.<client>.scope` | not set | The client bean needs a different CDI scope |
| `quarkus.rest-client.<client>.connect-timeout` | implementation default | Slow or unreliable networks require faster connect failure |
| `quarkus.rest-client.<client>.read-timeout` | implementation default | Calls must fail if the downstream stalls |
| `quarkus.rest-client.<client>.follow-redirects` | `false` | The downstream uses HTTP redirects intentionally |
| `quarkus.rest-client.<client>.max-redirects` | implementation default | Redirect hops should be bounded explicitly |
| `quarkus.rest-client.<client>.proxy-address` | - | Calls must go through an HTTP proxy |
| `quarkus.rest-client.<client>.verify-host` | `true` | Hostname verification must be tuned for TLS environments |
| `quarkus.rest-client.<client>.trust-store` | - | The downstream uses a custom CA or internal PKI |
| `quarkus.rest-client.<client>.key-store` | - | Mutual TLS client certificates are required |
| `quarkus.rest-client.<client>.connection-pool-size` | implementation default | High concurrency needs explicit pool tuning |
| `quarkus.rest-client.<client>.query-param-style` | implementation default | Multi-value query parameters must match a downstream convention |
| `quarkus.rest-client.logging.scope` | `none` | Request and response logging is needed during troubleshooting |
| `quarkus.rest-client.logging.body-limit` | implementation default | Logging should include more or less of the payload body |

`<client>` can be a `configKey` such as `inventory-api` or a fully qualified interface name.

## Base URL with configKey

```java
@RegisterRestClient(configKey = "inventory-api")
public interface InventoryClient {
}
```

```properties
quarkus.rest-client.inventory-api.url=http://inventory.internal:8080
```

Prefer `configKey` for stable property names across refactors.

## Timeouts and redirects

```properties
quarkus.rest-client.inventory-api.connect-timeout=2S
quarkus.rest-client.inventory-api.read-timeout=5S
quarkus.rest-client.inventory-api.follow-redirects=true
quarkus.rest-client.inventory-api.max-redirects=3
```

Set both timeouts intentionally for external dependencies; default values are easy to forget.

## Scope and per-profile endpoints

```properties
quarkus.rest-client.billing-api.scope=jakarta.inject.Singleton
%dev.quarkus.rest-client.billing-api.url=http://localhost:8089
%prod.quarkus.rest-client.billing-api.url=https://billing.internal
```

Use profiles to keep local and production targets separate. Set `scope` only when you intentionally need something other than the default client bean behavior.

## TLS and mutual TLS

```properties
quarkus.rest-client.partner-api.url=https://partner.internal
quarkus.rest-client.partner-api.trust-store=client-truststore.p12
quarkus.rest-client.partner-api.trust-store-password=changeit
quarkus.rest-client.partner-api.trust-store-type=PKCS12
quarkus.rest-client.partner-api.key-store=client-keystore.p12
quarkus.rest-client.partner-api.key-store-password=changeit
quarkus.rest-client.partner-api.key-store-type=PKCS12
quarkus.rest-client.partner-api.verify-host=true
```

Use trust stores for private CAs and key stores only when the downstream requires client certificates.

## Proxy and pool tuning

```properties
quarkus.rest-client.catalog-api.proxy-address=proxy.internal:8080
quarkus.rest-client.catalog-api.connection-pool-size=50
```

Tune the pool only for measured concurrency needs; oversizing can waste connections and memory.

## Query parameter style

```properties
quarkus.rest-client.search-api.query-param-style=multi-pairs
```

Use this when the downstream expects repeated keys, comma-separated values, or another non-default encoding style.

## Diagnostics logging

```properties
quarkus.rest-client.logging.scope=request-response
quarkus.rest-client.logging.body-limit=1024
quarkus.log.category."org.jboss.resteasy.reactive.client.logging".level=DEBUG
```

Enable detailed logging selectively; body logging can expose secrets and large payloads.

## See Also

- [../configuration/README.md](../configuration/README.md) - Property layering, profiles, and config sources
