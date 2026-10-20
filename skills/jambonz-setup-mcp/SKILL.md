---
name: jambonz-setup-mcp
description: Set up the jambonz MCP server in AI coding tools. Use when the user wants to connect Claude Code, Cursor, Codex, Windsurf, or VS Code Copilot to the jambonz MCP server at https://mcp-server.jambonz.app/mcp, or run the server locally via stdio. Covers HTTP and stdio transport, per-IDE configuration, verification, and troubleshooting.
license: MIT
---

# Setting Up the jambonz MCP Server

The jambonz MCP server lets AI coding tools query jambonz schemas, SDK examples, and the developer toolkit on demand — no manual docs hunting, no copy-pasting verb definitions.

## What the Server Provides

Three tools, exposed via the Model Context Protocol:

- **`jambonz_developer_toolkit`** — The full developer guide. Verb model, transport modes, callback patterns, schema index. Load first when starting any jambonz work.
- **`get_jambonz_schema`** — On-demand JSON Schema for any verb, component, callback, or guide (e.g. `verb:say`, `component:synthesizer`, `callback:actionHook`, `guide:node-sdk`).
- **`get_sdk_example`** — Source code for any `@jambonz/sdk` example (`agent`, `ivr-menu`, `dial`, `call-recording`, etc.).

## Two Transports

| Transport | When to use | Endpoint |
|---|---|---|
| **HTTP (remote)** | Default for most users. No install. Stays current automatically. | `https://mcp-server.jambonz.app/mcp` |
| **stdio (local)** | When the IDE requires stdio, or when you want to run offline / behind a firewall. | `npx -y @jambonz/mcp-schema-server` |

## Setup by IDE

### Claude Code

One command:

```bash
claude mcp add --transport http jambonz https://mcp-server.jambonz.app/mcp
```

Or for stdio:

```bash
claude mcp add jambonz -- npx -y @jambonz/mcp-schema-server
```

Confirm with `claude mcp list` — `jambonz` should appear with status `connected`.

### Cursor

Edit `~/.cursor/mcp.json` (global) or `<project>/.cursor/mcp.json` (project-scoped):

```json
{
  "mcpServers": {
    "jambonz": {
      "url": "https://mcp-server.jambonz.app/mcp"
    }
  }
}
```

For stdio:

```json
{
  "mcpServers": {
    "jambonz": {
      "command": "npx",
      "args": ["-y", "@jambonz/mcp-schema-server"]
    }
  }
}
```

Reload Cursor; the server appears under Settings → MCP.

### VS Code (with Copilot or Continue / other MCP-aware extensions)

Add to `.vscode/mcp.json` in the project (or the global equivalent your extension reads):

```json
{
  "servers": {
    "jambonz": {
      "type": "http",
      "url": "https://mcp-server.jambonz.app/mcp"
    }
  }
}
```

For stdio, use `"type": "stdio"` with `"command"` and `"args"` as above.

### Codex

Codex reads `~/.codex/config.toml`. Add:

```toml
[mcp_servers.jambonz]
command = "npx"
args = ["-y", "@jambonz/mcp-schema-server"]
```

(Codex currently uses stdio MCP. If/when HTTP support lands, switch to the URL form.)

### Windsurf

Edit `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "jambonz": {
      "serverUrl": "https://mcp-server.jambonz.app/mcp"
    }
  }
}
```

Restart Windsurf.

### Any other MCP-aware tool

For HTTP transport, point at `https://mcp-server.jambonz.app/mcp`. For stdio, run `npx -y @jambonz/mcp-schema-server`.

## Verification

After setup, three tools should be available:

- `jambonz_developer_toolkit`
- `get_jambonz_schema`
- `get_sdk_example`

Quickest check: prompt the assistant with *"Use the jambonz_developer_toolkit tool and summarize what it returns."* If you get a verb list and schema index, the connection is working.

## Pairing with the jambonz Skills

The MCP server and the `jambonz-skills` plugin complement each other — install both.

**Skills tell the model what to do and why. The MCP server tells the model the exact JSON shape to write.** Skills capture judgment (decision trees, patterns, gotchas) and are loaded into the model's context. The MCP server captures truth (live schemas, SDK example source) and is called as tools at runtime.

You can use them separately — skills alone for planning/offline work, MCP alone for one-off schema lookups — but real development wants both: skills decide, MCP verifies.

Install the skills with:

```bash
npx skills add jambonz/skills
```

Or copy the `skills/` directory of the [jambonz/skills repo](https://github.com/jambonz/skills) into your agent's skills path manually.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Tools don't appear after config change | IDE didn't reload | Restart the IDE (or its MCP subsystem) |
| `connection refused` on HTTP | Network / proxy issue | Try stdio transport (`npx -y @jambonz/mcp-schema-server`) |
| `command not found: npx` on stdio | Node not installed | Install Node.js ≥ 18 |
| Tools appear but always error | Stale package version | Pin to latest: `npx -y @jambonz/mcp-schema-server@latest` |
| AI doesn't call the tools | Description not triggering | Explicitly prompt: *"Call `get_jambonz_schema('verb:dial')` and show the result."* |
