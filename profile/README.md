# Yodel

**The open protocol for connecting humans to AI agents.**

Talk or type to any AI agent, from any device, through any backend. No vendor lock-in.

Think of it as a walkie-talkie for your AI agents: you speak, your agent responds. Whether you're running Ollama locally, calling the Claude API, orchestrating agents through n8n or OpenClaw, or hosting your own server — Yodel doesn't care. It's the last mile between you and your AI.

## How it works

Yodel builds on the OpenAI-compatible `/v1/chat/completions` format and extends it with optional features: TTS control, device metadata, session management. A backend that doesn't know Yodel still works — the extensions are fully optional.

```
  Devices                                          Agents & Backends

  iPhone              ┌─────────────┐              Local Models
  Android             │             │              ├── Ollama
  Browser / PWA       │             │              ├── LiteLLM / vLLM
  CarPlay        ────>│    Yodel    │────>         ├── llama.cpp
  Android Auto        │  Protocol   │
  Smart Speaker       │             │              AI APIs
  Smart Watch         │             │              ├── Claude API
  Custom Hardware     └─────────────┘              ├── OpenAI API
                                                   ├── Google Gemini
                       Voice | Text
                                                   Agent Platforms
                                                   ├── n8n
                                                   ├── OpenClaw
                                                   └── Any OpenAI-compatible endpoint
```

## Vision

Yodel v1 is the foundation — a clean, open protocol that works today with any OpenAI-compatible backend.

We're building this in phases — v1 ships with iOS + PWA first. But the roadmap goes further:

- **Device Mesh** — your phone, your car, your smart speaker, your laptop — all connected. Start a conversation in the car, finish it at home. Ask a question on your phone, hear the answer from the speaker in the room.
- **Cross-Device Routing** — audio goes where it makes sense. You whisper into your watch, the response plays on the nearest speaker.
- **Bidirectional Streaming** — agents don't just respond, they reach out. Proactive updates, real-time status, push notifications — over WebSocket.
- **IoT & Home Automation** — Yodel as a native integration for Home Assistant, Node-RED, MQTT. Voice-control your entire smart home through your own AI agent, not Alexa or Google.
- **Automotive** — talk to your own agent in the car via CarPlay and Android Auto, with your prompts, your backend, your data.
- **Dedicated Hardware** — a physical device. Microphone, speaker, WiFi, one button. The open-source walkie-talkie for AI agents.

MCP connects agents to tools. A2A connects agents to each other. **Yodel connects humans to agents.**

## Repos

| Repo | What | Status |
|---|---|---|
| [**spec**](https://github.com/openyodel/spec) | Protocol specification | In progress |
| [**swift**](https://github.com/openyodel/swift) | iOS SDK (SwiftUI, Apple SpeechAnalyzer) | In progress |
| [**js**](https://github.com/openyodel/js) | Web/PWA SDK (Web Speech API) | In progress |

## Gateway

Yodel works directly with any OpenAI-compatible endpoint. A **Gateway** adds device management, agent configuration, and audio pairing on top — the control plane for which device talks to which agent, with which API key, through which backend.

**WIRE** is the first gateway built on Yodel — a hosted platform with device pairing, agent management, and a web dashboard. But the protocol is open: anyone can build their own gateway, run it self-hosted, or integrate Yodel into an existing platform.

## Design principles

- **OpenAI-compatible as baseline** — any `/v1/chat/completions` endpoint works out of the box
- **Extending, not replacing** — Yodel adds optional headers and metadata, never breaks compatibility
- **Transport-agnostic** — HTTPS + SSE today, WebSocket for bidirectional communication later
- **Backend-agnostic** — your backend, your model, your rules
- **Native clients, shared protocol** — no cross-platform frameworks, each platform gets a native SDK

## What Yodel is NOT

- Not a replacement for MCP (tool integration) or A2A (agent-to-agent) — Yodel is transport-layer only
- Not a competitor to Pipecat or LiveKit — those are server-side frameworks, Yodel is a client-to-backend protocol
- Not tied to any specific AI provider
- Not a framework — it's a protocol and a set of SDKs

## License

MIT
