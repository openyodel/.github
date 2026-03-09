# Open Yodel

**The open protocol for connecting humans to AI agents.**

Talk or type to any AI agent, from any device, through any backend. No vendor lock-in.

Think of it as a walkie-talkie for your AI agents: you speak, your agent responds. Whether you're running Ollama locally, calling the Claude API, orchestrating agents through n8n or OpenClaw, or hosting your own server — Open Yodel doesn't care. It's the last mile between you and your AI.

## How it works

The Yodel protocol builds on the OpenAI-compatible `/v1/chat/completions` format and extends it with optional features: TTS control, device metadata, session management. A backend that doesn't know the Yodel protocol still works — the extensions are fully optional.

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

Open Yodel v1 is the foundation — a clean, open protocol that works today with any OpenAI-compatible backend.

We're building this in phases — v1 ships with iOS + PWA first. But the roadmap goes further:

- **Device Mesh** — your phone, your car, your smart speaker, your laptop — all connected. Start a conversation in the car, finish it at home. Ask a question on your phone, hear the answer from the speaker in the room.
- **Cross-Device Routing** — audio goes where it makes sense. You whisper into your watch, the response plays on the nearest speaker.
- **Bidirectional Streaming** — agents don't just respond, they reach out. Proactive updates, real-time status, push notifications — over WebSocket.
- **IoT & Home Automation** — Open Yodel as a native integration for Home Assistant, Node-RED, MQTT. Voice-control your entire smart home through your own AI agent, not Alexa or Google.
- **Automotive** — talk to your own agent in the car via CarPlay and Android Auto, with your prompts, your backend, your data.
- **Dedicated Hardware** — a physical device. Microphone, speaker, WiFi, one button. The open-source walkie-talkie for AI agents.

MCP connects agents to tools. A2A connects agents to each other. **Open Yodel connects humans to agents.**

## What can you do with it?

Yodel carries your voice or text to an agent. What that agent does with it is entirely up to you.

**Home & automation**
> *"Mow a heart into the lawn"* → Your family agent triggers the robot mower via MCP. You get a confirmation — or a photo — when it's done.

**Infrastructure**
> *"Restart the VPS"* → Your agent SSHes in, reboots the server, and streams the confirmation back to you in real time.

**Smart home**
> *"It's getting cold upstairs"* → Your agent reads the thermostat, adjusts the temperature, dims the lights. One voice command, done.

**Media & surveillance**
> *"Make a timelapse from today's garden cam"* → Your agent pulls the footage, runs it through ffmpeg, and sends you the video.

**Anything with an API**
> If your agent can call it, Yodel can trigger it.

None of this lives inside the Yodel protocol itself — and that's intentional. Yodel carries your intent to the agent. The agent decides what to do: using MCP to call tools, A2A to delegate to specialized sub-agents, or any backend logic you write. The protocol stays clean and backend-agnostic. The power is in what you build behind it.

```
You (voice / text)
        │
        │  Yodel Protocol
        ▼
  Your Agent
        ├── MCP  → tools, APIs, smart home, databases
        ├── A2A  → specialized sub-agents
        └── any backend logic you write
```

## Repos

| Repo | What | Status |
|---|---|---|
| [**spec**](https://github.com/openyodel/spec) | Protocol specification | ✓ [v1.0-draft.1](https://github.com/openyodel/spec/releases/tag/v1.0-draft.1) |
| [**swift**](https://github.com/openyodel/swift) | iOS SDK (SwiftUI, Apple SpeechAnalyzer) | Planned |
| [**js**](https://github.com/openyodel/js) | Web/PWA SDK (Web Speech API) | Planned |
| [**mcp**](https://github.com/openyodel/mcp) | MCP Server for Yodel discovery and connection | Planned |
| [**skill**](https://github.com/openyodel/skill) | Agent Skill for Yodel ([agentskills.io](https://agentskills.io) standard) | Planned |

## Roadmap

See [ROADMAP.md](https://github.com/openyodel/.github/blob/main/ROADMAP.md) for the full project roadmap.

| Milestone | What | Status |
|-----------|------|--------|
| **v1 Spec** | Protocol specification + OpenAPI 3.1 | ✓ Released ([v1.0-draft.1](https://github.com/openyodel/spec/releases/tag/v1.0-draft.1)) |
| **iOS SDK** | SwiftUI client with Apple SpeechAnalyzer + WhisperKit | Planned |
| **PWA SDK** | React/TypeScript client with Web Speech API | Planned |
| **Service Discovery** | Well-known endpoint, mDNS, known hosts | Planned |
| **v1 Demo** | End-to-end: speak into device → streamed response from LLM | Next |
| **MCP Server** | Yodel discovery and connection as MCP tools | Planned |
| **Agent Skill** | Yodel connection as Agent Skill ([agentskills.io](https://agentskills.io)) | Planned |
| **Android SDK** | Kotlin/Compose client with Whisper.cpp | Planned |
| **CarPlay / Android Auto** | Talk to your agent in the car | Planned |
| **Home Assistant** | Voice-control your smart home through your own agent | Planned |
| **v2 Spec** | WebSocket, bidirectional streaming, device mesh | Planned |

## Gateway

Open Yodel works directly with any OpenAI-compatible endpoint. A **Gateway** adds device management, agent configuration, and audio pairing on top — the control plane for which device talks to which agent, with which API key, through which backend.

**WIRE (Project Title)** will be the first propetary gateway built on the Yodel protocol — a hosted platform with device pairing, agent management, and a web dashboard. But the protocol is open: anyone can build their own gateway, run it self-hosted, or integrate the Yodel protocol into an existing platform.

## Design principles

- **OpenAI-compatible as baseline** — any `/v1/chat/completions` endpoint works out of the box
- **Extending, not replacing** — the Yodel protocol adds optional headers and metadata, never breaks compatibility
- **Transport-agnostic** — HTTPS + SSE today, WebSocket for bidirectional communication later
- **Backend-agnostic** — your backend, your model, your rules
- **Native clients, shared protocol** — no cross-platform frameworks, each platform gets a native SDK

## What Open Yodel is NOT

- Not a replacement for MCP (tool integration) or A2A (agent-to-agent) — the Yodel protocol is transport-layer only
- Not a competitor to Pipecat or LiveKit — those are server-side frameworks, Yodel is a client-to-backend protocol
- Not tied to any specific AI provider
- Not a framework — it's a protocol and a set of SDKs

## License

MIT
