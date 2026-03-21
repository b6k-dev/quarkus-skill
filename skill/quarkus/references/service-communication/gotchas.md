# Service Communication Gotchas

Common transport-selection mistakes, symptoms, and fixes.

| Symptom | Likely cause | Fix |
|---|---|---|
| A request/response call keeps getting stretched into retries, replay, or buffering logic | Synchronous RPC is being used for an async integration problem | Use `messaging` instead |
| Teams keep arguing about payload shape and compatibility | Transport was chosen before the contract and consumers were understood | Decide first whether HTTP resource semantics or protobuf RPC semantics are the better fit |
| The service layer is tightly coupled to transport annotations everywhere | Boundary adapters were skipped | Keep REST client and gRPC stub usage close to the integration edge |
