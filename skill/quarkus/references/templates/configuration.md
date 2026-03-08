# Quarkus Templates Configuration Reference (Qute)

Use this file for high-value Qute settings, validation behavior, and render-time controls.

## High-value Quarkus configuration keys

| Property                                         | Default                               | Use when                                                           |
|--------------------------------------------------|---------------------------------------|--------------------------------------------------------------------|
| `quarkus.qute.suffixes`                          | `qute.html,qute.txt,html,txt`         | Template lookup should allow custom suffixes                       |
| `quarkus.qute.content-types.*`                   | URLConnection-based mapping           | Variant/content type detection needs extra suffix mappings         |
| `quarkus.qute.type-check-excludes`               | -                                     | Specific properties or methods must be skipped during validation   |
| `quarkus.qute.template-path-exclude`             | hidden files excluded                 | Some template paths should be ignored entirely                     |
| `quarkus.qute.iteration-metadata-prefix`         | `<alias_>`                            | Loop metadata prefix should change                                 |
| `quarkus.qute.escape-content-types`              | HTML/XML set                          | Auto-escaping should apply to more or fewer content types          |
| `quarkus.qute.default-charset`                   | `UTF-8`                               | Template files use a different default charset                     |
| `quarkus.qute.duplicit-templates-strategy`       | `prioritize`                          | Duplicate template path handling must fail fast                    |
| `quarkus.qute.dev-mode.no-restart-templates`     | -                                     | Some templates should hot-reload without app restart               |
| `quarkus.qute.test-mode.record-rendered-results` | `true`                                | Test render recording should be disabled                           |
| `quarkus.qute.debug.enabled`                     | `true`                                | Experimental Qute debug mode should be turned off                  |
| `quarkus.qute.property-not-found-strategy`       | dev-specific behavior when non-strict | Missing properties should noop, throw, or echo original expression |
| `quarkus.qute.remove-standalone-lines`           | `true`                                | Whitespace around section-only lines must be preserved             |
| `quarkus.qute.strict-rendering`                  | `true`                                | Missing values should fail fast or be tolerated                    |
| `quarkus.qute.timeout`                           | `10000`                               | Global render timeout must change                                  |
| `quarkus.qute.use-async-timeout`                 | `true`                                | Async rendering timeout behavior must change                       |

## Strict vs non-strict rendering

Fail on unresolved expressions:

```properties
quarkus.qute.strict-rendering=true
```

Allow unresolved expressions and control output:

```properties
quarkus.qute.strict-rendering=false
quarkus.qute.property-not-found-strategy=noop
```

`property-not-found-strategy` is ignored when strict rendering is enabled.

## Template lookup and content types

Add a custom suffix and content type mapping:

```properties
quarkus.qute.suffixes=html,txt,email
quarkus.qute.content-types.email=text/plain
```

Useful when one logical template ID should resolve to custom file variants.

## Template validation controls

Skip noisy members during type checks:

```properties
quarkus.qute.type-check-excludes=org.acme.LegacyDto.*,*.class
```

Ignore generated or hidden templates completely:

```properties
quarkus.qute.template-path-exclude=generated/.*|mail/experimental/.*
```

Excluded templates are not parsed, validated, or available at runtime.

## Loop metadata naming

Use no prefix for loop metadata:

```properties
quarkus.qute.iteration-metadata-prefix=<none>
```

Then `{count}` and `{hasNext}` work directly inside loops.

## Escaping and output formatting

Disable standalone-line removal:

```properties
quarkus.qute.remove-standalone-lines=false
```

Extend auto-escaping to JSON templates:

```properties
quarkus.qute.escape-content-types=text/html,text/xml,application/xml,application/xhtml+xml,application/json
```

## Dev mode and test mode

Hot-reload selected templates without full restart:

```properties
quarkus.qute.dev-mode.no-restart-templates=templates/fragments/.*
```

Disable rendered result recording in tests:

```properties
quarkus.qute.test-mode.record-rendered-results=false
```

## Timeout tuning

Increase the render timeout for async or slow data resolution:

```properties
quarkus.qute.timeout=30000
quarkus.qute.use-async-timeout=true
```

Prefer fixing slow or never-completing `Uni`/`CompletionStage` inputs before raising this value.
