# Quarkus Web REST Gotchas

Common REST pitfalls, symptoms, and fixes.

## Routing and parameter binding

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Endpoint path is not where expected | `@ApplicationPath`, `quarkus.rest.path`, and `quarkus.http.root-path` are combined unexpectedly | Standardize on one base-path strategy and verify final path composition |
| Path/query parameter binding fails after refactor | Parameter names changed and compiler metadata is missing | Compile with `-parameters` or use explicit annotation values |

## JSON behavior

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Unknown JSON fields are silently accepted | Jackson defaults to ignore unknown properties in Quarkus | Set `quarkus.jackson.fail-on-unknown-properties=true` |
| JSON body is empty in native executable | Serialized type cannot be inferred from raw `Response` return types | Prefer concrete return types or annotate model with `@RegisterForReflection` |
| Extension-provided JSON behavior disappears | Custom `ObjectMapper` or `Jsonb` producer ignored Quarkus customizers | Inject and apply all customizer beans in custom producers |

## Multipart uploads

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Multipart requests fail with HTTP 413 | Part exceeds `quarkus.http.limits.max-form-attribute-size` | Increase the size limit |
| Uploaded file is unavailable after request completion | File remained in temp storage and was cleaned up | Move uploads to durable storage during request handling |

## Threading and filters

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Event-loop blocked warnings or degraded throughput | Blocking code runs on IO thread | Use reactive APIs or `@Blocking` |
| Filter behavior differs between endpoints | Filters follow endpoint threading model | Use `@ServerRequestFilter(nonBlocking = true)` when event-loop pre-processing is required |

## Exception mapping

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Custom mapper for `JsonMappingException` is not invoked | Built-in mapper for subtype (`MismatchedInputException`) takes precedence | Disable selected built-in mapper with `quarkus.rest.exception-mapping.disable-mapper-for` |
| Exception context is missing in logs | Quarkus REST suppresses some exception logs by default | Raise relevant REST log categories to `DEBUG` |
