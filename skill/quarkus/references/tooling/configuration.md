# Quarkus CLI Configuration Reference

Use this file for installation and CLI environment setup.

## Installation Matrix

Supported installation methods:

| Method | Platforms | Install | Update |
|--------|-----------|---------|--------|
| SDKMAN! | Linux, macOS | `sdk install quarkus` | `sdk upgrade quarkus` |
| Homebrew | Linux, macOS | `brew install quarkusio/tap/quarkus` | `brew update && brew upgrade quarkus` |
| Chocolatey | Windows | `choco install quarkus` | `choco upgrade quarkus` |
| Scoop | Windows | `scoop install quarkus-cli` | `scoop update quarkus-cli` |
| JBang | Linux, macOS, Windows | `jbang app install --fresh --force quarkus@quarkusio` | same command |

Verification:

```bash
quarkus --version
```
## Shell Configuration

Enable completion for current shell session:

```bash
source <(quarkus completion)
```

Optional alias:

```bash
alias q=quarkus
complete -F _complete_quarkus q
```

Persist these in shell profile files if needed.

## Plugin Catalog Configuration

Plugin metadata catalogs:

- User scope: `~/.quarkus/cli/plugins/quarkus-cli-catalog.json`
- Project scope: `<project>/.quarkus/cli/plugins/quarkus-cli-catalog.json`

Precedence rule:

- Project catalog overrides user catalog for that project.
- Use `--user` when an operation must target user scope explicitly.

## Registry Credentials for Image Push

Example flags:

```bash
quarkus image push --registry=<registry> --registry-username=<username> --registry-password-stdin
```

Prefer stdin-driven secrets to avoid exposing passwords in command history.
