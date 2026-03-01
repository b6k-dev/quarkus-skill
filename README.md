# Quarkus Skill

Comprehensive Quarkus platform reference docs for AI/LLM consumption. Covers implementation, troubleshooting, and architecture decisions across core Quarkus workflows and extension ecosystems.

## Install

### AI Agents

Add the skill to your AI coding assistant for richer context:

```bash
npx skills add b6k-dev/quarkus-skill
```

This works with many different tools including: Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot, Goose, OpenCode, Windsurf.


### Opencode 

Local installation (current project only):

```bash
curl -fsSL https://raw.githubusercontent.com/b6k-dev/quarkus-skill/main/install.sh | bash
```

Global installation (available in all projects):

```bash
curl -fsSL https://raw.githubusercontent.com/b6k-dev/quarkus-skill/main/install.sh | bash -s -- --global
```

## Usage

Once installed, the skill appears in OpenCode's `<available_skills>` list. The agent can load it automatically when working on Quarkus tasks.

Use the `/quarkus` command to load the skill and get contextual guidance:

```text
/quarkus add REST API with validation and OpenAPI docs
```

### Updating

To update to the latest version:

```text
/quarkus --update-skill
```

## License

MIT - see [LICENSE](LICENSE)
