# Quarkus Templates Gotchas (Qute)

Common Qute pitfalls, symptoms, and fixes.

## Template discovery and validation

| Symptom                                                   | Likely cause                                                                 | Fix                                                                                      |
|-----------------------------------------------------------|------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| Injected template is not found                            | Injection point name does not match the template path                        | Rename the field or add `@Location("...")`                                               |
| A template file is silently ignored                       | File name contains whitespace                                                | Rename the template to a non-whitespace path                                             |
| Build validation is skipped unexpectedly                  | Template comes from a custom `@Locate` locator                               | Use `@Locate` intentionally and understand validation is disabled for matching templates |
| Type-safe expression stops being validated inside a block | A section alias such as `item` or `foo` overrides the checked parameter name | Use a different loop/section alias or add explicit parameter declarations where needed   |

## Rendering and expression behavior

| Symptom                                           | Likely cause                                                | Fix                                                                                         |
|---------------------------------------------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| Output contains `NOT_FOUND` or rendering fails    | Expression resolves to a missing property                   | Keep `quarkus.qute.strict-rendering=true` or set a deliberate `property-not-found-strategy` |
| Template output has unexpected blank-line changes | Standalone section/comment lines are removed by default     | Set `quarkus.qute.remove-standalone-lines=false` if whitespace matters                      |
| HTML markup prints as escaped text                | HTML/XML content types escape special characters by default | Use `.raw` only for trusted content                                                         |
| JSON output becomes invalid                       | Raw content bypasses JSON escaping                          | Avoid `.raw` inside JSON templates unless you fully control the serialized text             |

## Type safety and native executables

| Symptom                                                                  | Likely cause                                                                     | Fix                                                                                                                  |
|--------------------------------------------------------------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| Template works on JVM but fails in native with property-not-found errors | Reflection-based resolver is unavailable in native mode                          | Prefer `@CheckedTemplate`, parameter declarations, or `@TemplateData`; use `@RegisterForReflection` only as fallback |
| Nested expressions inside `with` are not validated                       | `with` changes context in a way that prevents type-safe validation               | Prefer `let` with explicit bindings in checked templates                                                             |
| Build fails after adding a fragment helper method like `items$row()`     | Checked fragment method does not match a real fragment id or required parameters | Add the fragment to the template or align the method name and parameters                                             |

## REST integration and threading

| Symptom                                                                                 | Likely cause                                                                    | Fix                                                                              |
|-----------------------------------------------------------------------------------------|---------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| A REST endpoint returning `TemplateInstance` blocks unexpectedly                        | Rendering path performs blocking work on a non-blocking endpoint                | Move blocking work out of rendering or mark the resource method with `@Blocking` |
| Wrong variant is rendered for HTML vs text                                              | `Accept` negotiation or suffix/content-type mapping does not match expectations | Check `@Produces`, template suffixes, and `quarkus.qute.content-types.*`         |
| `RestTemplate` lookup succeeds in code but missing template is not caught at build time | `RestTemplate` templates are derived dynamically and not validated              | Prefer injected templates or checked templates for important views               |

## Tags, fragments, and async data

| Symptom                                     | Likely cause                                                                 | Fix                                                                                   |
|---------------------------------------------|------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| User tag cannot see parent data             | Tags are isolated by default                                                 | Pass explicit arguments or use `_unisolated` only when necessary                      |
| A fragment renders nothing in the main page | Fragment was defined as hidden via `capture`, `_hidden`, or `rendered=false` | Include or render the fragment explicitly                                             |
| Rendering times out                         | A `Uni` or `CompletionStage` never completes, or the timeout is too low      | Fix the async source first; then tune `quarkus.qute.timeout` if needed                |
| Hard to identify which expression hangs     | Async resolution stalls deep inside the template                             | Enable `TRACE` for `io.quarkus.qute.nodeResolve` and inspect the last unresolved node |
