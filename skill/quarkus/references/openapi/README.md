# OpenAPI Reference

Use this module when the task is about Quarkus OpenAPI generation, Swagger UI, API metadata, static contract files, schema filters, or multiple published API documents.

## Overview

Quarkus OpenAPI support is provided by the `quarkus-smallrye-openapi` extension.

- Generates an OpenAPI document from Quarkus REST endpoints and MicroProfile OpenAPI annotations.
- Serves the generated contract from a built-in endpoint and can expose Swagger UI for interactive exploration.
- Supports annotation-first, configuration-first, and static-document workflows.
- Can merge generated content with static files, apply filters, and publish multiple named documents.

## General guidelines

- Keep the OpenAPI contract close to the code for operation-level details; use configuration for environment-specific metadata.
- Add stable summaries, response docs, and operation IDs when the schema will drive client generation.
- Treat Swagger UI as a developer convenience by default; if you expose it in production, secure it intentionally.
- Use a static OpenAPI document only when contract-first ownership or external review matters more than code-first generation.
- Store generated schemas during build when downstream tooling or CI needs a checked artifact.

## Quick Routing

1. Annotations, filters, schema metadata, and built-in endpoints -> `api.md`
2. High-value OpenAPI and Swagger UI configuration keys -> `configuration.md`
3. Repeatable code-first, contract-first, filtering, and multi-doc workflows -> `patterns.md`
4. Common path, merge, filter, and client-generation pitfalls -> `gotchas.md`

## In This Reference

- [api.md](./api.md) - Core annotations, filter hooks, and OpenAPI entry points
- [configuration.md](./configuration.md) - High-value OpenAPI and Swagger UI settings
- [patterns.md](./patterns.md) - Repeatable OpenAPI implementation workflows
- [gotchas.md](./gotchas.md) - Common OpenAPI and Swagger UI pitfalls and fixes

## See Also

- [Web REST Reference](../web-rest/README.md) - REST endpoint design, payload handling, and HTTP behavior
