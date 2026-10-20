---
name: jambonz-starters
description: Ready-to-run jambonz starter applications. Use when the user wants to clone or scaffold a complete working voice app rather than assemble one from recipes — voice AI agents (OpenAI Realtime, Deepgram, ElevenLabs, Ultravox, Retell, Bedrock, AssemblyAI), IVR menus, dialers, recording, queuing, real-time translation, and more. Points at the @jambonz/sdk examples directory with one-line clone commands.
license: MIT
---

# jambonz Starter Applications

The `@jambonz/sdk` repository ships 18+ runnable starter apps in [its `examples/` directory](https://github.com/jambonz/node-sdk/tree/main/examples). Each is a self-contained, copy-and-run project you can use as a base.

## How to Choose a Starter

- **Just learning?** → `hello-world` (webhook) — single say + hangup
- **Building a voice AI agent and have a vendor preference?** → vendor-specific s2s starter (see "Voice AI Agents" table below)
- **Building a general voice agent?** → `agent` (fully featured: OpenAI + Deepgram + tool calling)
- **Building an IVR / phone menu?** → `ivr-menu`
- **Routing or transferring calls?** → `dial` or `two-stage-dialer`
- **Need to record calls?** → `call-recording` or `listen-record`
- **Support center / contact queue?** → `queue-with-hold`
- **Something exotic?** → see the full table below

## Cloning a Starter

Use `degit` to grab a single example folder without the rest of the repo:

```bash
npx degit jambonz/node-sdk/examples/<starter-name> my-app
cd my-app
npm install
```

Then read the starter's `app.ts` (or `ws-app.ts` for WebSocket starters) and `package.json` to see the dependencies and entry point.

## Voice AI Agents

| Starter | Transport | What it shows |
|---|---|---|
| [`agent`](https://github.com/jambonz/node-sdk/tree/main/examples/agent) | WebSocket | Full voice agent: OpenAI LLM + Deepgram STT/TTS + Silero VAD + tool calling (`get_weather`) |
| [`voice-agent`](https://github.com/jambonz/node-sdk/tree/main/examples/voice-agent) | WebSocket | Generic conversational AI voice agent powered by an LLM |
| [`openai-realtime`](https://github.com/jambonz/node-sdk/tree/main/examples/openai-realtime) | WebSocket | OpenAI Realtime API speech-to-speech integration |
| [`deepgram-voice-agent`](https://github.com/jambonz/node-sdk/tree/main/examples/deepgram-voice-agent) | WebSocket | Deepgram `deepgram_s2s` end-to-end |
| [`elevenlabs-voice-agent`](https://github.com/jambonz/node-sdk/tree/main/examples/elevenlabs-voice-agent) | WebSocket | ElevenLabs `elevenlabs_s2s` (uses `agent_id` auth — model & prompt live in the ElevenLabs dashboard) |
| [`ultravox-voice-agent`](https://github.com/jambonz/node-sdk/tree/main/examples/ultravox-voice-agent) | WebSocket | Ultravox `ultravox_s2s` |
| [`retell-hosted`](https://github.com/jambonz/node-sdk/tree/main/examples/retell-hosted) | Webhook | Retell-hosted agent via jambonz |
| [`bedrock-agent`](https://github.com/jambonz/node-sdk/tree/main/examples/bedrock-agent.ts) | WebSocket | AWS Bedrock-backed voice agent |
| [`assemblyai-autogenerate-prompt`](https://github.com/jambonz/node-sdk/tree/main/examples/assemblyai-autogenerate-prompt) | WebSocket | AssemblyAI with dynamic prompt generation |
| [`llm-streaming`](https://github.com/jambonz/node-sdk/tree/main/examples/llm-streaming) | WebSocket | LLM token-streaming with `say` TTS |

## IVR, Routing, Call Control

| Starter | Transport | What it shows |
|---|---|---|
| [`hello-world`](https://github.com/jambonz/node-sdk/tree/main/examples/hello-world) | Webhook + WS | The simplest jambonz application: answer, greet, hang up. Ships both a webhook (`app.ts`) and WebSocket (`ws-app.ts`) version side-by-side. |
| [`ivr-menu`](https://github.com/jambonz/node-sdk/tree/main/examples/ivr-menu) | Webhook | A classic interactive voice response menu that routes callers based on DTMF or speech input |
| [`dial`](https://github.com/jambonz/node-sdk/tree/main/examples/dial) | Webhook | Bridge an incoming call to a phone number, SIP endpoint, or registered user |
| [`two-stage-dialer`](https://github.com/jambonz/node-sdk/tree/main/examples/two-stage-dialer) | Webhook | Outbound dial that confirms the called party before bridging |
| [`echo`](https://github.com/jambonz/node-sdk/tree/main/examples/echo) | Webhook | Echo test — answers and plays back what the caller says |

## Recording, Queueing, Streaming

| Starter | Transport | What it shows |
|---|---|---|
| [`call-recording`](https://github.com/jambonz/node-sdk/tree/main/examples/call-recording) | Webhook | Mid-call recording control: incoming call is bridged to an agent, recording can be started/stopped dynamically |
| [`listen-record`](https://github.com/jambonz/node-sdk/tree/main/examples/listen-record) | Webhook | Combine `listen` (audio streaming) with recording |
| [`queue-with-hold`](https://github.com/jambonz/node-sdk/tree/main/examples/queue-with-hold) | Webhook | Support queue: callers placed in queue with hold music and position announcements. Agents connect separately to dequeue. |
| [`realtime-translator`](https://github.com/jambonz/node-sdk/tree/main/examples/realtime-translator) | WebSocket | Real-time bidirectional speech translation between two parties |

## After Cloning

Every starter expects a few application environment variables (set them in the jambonz portal under your application's settings). The starter's `app.ts` declares the expected variables at the top via `envVars: { ... }` — read that block, then create matching values in the portal.

For configuration patterns, load `jambonz-recipes` and read [`references/env-vars.md`](../jambonz-recipes/references/env-vars.md).
