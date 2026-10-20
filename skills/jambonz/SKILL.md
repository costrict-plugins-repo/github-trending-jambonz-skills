---
name: jambonz
description: Build voice applications on jambonz, a voice AI orchestration platform and CPaaS. Use when planning or reasoning about jambonz call flows — selecting verbs (say/gather/dial/agent/transcribe/s2s), choosing between webhook and WebSocket transport, understanding session lifecycle, or avoiding common pitfalls. Works with @jambonz/sdk (JavaScript or TypeScript, preferred) or raw JSON from Python. Pair with jambonz-recipes for implementation patterns, jambonz-starters for runnable scaffolds, and the jambonz MCP server for schema lookups.
license: MIT
---

# jambonz Voice Application Skill

jambonz is an open-source CPaaS (Communications Platform as a Service) for building voice and messaging applications. It handles telephony infrastructure — SIP, carriers, phone numbers, media processing — so you can focus on application logic. Your application controls calls by returning **arrays of verbs** — JSON instructions that execute sequentially.

This skill covers **decision-making**: which verb to pick, which transport to use, what mistakes to avoid. For implementation code, load the `jambonz-recipes` skill. For runnable starter apps, load `jambonz-starters`. For schema lookups, use the jambonz MCP server.

## Using This Skill with the MCP Server

1. **Before writing any jambonz code**, call `jambonz_developer_toolkit` for the complete SDK guide and schema index.
2. **For a verb's exact properties**, call `get_jambonz_schema` with the verb, component, or callback name.
3. **For a working example**, call `get_sdk_example` with the example name.

If MCP tools are unavailable, read `AGENTS.md` from the repo root and schema files from `schema/verbs/`, `schema/components/`, `schema/callbacks/`.

**Workflow**: Use this skill to decide *what* to build → use MCP tools to look up *how* to build it → load `jambonz-recipes` for the implementation pattern.

## Server Versions

jambonz has two editions: **v0.9.x (open source)** and **v10.x (commercial)**. Each verb schema includes a `minVersion` field. Only ask the user about their server version if a verb requires `minVersion` higher than `0.9.6`. If all verbs needed have `minVersion: "0.9.6"`, the code works on both editions.

## Transport Selection

Choose the transport based on what the application needs.

### Use WebSocket when:
- Using any speech-to-speech verb (`openai_s2s`, `google_s2s`, `deepgram_s2s`, `ultravox_s2s`, `elevenlabs_s2s`, `s2s`) or the cascaded `agent` verb — **mandatory**
- Streaming raw audio (`stream` verb with bidirectional audio)
- Using TTS token streaming
- Building complex conversational flows with session state
- Needing bidirectional real-time events or async mid-call control from the app

### Use Webhook when:
- Simple IVR menus (gather + say)
- Call routing (dial to a number or SIP endpoint)
- Voicemail or basic record-and-hang-up
- Any straightforward request-response pattern

**Rule**: If ANY verb in the application requires WebSocket, the entire application must use WebSocket transport. The verb JSON structure is identical in both modes — only the transport differs.

## Verb Selection by Task

Use this decision tree to pick the right verb(s) for the task. For implementation details on any verb listed here, load the `jambonz-recipes` skill.

### "Build a voice AI agent / chatbot"

The user wants a caller to have a conversation with an LLM.  There are two main patterns:
- use a speech-to-speech model, where the entire conversation is orchestrated by one model (`openai_s2s`, `google_s2s`, `deepgram_s2s`, `ultravox_s2s`, `elevenlabs_s2s`, or generic `s2s`)
- use a cascaded voice pipeline by combining separate STT + LLM + TTS verbs (`agent`)

**Never use `llm` in generated code** — it is a legacy name. Use either a vendor shortcut or `s2s`.

### "Build an IVR menu / collect input"

Use `gather` with nested `say` (for TTS prompt) or `play` (for audio file). Supports speech recognition, DTMF digits, or both.

### "Transfer / connect / bridge a call"

- **Bridge to another party**: `dial` with a target (phone number, SIP URI, registered user, Teams user)
- **Blind SIP transfer**: `sip-refer`
- **Transfer to a different application/webhook**: `redirect`

### "Queue calls / hold music"

- **Put caller in queue**: `enqueue` (with `waitHook` for hold music)
- **Picks up caller from queue**: `dequeue`

### "Record the call"

Two options:
- Enable recording at the account level in the jambonz portal (recordings are sent to user's configured cloud storage).
- Use a third-party SIPREC server via inject commands (WebSocket) or REST API (webhook). 

### "Stream raw audio for custom processing"

Use `stream` (preferred name; `listen` is a synonym — always use `stream`).  This is also known as "bidirectional streaming".  Customers that have their built-out back end for media processing and AI often prefer this low-level access to the audio.

### "Transcribe the call in real-time"

Use `transcribe` with a `transcriptionHook`.

### "Play audio or speak text"

- **TTS**: `say` (supports SSML, multiple voices)
- **Audio file**: `play` (from URL)
- **Background audio track**: `dub` (mixes alongside the call)

### "Reject an incoming call"

Use `sip-decline` with a SIP status code.

## Common Gotchas

1. **Using `llm` verb name** — Always use vendor shortcuts (`openai_s2s`, etc.) or `s2s`. Never `llm`.
2. **Using `listen` verb name** — Always use `stream`. They are synonyms; `stream` is preferred.
3. **`.send()` vs `.reply()` confusion** — `.send()` is the initial response only. `.reply()` is for all actionHook responses. Using `.send()` in an actionHook handler will fail.
4. **Using `process.env`** — jambonz apps should generally use application environment variables (`session.data.env_vars` / `req.body.env_vars`), not `process.env`.
5. **`env_vars` only on initial call** — The `env_vars` object is only present in the first webhook POST or `session:new`. Store values in a variable if needed in actionHook handlers.
6. **Webhook transport for s2s/agent apps** — These verbs require WebSocket. Always use `createEndpoint` from `@jambonz/sdk/websocket`.
7. **ElevenLabs: passing `model` or `messages`** — ElevenLabs uses `agent_id` auth. The model and prompt are configured in the ElevenLabs dashboard. Pass `llmOptions: {}`.
8. **Marks silently failing** — Marks require `bidirectionalAudio: { enabled: true, streaming: true }` on the listen/stream verb. Without streaming mode, marks are accepted but never fire.
9. **Not binding actionHook listeners before `.send()`** — In WebSocket mode, if no listener is bound for an actionHook, the SDK auto-replies with an empty verb array, which usually means the call hangs up unexpectedly.
10. **Forgetting the `/call-status` handler** — Webhook apps must handle call status POST requests. Missing this causes errors in the jambonz logs.

## Project Scaffolding

- **Language**: Default to **TypeScript**. Use JavaScript only when the user explicitly asks for it. Both work with `@jambonz/sdk`.
- **Simple apps (1-2 routes)**: Single file. This is the default and works fine for production.
- **Complex apps (3+ routes)**: Standard `src/app.ts` + `src/routes/` layout (use `.js` for JavaScript projects).
- **Dependencies**: `@jambonz/sdk` always. Add `express` for webhook apps. WebSocket apps need no additional deps.
- **TypeScript config**: `"module": "nodenext"`, `"moduleResolution": "nodenext"`.

For complete runnable scaffolds, load the `jambonz-starters` skill — it indexes 17+ ready-to-clone starter apps.

## Sibling Skills

This skill is part of the `jambonz-skills` plugin. Load these companion skills based on the task:

- **`jambonz-recipes`** — Load when implementing a specific feature. Contains @jambonz/sdk patterns and raw-JSON examples for voice AI agents, IVR menus, call control (dial/transfer/queue/conference/recording), and env_vars configuration.
- **`jambonz-starters`** — Load when the user wants to clone a complete runnable app rather than assemble one from recipes.
- **`jambonz-setup-mcp`** — Load when wiring the jambonz MCP server into Claude Code, Cursor, Codex, Windsurf, or VS Code Copilot.
