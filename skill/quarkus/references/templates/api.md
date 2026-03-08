# Quarkus Templates API Reference (Qute)

Use this file for Qute APIs and copy-ready examples.

## Extension entry points

Core templating:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-qute</artifactId>
</dependency>
```

Serve `TemplateInstance` from Quarkus REST:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-qute</artifactId>
</dependency>
```

Serve `templates/pub/*` directly over HTTP:

```xml
<dependency>
    <groupId>io.quarkiverse.qute.web</groupId>
    <artifactId>quarkus-qute-web</artifactId>
</dependency>
```

## Inject a template

```java
import io.quarkus.qute.Location;
import io.quarkus.qute.Template;

class Emails {
    @Inject
    Template welcome;

    @Inject
    @Location("mail/reset-password")
    Template resetPassword;
}
```

Without `@Location`, the injection point name maps to `src/main/resources/templates/<name>.*`.

## Return `TemplateInstance` from REST

```java
import io.quarkus.qute.Template;
import io.quarkus.qute.TemplateInstance;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.QueryParam;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;

@Path("/hello")
class HelloResource {
    @Inject
    Template hello;

    @GET
    @Produces(MediaType.TEXT_HTML)
    TemplateInstance hello(@QueryParam("name") String name) {
        return hello.data("name", name == null ? "Quarkus" : name);
    }
}
```

`quarkus-rest-qute` renders the returned `TemplateInstance` automatically.

## Type-safe templates with `@CheckedTemplate`

```java
import io.quarkus.qute.CheckedTemplate;
import io.quarkus.qute.TemplateInstance;

@Path("/items")
class ItemResource {
    @CheckedTemplate
    static class Templates {
        static native TemplateInstance page(Item item);
    }

    @GET
    @Path("/{id}")
    TemplateInstance get() {
        return Templates.page(service.find());
    }
}
```

This maps to `src/main/resources/templates/ItemResource/page.html` and turns `Item item` into a checked template parameter.

## Type-safe template record

```java
import io.quarkus.qute.TemplateInstance;

record Hello(String name) implements TemplateInstance {
}
```

In a resource class `HelloResource`, this maps to `src/main/resources/templates/HelloResource/Hello.html`.

## Parameter declarations in templates

```html
{@org.acme.Item item}
{@java.util.List<String> tags}

<h1>{item.name}</h1>
<p>{item.price}</p>
```

Expressions rooted at `item` and `tags` are validated at build time.

## Core sections and expressions

```html
{#if item.inStock}
  <p>{item.name}</p>
{#else}
  <p>Sold out</p>
{/if}

{#for tag in item.tags}
  <span>{tag}</span>
{/for}

{#let label=(item.inStock ? 'In stock' : 'Sold out')}
  <strong>{label}</strong>
{/let}
```

Useful built-ins:

- Elvis/default: `{item.name ?: 'Unknown'}`
- Ternary: `{item.inStock ? 'yes' : 'no'}`
- Current data namespace: `{data:item.name}`
- Raw output: `{htmlSnippet.raw}`

## Layouts with `include` and `insert`

Base template:

```html
<html>
<head><title>{#insert title}Default title{/}</title></head>
<body>{#insert}No body{/}</body>
</html>
```

Child template:

```html
{#include base}
  {#title}Catalog{/title}
  <main>{item.name}</main>
{/include}
```

## User tags

Tag template `templates/tags/badge.html`:

```html
<span class="badge badge-{kind}">{it}</span>
```

Usage:

```html
{#badge item.status kind='success' /}
```

By default, tags are isolated from the caller context. Pass `_unisolated` only when the tag must see parent data.

## Fragments

Template:

```html
{#fragment item_row}
<tr>
  <td>{item.name}</td>
  <td>{item.price}</td>
</tr>
{/fragment}
```

Java:

```java
String row = itemTemplate.getFragment("item_row")
        .data("item", item)
        .render();
```

Use fragments for partial page updates and reusable subtrees.

## Template extension methods

```java
import io.quarkus.qute.TemplateExtension;

@TemplateExtension
class ItemTemplateExtensions {
    static BigDecimal discountedPrice(Item item) {
        return item.price().multiply(new BigDecimal("0.9"));
    }
}
```

Template use:

```html
{item.discountedPrice}
```

## `@TemplateData` and `@TemplateEnum`

```java
import io.quarkus.qute.TemplateData;
import io.quarkus.qute.TemplateEnum;

@TemplateData
class ItemView {
    public String name;
    public BigDecimal price;
}

@TemplateEnum
enum Status {
    DRAFT,
    PUBLISHED
}
```

Template use:

```html
{item.name}
{#if status == Status:PUBLISHED}Live{/if}
```

These generated resolvers are native-friendly and avoid reflection.

## Inject beans directly in templates

```java
@Named
@ApplicationScoped
class PriceFormatter {
    String currency(BigDecimal value) {
        return "$" + value;
    }
}
```

```html
{inject:priceFormatter.currency(item.price)}
```

`cdi:` and `inject:` expressions are validated at build time.

## Message bundles

```java
import io.quarkus.qute.i18n.Message;
import io.quarkus.qute.i18n.MessageBundle;

@MessageBundle
interface Messages {
    @Message("Hello {name}!")
    String hello(String name);
}
```

Template use:

```html
{msg:hello(user.name)}
```

## Programmatic rendering

```java
String html = report.data("items", items)
        .data("generatedAt", LocalDateTime.now())
        .render();
```

Async options are also available via `renderAsync()`, `createUni()`, and `createMulti()`.

## Engine customization

```java
import io.quarkus.qute.EngineBuilder;

class QuteCustomizer {
    void configure(@Observes EngineBuilder builder) {
        builder.addValueResolver(MyResolver.INSTANCE);
    }
}
```

Use `@EngineConfiguration` for custom resolvers or section helpers that must also participate in build-time validation.
