# Quarkus OpenAPI Gotchas

Common OpenAPI and Swagger UI pitfalls, symptoms, and fixes.

## Paths and visibility

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Swagger UI works in dev but is missing in production | `quarkus.swagger-ui.always-include` defaults to `false` | Set `quarkus.swagger-ui.always-include=true` before building and secure access intentionally |
| OpenAPI or Swagger UI path is not where expected | Relative paths are resolved under Quarkus' non-application root | Use an absolute path starting with `/` when you need an exact public URL |
| Setting `quarkus.swagger-ui.path=/` breaks the app | `/` is not a valid Swagger UI path | Use a dedicated sub-path such as `docs` or `/docs` |

## Generated vs static documents

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Static YAML is present but generated endpoints still appear | Static documents are merged with generated output by default | Set `mp.openapi.scan.disable=true` for static-only mode |
| A checked-in static contract unexpectedly changes the main document | `META-INF/openapi.*` is being auto-loaded | Remove the file, move it to an additional docs directory, or set `quarkus.smallrye-openapi.ignore-static-document=true` |

## Filters and rebuilds

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Filter runs only once even though the document should be dynamic | Filter stage is `BUILD` or `RUNTIME_STARTUP` | Use `@OpenApiFilter(stages = OpenApiFilter.RunStage.RUNTIME_PER_REQUEST)` |
| Packaged app ignores Swagger UI visibility changes | `quarkus.swagger-ui.always-include` is a build-time property | Rebuild the application after changing it |
| Per-request filter logic is expensive | Dynamic filter runs on every schema request | Move work to `BUILD` or `RUNTIME_STARTUP` unless request-specific data is required |

## Multiple documents

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Named documents still share some settings unexpectedly | `mp.openapi.*` properties apply to all documents | Use `quarkus.smallrye-openapi.<document-name>.*` when a setting should affect only one named document |
| Operation appears in the wrong named document | Profile extension is missing or mismatched | Verify `@Extension(name = "x-smallrye-profile-...")` values and matching `scan-profiles` config |
| Class-level and method-level profile extensions do not combine | Method extensions override class extensions for that operation | Put the full intended profile set on the method when it needs custom routing |

## Contract quality

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Generated client method names are unstable | Operation IDs are omitted or derived from renamed Java methods | Set explicit `@Operation(operationId = ...)` values or configure an operation ID strategy |
| Secured endpoints look public in the contract | Security scheme metadata or requirements are missing | Define a `@SecurityScheme` or matching config and add `@SecurityRequirement` where needed |
| Split documents still contain unused schemas | Schema cleanup is disabled | Set `mp.openapi.extensions.smallrye.remove-unused-schemas.enable=true` |
