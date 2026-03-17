# Quarkus Vert.x Event Bus Gotchas

Common Event Bus pitfalls, symptoms, and fixes.

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Request times out waiting for a reply | No consumer is listening on the address, or it does not return a reply | Add a matching `@ConsumeEvent` consumer or use fire-and-forget messaging instead |
| Only one consumer receives a message but all were expected to | `send` uses point-to-point delivery | Use `publish` for local pub/sub fan-out |
| Handler blocks event-loop threads | Consumer performs blocking work without `@Blocking` | Add `@Blocking`, redesign around async APIs, or move work off the bus path |
| Event bus used for cross-service durable events | Event bus is local or clustered transport, not a broker with replay or consumer groups | Use Reactive Messaging with Kafka, RabbitMQ, AMQP, or another broker connector |
| Custom object works locally but fails with explicit transport assumptions | Default codec only covers local delivery expectations | Register and use an explicit codec, or keep payloads simple |
| `@RunOnVirtualThread` has no effect on async return types | Virtual-thread event-bus handlers require blocking signatures | Use `void` or plain return types with `@RunOnVirtualThread`, or keep the handler reactive |
| Startup fails on `@ConsumeEvent("${...}")` | Config property is missing and no default was provided | Add the property or include a default value in the expression |
| Expecting back-pressure, replay, or stream operators | Event Bus delivers messages, not broker-managed streams | Use Reactive Messaging for stream processing and broker-backed flow control |
