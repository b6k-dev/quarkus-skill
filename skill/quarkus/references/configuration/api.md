# Quarkus Configuration API Reference (MicroProfile Config / SmallRye)

Use this file for runtime config APIs with short, copy-ready examples.

## `@ConfigProperty`

```java
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.config.inject.ConfigProperty;

import java.util.Optional;

@ApplicationScoped
class GreetingConfig {
    @ConfigProperty(name = "greeting.message")
    String message;

    @ConfigProperty(name = "greeting.suffix", defaultValue = "!")
    String suffix;

    @ConfigProperty(name = "greeting.name")
    Optional<String> name;
}
```

- Missing required keys fail startup.
- `defaultValue` is used when the key is missing.
- `Optional<T>` allows missing keys.

## Programmatic access

```java
import io.smallrye.config.SmallRyeConfig;
import org.eclipse.microprofile.config.ConfigProvider;

SmallRyeConfig cfg = ConfigProvider.getConfig().unwrap(SmallRyeConfig.class);
String db = cfg.getValue("database.name", String.class);
ServerConfig server = cfg.getConfigMapping(ServerConfig.class);
```

## `@ConfigMapping` (recommended)

```java
import io.smallrye.config.ConfigMapping;
import io.smallrye.config.WithDefault;
import io.smallrye.config.WithName;

@ConfigMapping(prefix = "server")
interface ServerConfig {
    String host();
    int port();
    Log log();

    interface Log {
        @WithDefault("false")
        boolean enabled();

        @WithName("file-suffix")
        @WithDefault(".log")
        String suffix();
    }
}
```

```java
import jakarta.inject.Inject;

@Inject
ServerConfig server;
```

- `server.host` -> `host()`
- `server.log.file-suffix` -> `log().suffix()`

## Collections, maps, converters

```java
import io.smallrye.config.ConfigMapping;
import io.smallrye.config.WithConverter;
import org.eclipse.microprofile.config.spi.Converter;

import java.util.List;
import java.util.Map;

@ConfigMapping(prefix = "app")
interface AppConfig {
    List<String> origins();
    Map<String, String> labels();

    @WithConverter(UpperCaseConverter.class)
    String mode();
}

class UpperCaseConverter implements Converter<String> {
    @Override
    public String convert(String value) {
        return value.toUpperCase();
    }
}
```

```properties
app.origins[0]=https://a.example
app.labels.team=payments
app.mode=dev
```

## Validation on mappings

Requires `quarkus-hibernate-validator`.

```java
import io.smallrye.config.ConfigMapping;
import jakarta.validation.constraints.Max;
import jakarta.validation.constraints.Size;

@ConfigMapping(prefix = "server")
interface ValidatedServerConfig {
    @Size(min = 2, max = 20)
    String host();

    @Max(10000)
    int port();
}
```

## Profiles

```properties
quarkus.http.port=9090
%dev.quarkus.http.port=8181
quarkus.profile=staging,tenant-a
quarkus.config.profile.parent=common
```

- Profile-aware files: `application-dev.properties`, `application-staging.properties`.
- In `.env`, profile keys use `_PROFILE_` prefixes.

```properties
QUARKUS_HTTP_PORT=9090
_DEV_QUARKUS_HTTP_PORT=8181
```

## Environment variables

Runner JAR and native executable:

```bash
export QUARKUS_DATASOURCE_PASSWORD=youshallnotpass
java -jar target/quarkus-app/quarkus-run.jar
./target/myapp-runner
```

For `foo.BAR.baz`, config checks:

1. `foo.BAR.baz`
2. `foo_BAR_baz`
3. `FOO_BAR_BAZ`

SmallRye conversion examples:

- `foo."bar".baz` -> `FOO__BAR__BAZ`
- `foo.bar-baz` -> `FOO_BAR_BAZ`
- `foo.bar[0]` -> `FOO_BAR_0_`
- `foo.bar[0].baz` -> `FOO_BAR_0__BAZ`

Dynamic segments need a dotted key to disambiguate env names:

```properties
quarkus.datasource."datasource-name".jdbc.url=
```

```bash
export QUARKUS_DATASOURCE__DATASOURCE_NAME__JDBC_URL=jdbc:postgresql://localhost:5432/database
```

## Property expressions

```properties
host=quarkus.io
application.host=${HOST:${host}}
application.url=https://${application.host}
```

Supported forms: `${key}`, `${key:default}`, `${outer${inner}}`.

## Secret key expressions

```properties
my.secret=${aes-gcm-nopadding::ENCRYPTED_VALUE}
smallrye.config.secret-handler.aes-gcm-nopadding.encryption-key=some-key
```

## Static init safe mappings

```java
import io.smallrye.config.ConfigMapping;

@io.quarkus.runtime.annotations.StaticInitSafe
@ConfigMapping(prefix = "bootstrap")
interface BootstrapConfig {
    String region();
}
```
