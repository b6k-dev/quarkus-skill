# Quarkus CDI Events Usage Patterns

Use these patterns for repeatable in-process event choreography.

## Pattern: Publish a Local Domain Event After a Successful Transaction

When to use:

- One business action triggers several same-process side effects without hard-coding direct dependencies.

Example:

```java
record UserRegistered(String username) {
}

@ApplicationScoped
class UserService {
    @Inject
    Event<UserRegistered> events;

    @Transactional
    void register(String username) {
        repository.persist(new User(username));
        events.fire(new UserRegistered(username));
    }
}

@ApplicationScoped
class AuditListener {
    void onRegistered(@Observes(during = TransactionPhase.AFTER_SUCCESS) UserRegistered event) {
        audit.write(event.username());
    }
}
```

Use CDI events for local choreography. Move to `messaging` when the event must leave the service.

## Pattern: Keep Optional Side Effects Async and Local

When to use:

- The main transaction should finish without waiting for secondary work such as notifications.

Example:

```java
@ApplicationScoped
class RegistrationNotifier {
    @Inject
    Event<UserRegistered> events;

    CompletionStage<UserRegistered> notifyAsync(UserRegistered event) {
        return events.fireAsync(event);
    }
}

@ApplicationScoped
class WelcomeEmailListener {
    void onRegistered(@ObservesAsync UserRegistered event) {
        mailer.send(event.username());
    }
}
```

If failure handling, retries, or durability become important, graduate this flow to broker-backed messaging.

## Pattern: Start with CDI Events, Then Escalate to an Outbox

When to use:

- A local event flow later needs reliability across process boundaries.

Example:

```java
@ApplicationScoped
class UserService {
    @Transactional
    void register(String username) {
        userRepository.persist(new User(username));
        OutboxRecord.persist(OutboxRecord.userRegistered(username));
    }
}
```

Persist the outbox record in the same transaction as the business write, then publish it to a broker from a separate background component.
