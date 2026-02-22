# Quarkus CLI Reference

Use this file for command syntax, high-value command groups, and Maven/Gradle equivalence.

## CLI Model

- `quarkus` is a command-line facade over Quarkus build tooling.
- It can execute workflows for Maven, Gradle, and JBang-backed projects.
- Primary value: one stable command style across build systems.

## Using the CLI

```text
quarkus [global-options] [command] [subcommand] [args]
```
If in doubt inspect the behavior with:

```bash
quarkus --help
quarkus <command> --help
```

## Commands reference

```bash
quarkus --help
Usage: quarkus [-ehv] [--refresh] [--verbose] [--config=CONFIG]
               [-D=<String=String>]... [COMMAND]
Options:
      --refresh         Refresh the local Quarkus extension registry cache
      --config=CONFIG   Configuration file
  -h, --help            Display this help message.
  -v, --version         Print CLI version information and exit.
  -e, --errors          Display error messages.
      --verbose         Verbose mode.
  -D=<String=String>    Java properties

Commands:
  create                  Create a new project.
    app                   Create a Quarkus application project.
    cli                   Create a Quarkus command-line project.
    extension             Create a Quarkus extension project
  build                   Build the current project.
  dev                     Run the current project in dev (live coding) mode.
  test                    Run the current project in continuous testing mode.
  extension, ext          Configure extensions of an existing project.
    list, ls              List platforms and extensions.
    categories, cat       List extension categories.
    add                   Add extension(s) to this project.
    remove, rm            Remove extension(s) from this project.
  image                   Build or push project container image.
    build                 Build a container image.
      docker              Build a container image using Docker.
      podman              Build a container image using Podman.
      buildpack           Build a container image using Buildpack.
      jib                 Build a container image using Jib.
      openshift           Build a container image using OpenShift.
    push                  Push a container image.
  deploy                  Deploy application.
    kubernetes            Perform the deploy action on Kubernetes.
    openshift             Perform the deploy action on OpenShift.
    knative               Perform the deploy action on Knative.
    kind                  Perform the deploy action on Kind.
    minikube              Perform the deploy action on minikube.
  registry                Configure Quarkus registry client
    list                  List enabled Quarkus registries
    add                   Add a Quarkus extension registry
    remove                Remove a Quarkus extension registry
  info                    Display project information and verify versions
                            health (platform and extensions).
  update, up, upgrade     Suggest recommended project updates with the
                            possibility to apply them.
  version                 Display CLI version information.
  plugin, plug            Configure plugins of the Quarkus CLI.
    list, ls              List CLI plugins.
    add                   Add plugin(s) to the Quarkus CLI.
    remove                Remove plugin(s) to the Quarkus CLI.
    sync                  Sync (discover / purge) CLI Plugins.
  completion              bash/zsh completion:  source <(quarkus completion)
```

## Project Creation

```bash
quarkus create app
quarkus create app org.acme:my-app
quarkus create app org.acme:my-app:1.0.0
quarkus create app org.acme:my-app --gradle
```

Defaults:

- `groupId=org.acme`
- `artifactId=code-with-quarkus`
- `version=1.0.0-SNAPSHOT`

Version targeting options:

- `-P <groupId:artifactId:version>` for platform BOM
- `-S <platformKey:streamId>` for platform stream

## Extension API

List and search:

```bash
quarkus ext ls
quarkus ext list --concise -i -s jdbc
```

Add/remove:

```bash
quarkus ext add rest
quarkus ext add smallrye-*
quarkus ext rm kubernetes
```

Useful list flags:

- `--name` - display the name (artifactId) only
- `--concise` - display the name (artifactId) and description
- `--full` - display concise information 
- `--origins` - display concise information along with the Quarkus platform release origin of the extension
- `--support-scope` - display support scope associated with each extension (if any) for an offering configured in the registry client configuration
- `-i` - show currently installable only
- `-s <search>` - filter the list

## Dev/Test/Build API

You can run the commands with `--clean` to perform clean as the part of the build, e.g.:

```bash
quarkus dev --clean
```
## Plugin API

Discover/manage plugins:

```bash
quarkus plugin list
quarkus plugin list --installable
quarkus plugin add <name-or-location>
quarkus plugin remove <name>
quarkus plugin sync
```

Supported plugin sources include:

- local executables prefixed with `quarkus-`
- runnable JARs
- JBang aliases prefixed with `quarkus-`
- Maven coordinates to runnable JARs

## Build Tool Equivalence

Typical command translation examples:

| Intent | Quarkus CLI | Maven | Gradle |
|--------|-------------|-------|--------|
| Run dev mode | `quarkus dev` | `./mvnw quarkus:dev` | `./gradlew quarkusDev` |
| Test | `quarkus test` | `./mvnw test` | `./gradlew test` |
| Build package | `quarkus build` | `./mvnw package` | `./gradlew build` |
| Add extension | `quarkus ext add rest` | `./mvnw quarkus:add-extension -Dextension=rest` | `./gradlew addExtension --extensions="rest"` |
