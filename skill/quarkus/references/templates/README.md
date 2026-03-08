# Templates Reference

Use this module when the task is about Quarkus templating with Qute: server-side HTML, text/email/report rendering, template validation, fragments, tags, and localized messages.

## Overview

Qute is Quarkus' templating engine with build-time validation, live reload integration, and native-friendly resolution.

- Templates live under `src/main/resources/templates`.
- Quarkus can inject `Template`, render `TemplateInstance`, and validate type-safe templates at build time.
- Qute works for HTML pages, plain text, emails, reports, fragments, and localized message bundles.
- REST integration is provided by `quarkus-rest-qute`; zero-controller HTTP serving is available via `quarkus-qute-web`.

## General guidelines

- Prefer `@CheckedTemplate` or template parameter declarations for build-time validation.
- Keep business logic in Java; use templates for presentation, simple branching, and formatting.
- Use `@TemplateExtension` for computed properties and formatting helpers.
- Organize templates by feature or resource class so lookups stay predictable.
- Favor generated resolvers (`@CheckedTemplate`, `@TemplateData`) over reflection-heavy access, especially for native builds.

## Quick Routing

1. Injection, checked templates, sections, fragments, extensions, i18n -> `api.md`
2. Qute-specific config keys and rendering behavior -> `configuration.md`
3. Repeatable SSR, layout, fragment, and report workflows -> `patterns.md`
4. Validation failures, native issues, escaping, and threading traps -> `gotchas.md`

## In This Reference

- [api.md](./api.md) - Core Qute APIs, syntax, and integration points
- [configuration.md](./configuration.md) - High-value Qute settings and defaults
- [patterns.md](./patterns.md) - Repeatable templating workflows
- [gotchas.md](./gotchas.md) - Common Qute pitfalls and fixes

## See Also

- [Web REST Reference](../web-rest/README.md) - HTTP endpoint design and request/response handling
- [Configuration Reference](../configuration/README.md) - Quarkus config fundamentals beyond Qute-specific settings
