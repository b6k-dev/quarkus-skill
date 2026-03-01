# Quarkus Web REST Configuration Reference

Use this file for high-value REST, JSON, multipart, and HTTP behavior settings.

## High-value properties

| Property | Default | Use when |
|----------|---------|----------|
| `quarkus.rest.path` | - | You need a base path for all REST endpoints without `@ApplicationPath` |
| `quarkus.rest.exception-mapping.disable-mapper-for` | - | A built-in mapper must be disabled so custom mappers take precedence |
| `quarkus.rest.jackson.optimization.enable-reflection-free-serializers` | `false` | You want build-time generated Jackson serializers/deserializers |
| `quarkus.jackson.fail-on-unknown-properties` | `false` | Unknown JSON fields should fail request deserialization |
| `quarkus.jackson.write-dates-as-timestamps` | `false` | Date/time fields should be serialized as timestamps |
| `quarkus.http.limits.max-form-attribute-size` | `2048` | Multipart parts must allow payloads larger than the default |
| `quarkus.http.body.delete-uploaded-files-on-end` | - | Upload temp files should be deleted automatically after request end |
| `quarkus.http.body.uploads-directory` | - | Upload temp files should be written to a specific directory |
| `quarkus.http.enable-compression` | `false` | HTTP response compression should be enabled |
| `quarkus.http.compress-media-types` | built-in list | Compression should include or exclude specific content types |

## Base path

```properties
quarkus.http.root-path=/service
quarkus.rest.path=/api
```

With this setup, `@Path("/fruits")` is served from `/service/api/fruits`.

## JSON behavior

```properties
quarkus.jackson.fail-on-unknown-properties=true
quarkus.jackson.write-dates-as-timestamps=true
quarkus.rest.jackson.optimization.enable-reflection-free-serializers=true
```

## Multipart controls

```properties
quarkus.http.limits.max-form-attribute-size=20K
quarkus.http.body.uploads-directory=/var/tmp/quarkus-uploads
quarkus.http.body.delete-uploaded-files-on-end=true
```

## Compression controls

```properties
quarkus.http.enable-compression=true
quarkus.http.compress-media-types=application/json,text/plain,text/html
```

## Diagnostics logging

```properties
quarkus.log.category."org.jboss.resteasy.reactive.server.handlers.ParameterHandler".level=DEBUG
quarkus.log.category."org.jboss.resteasy.reactive.common.core.AbstractResteasyReactiveContext".level=DEBUG
quarkus.log.category."WebApplicationException".level=DEBUG
```
