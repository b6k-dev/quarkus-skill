# Observability Logging Reference

Use this module when the task is about Quarkus application logs: log levels, categories, handlers, structured JSON logging, MDC context, centralized shipping, and OpenTelemetry log correlation/export.

## Overview

Quarkus logging is built on JBoss Log Manager and routes supported logging APIs through one runtime logging system.

- Configure root and per-category levels in `application.properties`.
- Attach console, file, syslog, socket, and named handlers as needed.
- Use `quarkus-logging-json` for structured JSON output when downstream tooling expects machine-readable logs.
- Add MDC fields for request or tenant context, and correlate with OpenTelemetry trace IDs when tracing is enabled.
- Treat OpenTelemetry log export as one option, not the default path for all logging setups.

## General guidelines

- Start with root/category level tuning before adding custom handlers.
- Keep human-readable console logs in dev unless structured logs are required there too.
- Prefer console or socket shipping in containers; use file logging only when host-local retention is a requirement.
- Put stable correlation data in MDC; avoid dumping large or high-cardinality payloads into every record.
- Enable OpenTelemetry log export only when you explicitly need OTel log records sent to a collector.

## Quick Routing

1. Logger usage, MDC, filters, and supported logging APIs -> `api.md`
2. High-value root/category/handler/JSON/OTel properties -> `configuration.md`
3. Repeatable setup workflows for structured logs and shipping -> `patterns.md`
4. Missing logs, duplicate handlers, bad JSON output, and exporter surprises -> `gotchas.md`

## In This Reference

- [api.md](./api.md) - Runtime logging APIs and example-first snippets
- [configuration.md](./configuration.md) - High-value Quarkus logging, JSON, and OTel log properties
- [patterns.md](./patterns.md) - Repeatable logging setup workflows
- [gotchas.md](./gotchas.md) - Common logging pitfalls and fixes

## See Also

- [../configuration/README.md](../configuration/README.md) - Quarkus config source precedence, profiles, and runtime override behavior
