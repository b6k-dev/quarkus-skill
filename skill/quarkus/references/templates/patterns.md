# Quarkus Templates Usage Patterns (Qute)

Use these patterns for repeatable Qute and server-side rendering workflows.

## Pattern: Bootstrap server-side HTML with REST + Qute

When to use:

- You are building HTML endpoints rendered on the server.

Command:

```bash
quarkus create app com.acme:catalog-ui --extension='rest,qute,rest-qute' --no-code
```

If pages should be served directly from `templates/pub`, add `io.quarkiverse.qute.web:quarkus-qute-web`.

## Pattern: Prefer type-safe templates per resource

When to use:

- A resource owns one or more views and you want build-time validation.

Example:

```java
@Path("/orders")
class OrderResource {
    @CheckedTemplate
    static class Templates {
        static native TemplateInstance list(List<OrderView> orders);
        static native TemplateInstance detail(OrderView order);
    }

    @GET
    TemplateInstance list() {
        return Templates.list(service.list());
    }
}
```

Place templates under `src/main/resources/templates/OrderResource/`.

## Pattern: Keep templates declarative with extensions

When to use:

- A view needs computed properties or lightweight formatting.

Example:

```java
@TemplateExtension
class MoneyExtensions {
    static String currency(BigDecimal value) {
        return "$" + value.setScale(2, RoundingMode.HALF_UP);
    }
}
```

```html
{order.total.currency}
```

Prefer this over embedding formatting logic into template sections.

## Pattern: Share layouts with `include` and tags

When to use:

- Multiple pages reuse the same shell, header, or repeated component.

Example:

```html
{#include layout/base}
  {#title}Orders{/title}
  <main>
    {#orderTable orders=orders /}
  </main>
{/include}
```

Use `include` for page layout inheritance and `templates/tags/*` for reusable components.

## Pattern: Render partial updates with fragments

When to use:

- You need a reusable subtree for htmx/AJAX responses or repeated server-side partials.

Example:

```html
{#fragment row}
<tr>
  <td>{order.id}</td>
  <td>{order.status}</td>
</tr>
{/fragment}
```

```java
String html = ordersTemplate.getFragment("row")
        .data("order", order)
        .render();
```

## Pattern: Render emails, reports, and exports outside HTTP

When to use:

- Output is generated in schedulers, jobs, messaging consumers, or services.

Example:

```java
@ApplicationScoped
class ReportService {
    @Inject
    @Location("reports/daily")
    Template daily;

    String render(List<LineItem> items) {
        return daily.data("items", items).render();
    }
}
```

Return `String` when you need to persist, email, or attach the rendered output.

## Pattern: Localize copy with message bundles

When to use:

- UI strings must support more than one locale.

Example:

```java
@MessageBundle
interface Messages {
    @Message("Hello {name}!")
    String hello(String name);
}
```

```html
{msg:hello(user.name)}
```

Add `src/main/resources/messages/msg_<locale>.properties` files for localized variants.
