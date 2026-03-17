# Quarkus CDI Events Gotchas

Common CDI event pitfalls, symptoms, and fixes.

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Async observer never runs | Event was fired with `fire(...)` but observer uses `@ObservesAsync` | Use `fireAsync(...)` for async observers, or switch the observer to `@Observes` |
| Async observer exception seems lost | `fireAsync()` completion is ignored | Observe the returned `CompletionStage` and handle exceptional completion |
| Welcome email or external side effect happens even though the transaction rolled back | Observer ran in the default `IN_PROGRESS` phase | Use `@Observes(during = TransactionPhase.AFTER_SUCCESS)` for post-commit side effects |
| Expecting CDI events to reach another service | CDI events are same-process only and not durable | Use `messaging` for broker-backed delivery |
| Using CDI async observers as a reactive pipeline stage | `@ObservesAsync` is thread-offloaded dispatch, not a stream operator | Keep observers `void` and use reactive APIs for stream composition |
