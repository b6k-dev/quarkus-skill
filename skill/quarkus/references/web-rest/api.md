# Quarkus Web REST API Reference

Use this file for runtime REST APIs with short, copy-ready examples.

## Extension entry points

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest</artifactId>
</dependency>
```

JSON variants:

- `io.quarkus:quarkus-rest-jackson`
- `io.quarkus:quarkus-rest-jsonb`

## Minimal endpoint

```java
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/hello")
class GreetingResource {
    @GET
    String hello() {
        return "hello";
    }
}
```

## Base path

```java
import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;

@ApplicationPath("/api")
public class ApiApplication extends Application {
}
```

Alternative: set `quarkus.rest.path=/api`.

## Request parameters

```java
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import org.jboss.resteasy.reactive.RestCookie;
import org.jboss.resteasy.reactive.RestHeader;
import org.jboss.resteasy.reactive.RestPath;
import org.jboss.resteasy.reactive.RestQuery;

@Path("/items/{id}")
class ItemResource {
    @GET
    String get(@RestPath String id,
               @RestQuery String expand,
               @RestCookie("tenant") String tenant,
               @RestHeader("X-Request-Id") String requestId) {
        return id;
    }
}
```

`@RestPath` is optional when the parameter name matches a URI template variable.

## Typed responses

```java
import org.jboss.resteasy.reactive.RestResponse;

@GET
RestResponse<Fruit> byId() {
    return RestResponse.ok(new Fruit());
}
```

Prefer `RestResponse<T>` over raw `Response` when possible.

## Multipart form handling

```java
import jakarta.ws.rs.Consumes;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.core.MediaType;
import org.jboss.resteasy.reactive.PartType;
import org.jboss.resteasy.reactive.RestForm;
import org.jboss.resteasy.reactive.multipart.FileUpload;

@Path("/uploads")
class UploadResource {
    static class Metadata {
        public String owner;
    }

    @POST
    @Consumes(MediaType.MULTIPART_FORM_DATA)
    void upload(@RestForm("file") FileUpload file,
                @RestForm @PartType(MediaType.APPLICATION_JSON) Metadata metadata) {
    }
}
```

## Reactive endpoints

```java
import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;

@GET
Uni<Fruit> latest() {
    return Uni.createFrom().item(new Fruit());
}

@GET
Multi<Fruit> stream() {
    return Multi.createFrom().items(new Fruit(), new Fruit());
}
```

Use `Uni` for one async value and `Multi` for streams.

## Exception mapping

```java
import jakarta.ws.rs.core.Response;
import org.jboss.resteasy.reactive.RestResponse;
import org.jboss.resteasy.reactive.server.ServerExceptionMapper;

record ApiError(String message) {
}

class Mappers {
    @ServerExceptionMapper
    RestResponse<ApiError> map(UnknownFruitException x) {
        return RestResponse.status(Response.Status.NOT_FOUND, new ApiError(x.getMessage()));
    }
}
```

## Request/response filters

```java
import jakarta.ws.rs.container.ContainerRequestContext;
import jakarta.ws.rs.container.ContainerResponseContext;
import org.jboss.resteasy.reactive.server.ServerRequestFilter;
import org.jboss.resteasy.reactive.server.ServerResponseFilter;

class Filters {
    @ServerRequestFilter
    void before(ContainerRequestContext ctx) {
    }

    @ServerResponseFilter
    void after(ContainerResponseContext ctx) {
    }
}
```
