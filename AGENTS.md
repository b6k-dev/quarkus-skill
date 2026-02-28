# Overview
Skill repository for the Quarkus platform - structured reference docs to be used by AI/LLM.

# Reference File Structure
Every module follows 5-file pattern:

- README.md — Overview, when to use
- api.md — Runtime API reference
- configuration.md — Configuration reference 
- patterns.md — Usage patterns
- gotchas.md — Pitfalls, limitations

# General Guidelines

- Module IDs in `skill/quarkus/SKILL.md` are routing keys; they must match folder names under `skill/quarkus/references/`.
- Keep `api.md` concise and example-first; keep the examples condensed and minimal yet informative; long narrative text quickly becomes noisy for this module.
- Avoid hardcoding spec versions in reference prose unless they are intentionally pinned and regularly maintained.
