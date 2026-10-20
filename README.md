# jambonz Agent Skills

[Agent Skills](https://agentskills.io) for building voice applications on [jambonz](https://jambonz.org), packaged as a single Claude plugin.

## What's in the plugin

| Skill | Triggers when... |
|---|---|
| [`jambonz`](skills/jambonz/) | The agent is planning or reasoning about a jambonz call flow — picking verbs, choosing transport, avoiding gotchas |
| [`jambonz-recipes`](skills/jambonz-recipes/) | The agent is writing code for a specific feature — voice AI, IVR, dial/transfer/queue/conference/recording, env_vars |
| [`jambonz-starters`](skills/jambonz-starters/) | The user wants to clone a complete runnable starter app |
| [`jambonz-setup-mcp`](skills/jambonz-setup-mcp/) | The user is wiring the jambonz MCP server into Claude Code, Cursor, Codex, Windsurf, or VS Code |

## Skills vs. the MCP server — when to use which

These skills and the [jambonz MCP server](https://www.npmjs.com/package/@jambonz/mcp-schema-server) complement each other. Use both for the best experience.

|  | Skills | MCP server |
|---|---|---|
| **What it is** | Markdown guidance loaded into the model's context | Tools the model calls at runtime |
| **Provides** | Decisions, patterns, conventions, gotchas | Live JSON schemas, SDK example source, the canonical developer toolkit |
| **Updates** | When you re-run `npx skills add jambonz/skills` | Automatically — always reflects the current schema |
| **Analogy** | The cookbook | The pantry inventory |

**Skills tell the model what to do and why. The MCP server tells the model the exact JSON shape to write.** Skills capture judgment; MCP captures truth.

You can use them separately:

- **Skills alone** is fine for planning, learning, and offline work — but schemas in the prose may go stale.
- **MCP alone** is fine for one-off schema lookups — but the model won't know which verb to pick or which transport to use.
- **Both together** is what real development wants: skills decide, MCP verifies.

### Install both (~30 seconds)

```bash
# 1. Install the skills
npx skills add jambonz/skills

# 2. Connect the MCP server (Claude Code example)
claude mcp add --transport http jambonz https://mcp-server.jambonz.app/mcp
```

For Cursor, Codex, Windsurf, or VS Code, the [`jambonz-setup-mcp`](skills/jambonz-setup-mcp/) skill has copy-paste config for each.

## Installation

### As a Claude Code marketplace (recommended for Claude Code)

This repo is itself a Claude Code plugin marketplace. Add it and install in two commands:

```bash
/plugin marketplace add jambonz/skills
/plugin install jambonz-skills@jambonz
```

Updates ship instantly when we tag a new release — re-run `/plugin install jambonz-skills@jambonz` to pick them up.

### Using the agentskills.io CLI (cross-IDE)

Works with Claude Code, Cursor, VS Code Copilot, and other [compatible agents](https://agentskills.io):

```bash
npx skills add jambonz/skills
```

Add `--global` to install for all projects:

```bash
npx skills add jambonz/skills --global
```

### Manual installation

Copy any of the `skills/<name>/` folders into your agent's skills directory:

| Agent | Path |
|-------|------|
| Claude Code (project) | `.claude/skills/<name>/` |
| Claude Code (global) | `~/.claude/skills/<name>/` |
| Cursor | `.cursor/skills/<name>/` |
| VS Code Copilot | `.vscode/skills/<name>/` |
| Cross-client | `.agents/skills/<name>/` |

## Language support

- **JavaScript or TypeScript** via [`@jambonz/sdk`](https://github.com/jambonz/node-sdk) — preferred. Fluent verb builder, type safety, and tool-calling helpers all live here. **Default to TypeScript** when generating new apps; use JavaScript only if the user asks for it.
- **Python** via raw JSON verb arrays over HTTP webhooks or WebSocket.

## Source

Maintained at [github.com/jambonz/skills](https://github.com/jambonz/skills) alongside the [SDK](https://github.com/jambonz/node-sdk) and [MCP server](https://github.com/jambonz/mcp-schema-server).

## License

MIT
