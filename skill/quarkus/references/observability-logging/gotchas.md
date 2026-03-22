# Quarkus Observability Logging Gotchas

Common logging pitfalls, symptoms, and fixes.

## Levels and category matching

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `TRACE` or `DEBUG` logs never appear for one category | Runtime level was lowered but `min-level` was not | Set `quarkus.log.min-level` or `quarkus.log.category."...".min-level` low enough at build time |
| Category override seems ignored | Category name is wrong or missing required quotes in config | Use the exact logger category and quote it: `quarkus.log.category."org.hibernate".level=DEBUG` |
| Changing one category also changes child packages | Category config applies recursively | Add a more specific subcategory override when needed |

## Handler attachment and duplication

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Same message appears twice | Category handler is attached but parent handlers are still enabled | Set `quarkus.log.category."...".use-parent-handlers=false` when isolating output |
| Named handler never writes anything | Handler was defined but never attached to root or a category | Add it to `quarkus.log.handlers` or `quarkus.log.category."...".handlers` |
| File logging is configured but no file is created | File handler is disabled | Set `quarkus.log.file.enabled=true` or enable the named file handler |

## Structured JSON output

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Console `format` pattern changes have no effect | JSON logging extension owns that handler formatting | Change `quarkus.log.<handler>.json.*` properties instead of `quarkus.log.<handler>.format` |
| Downstream parser rejects the payload | Pretty printing or record delimiters do not match consumer expectations | Keep pretty printing off unless the receiver explicitly supports it |
| Elasticsearch field mapping errors appear | Dynamic fields from message parameters or MDC vary in type | Keep structured fields stable, or normalize/index them as text/keyword downstream |

## MDC and correlation

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| MDC key is missing from output | Pattern does not reference `%X{key}` or `%X` | Update the text pattern or enable JSON/MDC inclusion where appropriate |
| Context from one request leaks into another | MDC entries are not cleared in imperative code | Use `try/finally` and clear keys after the log-producing scope |
| Trace IDs are missing from logs | Tracing is not enabled, or the log format does not render MDC trace fields | Enable tracing/correlation path and include `%X{traceId}` / `%X{spanId}` in the pattern |

## Centralized shipping

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Logs still go to console after enabling socket or syslog | Console handler remains enabled by design | Disable console if needed, or keep it intentionally for local diagnostics |
| Socket or syslog shipping produces nothing | Handler is disabled or endpoint is wrong | Verify `quarkus.log.socket.enabled` / `quarkus.log.syslog.enabled` and the target endpoint |
| Multiline stack traces are split unexpectedly in downstream tooling | Receiver format does not preserve multiline events well | Prefer JSON shipping with formatted exception fields when the pipeline supports it |

## OpenTelemetry logging

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| No OTel logs are exported | `quarkus.otel.logs.enabled` is false | Enable `quarkus.otel.logs.enabled=true` and rebuild if required |
| OTel logs stop while normal Quarkus logs continue | OTel handler bridge is disabled at runtime | Check `quarkus.otel.logs.handler.enabled` |
| Log export endpoint changes do nothing | Wrong signal-specific OTLP property or wrong protocol/port pairing | Use `quarkus.otel.exporter.otlp.logs.*` and align `grpc` with `4317` or `http/protobuf` with `4318` |
| Setup becomes brittle after version updates | OpenTelemetry logging is still tech preview | Prefer Quarkus-native handlers first; enable OTel log export only when the collector path is a requirement |
