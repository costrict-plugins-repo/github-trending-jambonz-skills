---
name: jambonz-recipes
description: Implementation recipes for jambonz voice app features. Use when writing code to build conversational voice AI agents (s2s, agent verb, vendor shortcuts, tool calling), collect input via IVR menus and gather, bridge or transfer calls (dial, SIP, queue, conference, recording, SIPREC), or configure applications via env_vars. Contains @jambonz/sdk JavaScript/TypeScript patterns (preferred) and raw-JSON examples for Python. Pair with the jambonz skill (decision-making) and the jambonz MCP server (schema lookups).
license: MIT
---

# jambonz Implementation Recipes

This skill holds the runnable patterns for building jambonz voice apps. Use the parent `jambonz` skill to decide *what* to build and which transport to pick. Use this skill once you're writing code for a specific feature.

## When to Load Each Reference

The four reference files below are loaded on-demand. Read only the ones relevant to the task.

- **[references/voice-ai.md](references/voice-ai.md)** — Load when building s2s or cascaded `agent`-verb voice AI apps. Covers vendor shortcuts (OpenAI, Google, Deepgram, Ultravox, ElevenLabs), s2s vs agent trade-offs, tool/function calling, eventHook patterns, TTS token streaming.
- **[references/ivr.md](references/ivr.md)** — Load when building IVR menus or `gather`-based input collection. Covers speech/DTMF/mixed input, multi-level menus, timeout and retry, default recognizer/synthesizer.
- **[references/call-control.md](references/call-control.md)** — Load when building apps with `dial`, transfer, queuing, conference, recording, or mid-call control. Covers dial targets, SIP operations (sip-decline / sip-refer / sip-request), enqueue/dequeue, conference, REST API control, inject commands, SIPREC recording.
- **[references/env-vars.md](references/env-vars.md)** — Load when the app needs configurable parameters. Covers the two-step declare+read pattern, schema property types, the "only on initial call" gotcha, session.locals state.

## Building the Application

### With @jambonz/sdk (JavaScript or TypeScript — preferred)

**Default to TypeScript** when generating new code. Only emit JavaScript if the user explicitly asks for it. Both work with the same `@jambonz/sdk` package and the same imports — only file extensions and type annotations differ.

Install: `npm install @jambonz/sdk` (plus `express` for webhook apps).

**Webhook pattern**:
```typescript
import express from 'express';
import { WebhookResponse } from '@jambonz/sdk/webhook';

const app = express();
app.use(express.json());

app.post('/incoming', (req, res) => {
  const jambonz = new WebhookResponse();
  jambonz.say({ text: 'Hello!' }).hangup();
  res.json(jambonz);
});

// Required: call status handler
app.post('/call-status', (req, res) => {
  console.log(`Call ${req.body.call_sid} status: ${req.body.call_status}`);
  res.sendStatus(200);
});

app.listen(3000);
```

**WebSocket pattern**:
```typescript
import http from 'http';
import { createEndpoint } from '@jambonz/sdk/websocket';

const server = http.createServer();
const makeService = createEndpoint({ server, port: 3000 });
const svc = makeService({ path: '/' });

svc.on('session:new', (session) => {
  // Bind actionHook handlers BEFORE send()
  session.on('/gather-result', (evt) => {
    session.say({ text: `You said: ${evt.speech?.alternatives?.[0]?.transcript}` })
      .reply();  // reply() for all subsequent responses
  });

  session
    .gather({ input: ['speech'], actionHook: '/gather-result', timeout: 10,
      say: { text: 'Say something.' } })
    .send();  // send() — initial response only, exactly once
});
```

**Critical SDK rules**:
- `.send()` — use ONCE for the initial verb array (response to `session:new`)
- `.reply()` — use for ALL subsequent responses (actionHook events)
- Bind actionHook handlers BEFORE calling `.send()`
- Use `session.locals` for call-scoped state that persists across hooks
- Use `session.data.env_vars` (WS) or `req.body.env_vars` (webhook) for configurable values — never `process.env`

### Without the SDK (Python)

jambonz verbs are plain JSON. Python apps build jambonz apps by serving HTTP for webhooks (via Flask, FastAPI, etc.) or connecting via WebSocket for streaming. The verb JSON format is identical to what the TypeScript SDK produces.

**Webhook**: HTTP server receives POST with call data, returns JSON array of verbs:
```json
[
  { "verb": "say", "text": "Hello!" },
  { "verb": "gather", "input": ["speech"], "actionHook": "/result", "timeout": 10,
    "say": { "text": "Say something." } }
]
```

**WebSocket**: Connect to jambonz WS endpoint, receive JSON messages, send JSON verb arrays back. See the WebSocket Protocol section of AGENTS.md (via the MCP `jambonz_developer_toolkit` tool) for the message format.

Use `get_jambonz_schema` from the jambonz MCP server to look up the exact JSON structure for any verb.

### Both modes

- Webhook apps **must** include a call-status POST handler (logs lifecycle events, returns 200).
- Use jambonz application environment variables for configurable values — see [references/env-vars.md](references/env-vars.md).

## Picking the Right Recipe

| Task | Reference | MCP follow-up |
|---|---|---|
| Voice AI agent with one vendor (e.g. OpenAI Realtime) | `voice-ai.md` | `get_jambonz_schema("verb:openai_s2s")` |
| Voice AI agent with runtime vendor selection | `voice-ai.md` | `get_jambonz_schema("verb:s2s")` |
| LLM tool/function calling | `voice-ai.md` | `get_sdk_example("agent")` |
| Cascaded agent (STT + LLM + TTS as separate components) | `voice-ai.md` | `get_jambonz_schema("verb:agent")` |
| Single-level IVR | `ivr.md` | `get_jambonz_schema("verb:gather")` |
| Multi-level menu | `ivr.md` | `get_sdk_example("ivr-menu")` |
| Bridge to phone number or SIP endpoint | `call-control.md` | `get_jambonz_schema("verb:dial")` |
| Blind SIP transfer | `call-control.md` | `get_jambonz_schema("verb:sip-refer")` |
| Call queueing with hold music | `call-control.md` | `get_sdk_example("queue-with-hold")` |
| Conference | `call-control.md` | `get_jambonz_schema("verb:conference")` |
| Recording (SIPREC) | `call-control.md` | `get_jambonz_schema("verb:siprec")` |
| Mid-call control from REST | `call-control.md` | `get_jambonz_schema("component:command")` |
| Runtime configuration via env_vars | `env-vars.md` | (no schema needed — declare in app settings) |
