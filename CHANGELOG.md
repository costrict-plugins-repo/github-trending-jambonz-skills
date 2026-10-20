# Changelog

## 0.2.0

Restructured into a Claude plugin with four specialized skills. All content from the previous monolithic `jambonz` skill is preserved; only the boundaries changed.

### New layout

- Repo is now a Claude plugin (`.claude-plugin/plugin.json` manifest).
- Skills live under `skills/<name>/`.

### Skills

- **`jambonz`** — Decision-making skill (verb selection, transport choice, gotchas). Trimmed from ~243 lines to a focused router; pointers to sibling skills replace the inline reference index.
- **`jambonz-recipes`** *(new)* — Holds the four implementation reference files (voice-ai, ivr, call-control, env-vars) plus the SDK + raw-JSON building patterns moved from the old `jambonz/SKILL.md`. Reference files renamed for cleaner triggering:
  - `voice-ai-guide.md` → `voice-ai.md`
  - `ivr-patterns.md` → `ivr.md`
  - `env-vars-and-config.md` → `env-vars.md`
  - `call-control.md` (unchanged name)
- **`jambonz-starters`** *(new)* — Index of the 18+ runnable starter apps in `@jambonz/node-sdk/examples/`, with one-line clone commands.
- **`jambonz-setup-mcp`** *(new)* — Per-IDE setup instructions for wiring the jambonz MCP server into Claude Code, Cursor, Codex, Windsurf, and VS Code, plus verification and troubleshooting.

### Why

Splitting by intent gives each skill a distinct triggering description, so AI coding tools load only what's relevant to the current task. Mirrors the multi-skill plugin convention used by Anthropic's first-party plugins and Deepgram's `deepgram/skills`.

## 0.1.0

Initial release of the monolithic `jambonz` skill.
