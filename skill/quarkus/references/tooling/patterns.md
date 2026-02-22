# Quarkus CLI Usage Patterns

Use these patterns for repeatable command-line workflows.

## Pattern: Daily Dev Loop

When to use:

- Active feature work on an existing service.

Commands:

```bash
quarkus dev
quarkus test
quarkus build
```

## Pattern: Create a New Service with Explicit Coordinates

When to use:

- Bootstrap a new app and keep metadata explicit in terminal history.

Commands:

```bash
quarkus create app com.acme:orders-service:1.0.0-SNAPSHOT
```

Gradle variant:

```bash
quarkus create app com.acme:orders-service --gradle
```

## Pattern: Discover Then Add Extensions

When to use:

- You know capability area (for example JDBC or messaging) but not exact extension IDs.

Commands:

```bash
quarkus ext list --concise -i -s jdbc
quarkus ext add jdbc-postgresql
```

## Pattern: Build Container Images Without Editing Build Files

When to use:

- Need quick image production with minimal project churn.

Commands:

```bash
quarkus image build docker
quarkus image build jib
quarkus image push --registry=<registry> --registry-username=<username> --registry-password-stdin
```

## Pattern: Switch Deployment Targets Quickly

When to use:

- Same application must be deployed to different local or cluster environments.

Commands:

```bash
quarkus deploy kind
quarkus deploy minikube
quarkus deploy kubernetes
```

## Pattern: Team Automation with Plugins

When to use:

- Team wants reusable custom commands (scaffolding, validation, operational routines).

Commands:

```bash
quarkus plugin list --installable
quarkus plugin add <name-or-location>
quarkus plugin sync
```
