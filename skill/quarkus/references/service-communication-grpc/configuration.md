# Quarkus gRPC Configuration Reference

Use this file for high-value gRPC server, client, TLS, transport, and code generation settings.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.grpc.server.use-separate-server` | `true` | You want a dedicated gRPC server instead of the shared Quarkus HTTP server |
| `quarkus.grpc.server.host` | `0.0.0.0` | The server must bind to a specific interface |
| `quarkus.grpc.server.port` | `9000` | The dedicated gRPC server port must change |
| `quarkus.grpc.server.test-port` | `9001` | Tests should use a different dedicated gRPC port |
| `quarkus.grpc.server.plain-text` | `true` | TLS should be enabled or disabled explicitly |
| `quarkus.grpc.server.max-inbound-message-size` | mode-dependent | Requests may exceed the default message size |
| `quarkus.grpc.server.max-inbound-metadata-size` | implementation default | Large metadata headers must be accepted |
| `quarkus.grpc.server.enable-reflection-service` | `false` outside dev | Reflection should stay available in test or production |
| `quarkus.grpc.server.instances` | `1` | Server handling should scale across more event loops |
| `quarkus.grpc.clients.<name>.host` | `localhost` | A client should target a specific service host |
| `quarkus.grpc.clients.<name>.port` | `9000` | A client should target a specific service port |
| `quarkus.grpc.clients.<name>.plain-text` | auto | Client transport mode must be explicit |
| `quarkus.grpc.clients.<name>.deadline` | - | Calls should fail after a bounded duration |
| `quarkus.grpc.clients.<name>.name-resolver` | `dns` | Service discovery should use Stork instead of plain DNS |
| `quarkus.grpc.clients.<name>.keep-alive-time` | - | Long-lived channels need keepalive tuning |
| `quarkus.grpc.clients.<name>.flow-control-window` | `1048576` | Large streamed responses need better throughput |
| `quarkus.grpc.clients.<name>.max-inbound-message-size` | implementation default | Large responses must be accepted |
| `quarkus.grpc.codegen.proto-directory` | `src/main/proto` | Proto files live outside the default build path |
| `quarkus.generate-code.grpc.scan-for-proto` | `none` | Stubs should be generated from proto files in dependencies |
| `quarkus.generate-code.grpc.scan-for-imports` | protobuf imports only | Imported proto files should be scanned from more dependencies |

## Server mode selection

Use the shared Quarkus HTTP server when you want REST and gRPC on the same listener:

```properties
quarkus.grpc.server.use-separate-server=false
quarkus.http.port=8080
```

In unified server mode, most transport settings come from the HTTP server configuration instead of dedicated gRPC server properties.

Use the dedicated gRPC server when you want an isolated port or Netty-specific tuning:

```properties
quarkus.grpc.server.use-separate-server=true
quarkus.grpc.server.host=0.0.0.0
quarkus.grpc.server.port=9000
quarkus.grpc.server.test-port=9001
```

## Client target configuration

Point a named client at another service:

```properties
quarkus.grpc.clients.orders.host=orders.internal
quarkus.grpc.clients.orders.port=9000
quarkus.grpc.clients.orders.plain-text=true
```

The `orders` segment must match the `@GrpcClient("orders")` value, or the field name when no value is provided.

## TLS server configuration

Unified server mode with centralized TLS configuration:

```properties
quarkus.grpc.server.use-separate-server=false
quarkus.grpc.server.plain-text=false

quarkus.tls.key-store.p12.path=certs/grpc-server.p12
quarkus.tls.key-store.p12.password=changeit
quarkus.http.insecure-requests=disabled
```

Dedicated gRPC server with PEM files:

```properties
quarkus.grpc.server.plain-text=false
quarkus.grpc.server.ssl.certificate=tls/server.pem
quarkus.grpc.server.ssl.key=tls/server.key
```

If TLS is enabled, make sure the client and server agree on both encryption and trust material.

## TLS client configuration

Quarkus gRPC client with centralized TLS config:

```properties
quarkus.tls.trust-store.p12.path=certs/grpc-client-truststore.p12
quarkus.tls.trust-store.p12.password=changeit

quarkus.grpc.clients.orders.use-quarkus-grpc-client=true
quarkus.grpc.clients.orders.plain-text=false
```

Quarkus gRPC client with client-local trust material:

```properties
quarkus.grpc.clients.orders.use-quarkus-grpc-client=true
quarkus.grpc.clients.orders.plain-text=false
quarkus.grpc.clients.orders.tls.enabled=true
quarkus.grpc.clients.orders.tls.trust-certificate-pem.certs=certs/ca.crt
quarkus.grpc.clients.orders.tls.verify-hostname=true
```

The `tls.*` client properties shown here apply when `quarkus.grpc.clients.<name>.use-quarkus-grpc-client=true`.

Avoid `tls.trust-all=true` outside short-lived local development.

## Discovery and static endpoints

Static DNS host:

```properties
quarkus.grpc.clients.orders.host=orders.internal
quarkus.grpc.clients.orders.port=9000
quarkus.grpc.clients.orders.name-resolver=dns
```

Stork-backed discovery:

```properties
quarkus.grpc.clients.orders.host=orders-service
quarkus.grpc.clients.orders.name-resolver=stork
quarkus.grpc.clients.orders.stork.deadline=5000
quarkus.grpc.clients.orders.stork.retries=3
```

Use plain host and port for small static environments; use Stork when service instances are dynamic.

## Deadlines and keepalive

```properties
quarkus.grpc.clients.orders.deadline=2s
quarkus.grpc.clients.orders.keep-alive-time=30s
quarkus.grpc.clients.orders.keep-alive-timeout=10s
quarkus.grpc.clients.orders.keep-alive-without-calls=false
```

Prefer conservative keepalive values unless you have confirmed idle channel drops from proxies or load balancers.

## Message size and flow control

```properties
quarkus.grpc.server.max-inbound-message-size=8M
quarkus.grpc.clients.orders.max-inbound-message-size=8M
quarkus.grpc.clients.orders.max-inbound-metadata-size=16K
quarkus.grpc.clients.orders.flow-control-window=10485760
```

Increase these only for known large payloads or streams; oversized limits make accidental misuse harder to notice.

## Local development choices

Random port in tests or local runs:

```properties
quarkus.grpc.server.port=0
quarkus.grpc.server.test-port=0
```

Shared HTTP server in dev when you also expose REST:

```properties
quarkus.grpc.server.use-separate-server=false
quarkus.http.port=8080
```

Reflection outside dev mode:

```properties
quarkus.grpc.server.enable-reflection-service=true
```

## Code generation properties

Custom proto directory:

```xml
<plugin>
    <groupId>${quarkus.platform.group-id}</groupId>
    <artifactId>quarkus-maven-plugin</artifactId>
    <extensions>true</extensions>
    <executions>
        <execution>
            <configuration>
                <properties>
                    <quarkus.grpc.codegen.proto-directory>${project.basedir}/proto/contracts</quarkus.grpc.codegen.proto-directory>
                </properties>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Generate stubs from dependency-provided proto files:

```properties
quarkus.generate-code.grpc.scan-for-proto=com.acme:contracts
quarkus.generate-code.grpc.scan-for-imports=all
```

Descriptor set generation can help tooling that needs schema metadata at runtime:

```properties
quarkus.generate-code.grpc.descriptor-set.generate=true
quarkus.generate-code.grpc.descriptor-set.name=orders.dsc
```
