# Quarkus Service Communication REST Gotchas

Common outbound REST client pitfalls, symptoms, and fixes.

## Injection and client registration

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Unsatisfied dependency for a client interface | The interface is missing `@RegisterRestClient` or the injection point is missing `@RestClient` | Add both annotations so Quarkus can create and select the client bean |
| Client works in one place but not another | The bean scope or qualifier changed unexpectedly | Keep client injection points explicit and scope changes deliberate |

## Configuration mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Startup or first call fails because the base URL is missing | No `quarkus.rest-client.<client>.url` property exists for the client | Set the URL for the `configKey` or fully qualified client name |
| Refactor breaks configuration silently | Properties are keyed by the interface name and the class moved packages | Use `configKey` so config names stay stable |
| Requests go to the wrong host in tests or multi-tenant flows | `@Url` override and configured URL are mixed carelessly | Use configured URLs by default and reserve `@Url` for explicit per-call overrides |

## Blocking and reactive mismatch

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Event-loop blocked warnings appear under load | A blocking client call runs inside a reactive request path | Use `Uni`-returning client methods or shift blocking work off the event loop |
| Reactive code loses its benefit | The flow calls `.await().indefinitely()` too early | Keep the chain reactive until the boundary that can safely block |

## Response and error mapping

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Useful error payload details are lost | Exceptions are thrown before the response body is mapped intentionally | Use `RestResponse<T>` or a client exception mapper when status and body both matter |
| All non-2xx responses become generic failures | No client-specific exception mapping exists | Add `@ClientExceptionMapper` or a registered response mapper for expected statuses |
| Success payload mapping fails on schema drift | The downstream adds fields or changes media type unexpectedly | Validate media types explicitly and configure JSON strictness where needed |

## Headers, auth, and transport settings

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Downstream rejects requests as unauthorized | Required auth or tenant headers are not propagated | Centralize header generation with `@ClientHeaderParam`, filters, or token propagation support |
| HTTPS calls fail only in internal environments | The service certificate chain is not trusted or hostname verification does not match | Configure the correct trust store and verify host settings |
| High concurrency calls stall unexpectedly | Pool size, proxy, or timeout settings were left at unsuitable defaults | Set connection pool, proxy, connect timeout, and read timeout intentionally per client |

## Testing blind spots

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Unit tests pass but the real integration breaks | The test mocked service behavior but never exercised HTTP serialization or headers | Add a stub-server test for request shape, payload mapping, and error handling |
| Stub tests are flaky | Shared ports or non-profile-scoped URLs leak across tests | Keep test URLs profile-scoped and isolate stub server lifecycle |
