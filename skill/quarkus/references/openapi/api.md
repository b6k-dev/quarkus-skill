# Quarkus OpenAPI API Reference

Use this file for runtime OpenAPI annotations, filter hooks, and built-in endpoint behavior.

## Extension entry point

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-openapi</artifactId>
</dependency>
```

The extension generates the OpenAPI document and includes Swagger UI support.

## Default endpoints

- OpenAPI document: `/q/openapi`
- JSON format: `/q/openapi?format=json`
- Swagger UI: `/q/swagger-ui` in dev and test by default

Use `configuration.md` for path overrides and packaged Swagger UI behavior.

## Application-level metadata

```java
import jakarta.ws.rs.core.Application;
import org.eclipse.microprofile.openapi.annotations.OpenAPIDefinition;
import org.eclipse.microprofile.openapi.annotations.info.Contact;
import org.eclipse.microprofile.openapi.annotations.info.Info;
import org.eclipse.microprofile.openapi.annotations.tags.Tag;

@OpenAPIDefinition(
    info = @Info(
        title = "Catalog API",
        version = "1.0.0",
        description = "Operations for product catalog management",
        contact = @Contact(name = "API Support", email = "support@example.com")
    ),
    tags = @Tag(name = "catalog", description = "Catalog operations")
)
public class ApiApplication extends Application {
}
```

Use configuration instead when titles or URLs differ by environment.

## Documenting an operation

```java
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.eclipse.microprofile.openapi.annotations.Operation;
import org.eclipse.microprofile.openapi.annotations.responses.APIResponse;
import org.eclipse.microprofile.openapi.annotations.tags.Tag;

@Path("/products")
@Tag(name = "products")
class ProductResource {
    @GET
    @Operation(summary = "List products", operationId = "listProducts")
    @APIResponse(responseCode = "200", description = "Products returned")
    public List<Product> list() {
        return service.findAll();
    }
}
```

Prefer explicit `operationId` values when clients will be generated from the schema.

## Request and response schemas

```java
import org.eclipse.microprofile.openapi.annotations.media.Content;
import org.eclipse.microprofile.openapi.annotations.media.Schema;
import org.eclipse.microprofile.openapi.annotations.parameters.RequestBody;
import org.eclipse.microprofile.openapi.annotations.responses.APIResponse;

@POST
@RequestBody(required = true, content = @Content(schema = @Schema(implementation = CreateProduct.class)))
@APIResponse(
    responseCode = "201",
    description = "Product created",
    content = @Content(schema = @Schema(implementation = Product.class))
)
public Product create(CreateProduct command) {
    return service.create(command);
}
```

## Schema hints on models

```java
import org.eclipse.microprofile.openapi.annotations.media.Schema;

@Schema(name = "Product", description = "Catalog product returned by the API")
public class Product {
    @Schema(example = "sku-123")
    public String sku;

    @Schema(example = "Coffee mug")
    public String name;
}
```

## Security metadata

```java
import org.eclipse.microprofile.openapi.annotations.Operation;
import org.eclipse.microprofile.openapi.annotations.enums.SecuritySchemeType;
import org.eclipse.microprofile.openapi.annotations.security.SecurityRequirement;
import org.eclipse.microprofile.openapi.annotations.security.SecurityScheme;

@SecurityScheme(
    securitySchemeName = "bearerAuth",
    type = SecuritySchemeType.HTTP,
    scheme = "bearer",
    bearerFormat = "JWT"
)
class OpenApiSecurity {
}

@GET
@Operation(summary = "Read current profile")
@SecurityRequirement(name = "bearerAuth")
public Profile me() {
    return service.currentProfile();
}
```

Use `configuration.md` or `patterns.md` when the security scheme should be declared centrally from properties.

## OpenAPI filters

```java
import io.quarkus.smallrye.openapi.OpenApiFilter;
import org.eclipse.microprofile.openapi.OASFilter;
import org.eclipse.microprofile.openapi.models.OpenAPI;

@OpenApiFilter(stages = OpenApiFilter.RunStage.BUILD)
public class BuildTimeFilter implements OASFilter {
    @Override
    public void filterOpenAPI(OpenAPI openAPI) {
        openAPI.getInfo().setDescription("Contract enriched during build");
    }
}
```

Use `RUNTIME_PER_REQUEST` only when the schema truly needs request-specific changes.

## Static documents

Supported contract file locations include:

- `src/main/resources/META-INF/openapi.yml`
- `src/main/resources/META-INF/openapi.yaml`
- `src/main/resources/META-INF/openapi.json`

Static content is merged with generated content unless scanning is disabled.
