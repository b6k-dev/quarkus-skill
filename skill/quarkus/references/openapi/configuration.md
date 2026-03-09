# Quarkus OpenAPI Configuration Reference

Use this file for corrected, high-value OpenAPI and Swagger UI configuration keys.

## Rules that matter first

- Quarkus exposes the same property families for the default document and named documents: `quarkus.smallrye-openapi.<document-name>.*`.
- `mp.openapi.*` properties apply to all generated documents, not just one named document.
- Endpoint/UI inclusion and some path/build output settings are build-time sensitive; rebuild packaged apps after changing them.
- Deprecated keys to avoid: `quarkus.smallrye-openapi.enable`, `quarkus.swagger-ui.enable`, and `quarkus.smallrye-openapi.always-run-filter`.

## OpenAPI endpoint and document output

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.smallrye-openapi.management.enabled` | `true` | The management interface is enabled and OpenAPI should or should not be exposed there |
| `quarkus.smallrye-openapi.enabled` | `true` | The OpenAPI endpoint itself must be disabled entirely |
| `quarkus.smallrye-openapi.path` | `openapi` | The default document should be served from a custom path |
| `quarkus.smallrye-openapi.<document-name>.path` | `openapi-<document-name>` | A named document needs its own custom path |
| `quarkus.smallrye-openapi.store-schema-directory` | - | Build output should include generated `openapi.yaml` and `openapi.json` files |
| `quarkus.smallrye-openapi.store-schema-file-name` | `openapi` | Stored schema files need a predictable custom base name |
| `quarkus.smallrye-openapi.ignore-static-document` | `false` | `META-INF/openapi.*` exists but should not be merged |
| `quarkus.smallrye-openapi.additional-docs-directory` | - | Static YAML or JSON fragments should be merged from additional directories |
| `quarkus.smallrye-openapi.open-api-version` | generated default | The served contract must target a specific OpenAPI version |
| `quarkus.smallrye-openapi.servers` | - | The default document should advertise explicit server URLs |
| `quarkus.smallrye-openapi.<document-name>.servers` | - | A named document should advertise a different server list |

For named documents, use the document-specific form such as `quarkus.smallrye-openapi.user.store-schema-directory=target/openapi`.

## Metadata and generated content

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.smallrye-openapi.info-title` | - | Title should come from config instead of annotations |
| `quarkus.smallrye-openapi.info-version` | - | Version should be injected by environment or release process |
| `quarkus.smallrye-openapi.info-description` | - | Top-level description should be set from config |
| `quarkus.smallrye-openapi.info-terms-of-service` | - | Terms of service should be published in the document |
| `quarkus.smallrye-openapi.info-contact-name` | - | Contact metadata should be included |
| `quarkus.smallrye-openapi.info-contact-email` | - | Contact email should be included |
| `quarkus.smallrye-openapi.info-contact-url` | - | Contact URL should be included |
| `quarkus.smallrye-openapi.info-license-name` | - | License metadata should be included |
| `quarkus.smallrye-openapi.info-license-url` | - | License URL should be included |
| `quarkus.smallrye-openapi.operation-id-strategy` | - | Operation IDs should be generated consistently for client generation |
| `quarkus.smallrye-openapi.merge-schema-examples` | `true` | You need to preserve deprecated `@Schema(example=...)` behavior instead of merging into `examples` |
| `quarkus.smallrye-openapi.auto-add-tags` | `true` | Automatic class-name tags should be enabled or disabled |
| `quarkus.smallrye-openapi.auto-add-operation-summary` | `true` | Method-name-based summaries should be enabled or disabled |
| `quarkus.smallrye-openapi.auto-add-bad-request-response` | `true` | Quarkus should or should not add default 400 responses for operations with input |
| `quarkus.smallrye-openapi.auto-add-security-requirement` | `true` | `@RolesAllowed` endpoints should automatically gain security requirements |
| `quarkus.smallrye-openapi.auto-add-security` | `true` | Generated security requirements should be added from configured schemes |
| `quarkus.smallrye-openapi.auto-add-server` | unset | A default server entry should be generated when none is provided |
| `quarkus.smallrye-openapi.auto-add-open-api-endpoint` | `false` | The schema should include the OpenAPI endpoint itself |
| `quarkus.smallrye-openapi.scan-profiles` | - | Named documents should include only selected profile-tagged operations |
| `quarkus.smallrye-openapi.scan-exclude-profiles` | - | Named documents should exclude selected profile-tagged operations |

The documented built-in operation ID strategies are `method`, `class-method`, and `package-class-method`.

## MicroProfile OpenAPI properties

| Property | Default | Use when |
|----------|---------|----------|
| `mp.openapi.scan.disable` | `false` | Only static OpenAPI documents should be served |
| `mp.openapi.filter` | - | A single MicroProfile `OASFilter` should be registered declaratively |
| `mp.openapi.servers` | - | The same server list should apply to all documents |
| `mp.openapi.extensions.smallrye.remove-unused-schemas.enable` | `false` | Filtered/named documents should drop unreferenced schemas |

## Security scheme helpers

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.smallrye-openapi.security-scheme` | - | A standard security scheme should be declared from config |
| `quarkus.smallrye-openapi.security-scheme-name` | `SecurityScheme` | The generated security scheme needs a stable public name |
| `quarkus.smallrye-openapi.security-scheme-description` | `Authentication` | The generated scheme needs a better description |
| `quarkus.smallrye-openapi.security-scheme-extensions."extension-name"` | - | Vendor extensions should be added to the scheme |
| `quarkus.smallrye-openapi.api-key-parameter-in` | - | `api-key` security needs a `query`, `header`, or `cookie` location |
| `quarkus.smallrye-openapi.api-key-parameter-name` | - | `api-key` security needs the header/query/cookie name |
| `quarkus.smallrye-openapi.basic-security-scheme-value` | `basic` | Basic auth scheme text must be customized |
| `quarkus.smallrye-openapi.jwt-security-scheme-value` | `bearer` | JWT scheme value must be customized |
| `quarkus.smallrye-openapi.jwt-bearer-format` | `JWT` | JWT bearer format needs a custom value |
| `quarkus.smallrye-openapi.oauth2-security-scheme-value` | `bearer` | OAuth2 opaque token scheme value must be customized |
| `quarkus.smallrye-openapi.oauth2-bearer-format` | `Opaque` | OAuth2 bearer format must be customized |
| `quarkus.smallrye-openapi.oidc-open-id-connect-url` | - | OIDC discovery metadata should be published |
| `quarkus.smallrye-openapi.oauth2-implicit-authorization-url` | - | OAuth2 implicit flow needs an authorization URL |
| `quarkus.smallrye-openapi.oauth2-implicit-token-url` | - | OAuth2 implicit flow needs a token URL |
| `quarkus.smallrye-openapi.oauth2-implicit-refresh-url` | - | OAuth2 implicit flow needs a refresh URL |

## Swagger UI essentials

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.swagger-ui.path` | `swagger-ui` | Swagger UI should live at a custom path |
| `quarkus.swagger-ui.always-include` | `false` | Swagger UI must be packaged outside dev and test |
| `quarkus.swagger-ui.enabled` | `true` | Included Swagger UI should still be switchable on or off |
| `quarkus.swagger-ui.urls."name"` | - | The top bar should offer multiple OpenAPI documents |
| `quarkus.swagger-ui.urls-primary-name` | - | One of the configured document entries should be preselected |
| `quarkus.swagger-ui.title` | - | The browser page title should be customized |
| `quarkus.swagger-ui.theme` | - | A non-default Swagger UI theme should be used |
| `quarkus.swagger-ui.footer` | - | The UI should show a custom footer |
| `quarkus.swagger-ui.filter` | - | Tag filtering should be enabled |
| `quarkus.swagger-ui.doc-expansion` | Swagger UI default | Tags and operations should start expanded or collapsed |
| `quarkus.swagger-ui.display-operation-id` | `false` | Operation IDs should be visible in the UI |
| `quarkus.swagger-ui.display-request-duration` | Swagger UI default | Try-it-out request duration should be visible |
| `quarkus.swagger-ui.default-models-expand-depth` | Swagger UI default | Model sidebar depth should be controlled |
| `quarkus.swagger-ui.default-model-expand-depth` | Swagger UI default | Example/model section depth should be controlled |
| `quarkus.swagger-ui.default-model-rendering` | Swagger UI default | Example vs model view should be controlled |
| `quarkus.swagger-ui.operations-sorter` | server order | Operations should be sorted by path or method |
| `quarkus.swagger-ui.tags-sorter` | Swagger UI default | Tags should be sorted predictably |
| `quarkus.swagger-ui.supported-submit-methods` | all methods | Try-it-out should be limited to specific HTTP verbs |
| `quarkus.swagger-ui.try-it-out-enabled` | `false` | Try-it-out should start enabled by default |
| `quarkus.swagger-ui.query-config-enabled` | `false` | Query parameter tweaking in the UI should be enabled |
| `quarkus.swagger-ui.persist-authorization` | Swagger UI default | Browser refresh should keep auth state |
| `quarkus.swagger-ui.validator-url` | swagger.io validator | External schema validation should be redirected or disabled |

## Swagger UI advanced hooks and auth

Use these when the built-in UI needs runtime customization or OAuth preconfiguration:

- Request/response hooks: `quarkus.swagger-ui.request-interceptor`, `quarkus.swagger-ui.response-interceptor`, `quarkus.swagger-ui.request-curl-options`, `quarkus.swagger-ui.show-mutated-request`
- Rendering/customization: `quarkus.swagger-ui.show-extensions`, `quarkus.swagger-ui.show-common-extensions`, `quarkus.swagger-ui.syntax-highlight`, `quarkus.swagger-ui.layout`, `quarkus.swagger-ui.plugins`, `quarkus.swagger-ui.scripts`, `quarkus.swagger-ui.presets`, `quarkus.swagger-ui.on-complete`
- Browser/network behavior: `quarkus.swagger-ui.with-credentials`, `quarkus.swagger-ui.oauth2-redirect-url`
- OAuth init settings: `quarkus.swagger-ui.oauth-client-id`, `quarkus.swagger-ui.oauth-client-secret`, `quarkus.swagger-ui.oauth-realm`, `quarkus.swagger-ui.oauth-app-name`, `quarkus.swagger-ui.oauth-scope-separator`, `quarkus.swagger-ui.oauth-scopes`, `quarkus.swagger-ui.oauth-additional-query-string-params`, `quarkus.swagger-ui.oauth-use-basic-authentication-with-access-code-grant`, `quarkus.swagger-ui.oauth-use-pkce-with-authorization-code-grant`
- Preauthorization helpers: `quarkus.swagger-ui.preauthorize-basic-auth-definition-key`, `quarkus.swagger-ui.preauthorize-basic-username`, `quarkus.swagger-ui.preauthorize-basic-password`, `quarkus.swagger-ui.preauthorize-api-key-auth-definition-key`, `quarkus.swagger-ui.preauthorize-api-key-api-key-value`

## Workflow note

Use `patterns.md` for worked configuration combinations such as metadata-by-environment, static-only contracts, stored schema artifacts, multi-document setups, and Swagger UI document pickers.

## Build-time reminders

- `quarkus.swagger-ui.always-include` is build-time fixed.
- OpenAPI/Swagger endpoint packaging and some stored-schema output behavior are decided at build time.
- Rebuild after changing build-time OpenAPI or Swagger UI properties in packaged applications.

## See Also

- [Patterns](./patterns.md) - repeatable OpenAPI and Swagger UI workflows.
