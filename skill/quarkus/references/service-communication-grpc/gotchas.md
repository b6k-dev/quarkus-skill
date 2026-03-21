# Quarkus gRPC Gotchas

Common gRPC pitfalls, symptoms, and fixes.

## Proto generation and contract setup

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Generated service or message classes are missing | `.proto` files are not under `src/main/proto`, or custom proto directory was not configured in the build | Put proto files under the default directory or set `quarkus.grpc.codegen.proto-directory` in the build plugin |
| Client injection fails for shared contracts from another artifact | The project depends on generated classes only, or dependency proto scanning is not configured | Prefer sharing proto files or configure `quarkus.generate-code.grpc.scan-for-proto` for contract dependencies |
| Build fails after adding deprecated proto options | `java_generic_services` was enabled | Remove `option java_generic_services = true;` |

## Naming and wiring

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `@GrpcClient` uses the wrong host or port | Client name in code does not match the configured client name | Make `@GrpcClient("name")` match `quarkus.grpc.clients.name.*`, or rely on the exact field name |
| Generated type names are surprising | Proto `service`, `package`, or `java_package` names changed | Verify generated names from the proto contract before wiring implementations or stubs |
| Service works in dev but another service cannot call it | Server is bound to the wrong port or listener mode | Confirm `use-separate-server`, host, and port settings on both client and server |

## Threading and reactive misuse

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Event-loop blocked warnings appear under load | Blocking I/O runs inside a gRPC method without `@Blocking` | Mark blocking methods with `@Blocking` or switch to non-blocking dependencies |
| Throughput collapses when calling gRPC from reactive code | Blocking stub is used on an event-loop path | Use the Mutiny service interface or Mutiny stub for reactive flows |
| Streaming code never completes or leaks resources | `Multi` pipelines do not terminate correctly or upstream is unbounded | Define stream completion rules clearly and test cancellation behavior |

## TLS and transport config

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Handshake failures or connection refusals | One side uses TLS while the other still uses plaintext | Align `plain-text`, trust material, and server certificate configuration on both sides |
| TLS is configured but traffic still arrives on the HTTP server port unexpectedly | Unified server mode is enabled | Remember that `use-separate-server=false` moves gRPC onto the Quarkus HTTP server |
| Local setup works only with `trust-all=true` | Certificates or hostname verification are misconfigured | Fix trust chain and hostname settings instead of keeping insecure trust-all config |

## Message sizing and metadata

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Large requests or responses fail | Inbound message size limits are too small | Increase `max-inbound-message-size` on the receiving side |
| Auth or tenant headers are dropped or rejected | Metadata exceeds limits or uses inconsistent keys | Standardize metadata keys and raise metadata size limits only when needed |
| Calls fail too early with `DEADLINE_EXCEEDED` | Global client deadline is too aggressive for the real latency budget | Revisit `quarkus.grpc.clients.<name>.deadline` per dependency |

## Tooling, reflection, and native

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `grpcurl` works in dev but not in test or production | Reflection service is only enabled by default in dev mode | Set `quarkus.grpc.server.enable-reflection-service=true` where needed |
| Shared generated classes from another module are not discovered correctly | Dependency metadata indexing is missing | Ensure the dependency is indexed when you ship generated classes rather than proto sources |
| Native build or runtime misses non-class resources used by custom gRPC features | Required resources were not included in the native image | Add the necessary native resource configuration and test the native executable explicitly |
