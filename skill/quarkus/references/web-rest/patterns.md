# Quarkus Web REST Usage Patterns

Use these patterns for repeatable HTTP API implementation workflows.

## Pattern: Bootstrap a JSON API

When to use:

- You are starting a new JSON-first service.

Command:

```bash
quarkus create app com.acme:catalog-service --extension='rest-jackson' --no-code
```

## Pattern: Keep endpoint return types concrete

When to use:

- You want native-friendly serialization and clearer API contracts.

Example:

```java
import java.util.List;

import org.jboss.resteasy.reactive.RestResponse;

@GET
public List<Fruit> list() {
    return service.findAll();
}

@GET
@Path("{id}")
public RestResponse<Fruit> byId(String id) {
    return RestResponse.ok(service.find(id));
}
```

Prefer this over returning raw `Response` unless dynamic payload typing is required.

## Pattern: Map domain exceptions at the HTTP boundary

When to use:

- Service-layer exceptions should map to stable HTTP responses.

Example:

```java
import jakarta.ws.rs.core.Response;
import org.jboss.resteasy.reactive.RestResponse;
import org.jboss.resteasy.reactive.server.ServerExceptionMapper;

record ApiError(String code, String message) {
}

class Mappers {
    @ServerExceptionMapper
    RestResponse<ApiError> map(UnknownFruitException x) {
        return RestResponse.status(Response.Status.NOT_FOUND, new ApiError("FRUIT_NOT_FOUND", x.getMessage()));
    }
}
```

## Pattern: Upload file + JSON metadata in one request

When to use:

- Clients submit binary data and structured metadata together.

Example:

```java
@POST
@Consumes(MediaType.MULTIPART_FORM_DATA)
void upload(@RestForm("file") FileUpload file,
            @RestForm @PartType(MediaType.APPLICATION_JSON) Metadata metadata) {
    // move uploaded file to durable storage before request ends
}
```

## Pattern: Use reactive signatures intentionally

When to use:

- Endpoint behavior is asynchronous or streaming.

Example:

```java
@GET
Uni<Event> latest() {
    return service.latest();
}

@GET
Multi<Event> stream() {
    return service.stream();
}
```

Use `@Blocking` when interacting with blocking technologies.
