# Quarkus OpenAPI Usage Patterns

Use these patterns for repeatable OpenAPI and Swagger UI workflows.

## Pattern: Add OpenAPI to an existing REST service

When to use:

- You already have Quarkus REST endpoints and want generated API docs quickly.

Command:

```bash
quarkus extension add quarkus-smallrye-openapi
```

Verify by starting dev mode and opening `/q/openapi` or `/q/swagger-ui`.

## Pattern: Set global API metadata without an `Application` class

When to use:

- Titles, versions, or contact details vary by environment.

Example:

```properties
quarkus.smallrye-openapi.info-title=Inventory API
%dev.quarkus.smallrye-openapi.info-title=Inventory API (dev)
quarkus.smallrye-openapi.info-version=1.0.0
quarkus.smallrye-openapi.info-description=Inventory and stock management endpoints
quarkus.smallrye-openapi.info-contact-name=API Support
quarkus.smallrye-openapi.info-contact-email=support@example.com
quarkus.smallrye-openapi.info-license-name=Apache-2.0
```

Prefer this over hardcoding metadata annotations when deployments need different labels.

## Pattern: Make generated contracts client-friendly

When to use:

- OpenAPI will feed SDK generation, automation, or external consumers.

Example:

```java
@GET
@Operation(summary = "Get product by id", operationId = "getProductById")
@APIResponse(responseCode = "200", description = "Product returned")
@APIResponse(responseCode = "404", description = "Product not found")
public Product byId(@RestPath String id) {
    return service.find(id);
}
```

Also consider `quarkus.smallrye-openapi.operation-id-strategy` when explicit IDs are not practical.

## Pattern: Serve a static contract instead of generated output

When to use:

- The API contract is reviewed or owned outside the implementation codebase.

Example:

```yaml
openapi: 3.1.0
info:
  title: Inventory API
  version: "1.0"
paths:
  /products:
    get:
      responses:
        "200":
          description: OK
```

```properties
mp.openapi.scan.disable=true
```

Put the file at `src/main/resources/META-INF/openapi.yml`.

If a checked-in `META-INF/openapi.*` file should be ignored for one document, use `quarkus.smallrye-openapi.<document-name>.ignore-static-document=true`.

## Pattern: Customize public docs paths

When to use:

- OpenAPI and Swagger UI should live at stable URLs outside the Quarkus defaults.

Example:

```properties
quarkus.smallrye-openapi.path=/swagger
quarkus.swagger-ui.path=docs
```

Paths starting with `/` are absolute instead of being resolved under the non-application root.

## Pattern: Merge static fragments with generated endpoints

When to use:

- The generated schema is mostly correct but some shared components or descriptive sections live in files.

Example:

```properties
quarkus.smallrye-openapi.additional-docs-directory=META-INF/openapi
```

Keep generated endpoint details in code and shared reusable fragments in static YAML or JSON.

## Pattern: Publish multiple OpenAPI documents

When to use:

- A large application needs separate contracts for different consumers or bounded contexts.

Example:

```java
import org.eclipse.microprofile.openapi.annotations.extensions.Extension;

@GET
@Path("/users")
@Extension(name = "x-smallrye-profile-user", value = "")
public List<UserDto> listUsers() {
    return service.listUsers();
}

@GET
@Path("/orders")
@Extension(name = "x-smallrye-profile-order", value = "")
public List<OrderDto> listOrders() {
    return service.listOrders();
}
```

```properties
quarkus.smallrye-openapi.user.scan-profiles=user
quarkus.smallrye-openapi.order.scan-profiles=order
quarkus.smallrye-openapi.user.path=/openapi-user
quarkus.smallrye-openapi.order.path=/openapi-order
mp.openapi.extensions.smallrye.remove-unused-schemas.enable=true
```

If all documents should advertise the same server list, use `mp.openapi.servers`; if only one named document differs, use `quarkus.smallrye-openapi.<document-name>.servers`.

## Pattern: Show multiple documents in Swagger UI

When to use:

- Consumers should switch between combined and split contracts from one UI.

Example:

```properties
quarkus.swagger-ui.always-include=true
quarkus.swagger-ui.urls."Combined"=/q/openapi
quarkus.swagger-ui.urls."User Service"=/q/openapi-user
quarkus.swagger-ui.urls."Order Service"=/q/openapi-order
quarkus.swagger-ui.urls-primary-name=Combined
quarkus.swagger-ui.try-it-out-enabled=true
quarkus.swagger-ui.persist-authorization=true
```

## Pattern: Enrich the schema with a filter

When to use:

- Generated output needs consistent post-processing that would be noisy in annotations.

Example:

```java
@OpenApiFilter(stages = OpenApiFilter.RunStage.BUILD)
public class ContractFilter implements OASFilter {
    @Override
    public void filterOpenAPI(OpenAPI openAPI) {
        openAPI.getInfo().setDescription("Generated contract for platform consumers");
    }
}
```

Prefer `BUILD` or `RUNTIME_STARTUP`; use `RUNTIME_PER_REQUEST` only for truly dynamic contracts.

## Pattern: Keep a generated artifact for CI or client generation

When to use:

- Pipelines need a stable schema file after the build finishes.

Example:

```properties
quarkus.smallrye-openapi.store-schema-directory=target/generated-openapi
quarkus.smallrye-openapi.store-schema-file-name=inventory
```

This writes files such as `inventory.yaml` and `inventory.json` during the build.

## Pattern: Configure security metadata from properties

When to use:

- The API should publish a standard security scheme without repeating annotations across resources.

Example:

```properties
quarkus.smallrye-openapi.security-scheme=jwt
quarkus.smallrye-openapi.security-scheme-name=bearerAuth
quarkus.smallrye-openapi.security-scheme-description=JWT bearer authentication
quarkus.smallrye-openapi.auto-add-security=true
quarkus.smallrye-openapi.auto-add-security-requirement=true
```

For API key auth, also set `quarkus.smallrye-openapi.api-key-parameter-in` and `quarkus.smallrye-openapi.api-key-parameter-name`.
