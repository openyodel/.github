# Open Yodel

**A universal semantics for human-machine communication.**

Talk or type to any AI agent, from any device, through any backend, over any medium. No vendor lock-in.

In the Alps, a yodeler stands on a peak and calls into the valley. He doesn't control who listens. He doesn't control how the sound travels — air, rock, water. He just yodels. Whoever hears it, knows what it means.

Open Yodel translates this principle into the digital world. It defines *what is called* — not how the signal carries. A universal layer of meaning for communication between humans and machines, lightweight enough to fit in 7 bytes for discovery, rich enough for full conversations when needed.

> **New here?** Start with the [Vision](https://github.com/openyodel/spec/blob/main/VISION.md) to understand where Open Yodel is heading.

---

## How it works today

Yodel v1 builds on the OpenAI-compatible `/v1/chat/completions` format and extends it with optional headers and metadata: TTS control, device context, session management. A backend that doesn't know Yodel still works — the extensions are fully optional.

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

## Where it's going

The v1 HTTP binding is the first transport. But Yodel is designed to ride on *any* medium — Bluetooth LE, LoRa, MQTT, ultrasound, even the power grid. A 7-byte presence beacon fits into any of them.

The [Yodel layer model](https://github.com/openyodel/spec/blob/main/VISION.md#part-ii--the-layer-model-how-yodelers-find-each-other) describes how nodes find each other with minimal energy:

- **Layer 0 — Presence:** Nodes pulse their existence. Seven bytes. Always on.
- **Layer 1 — Intent:** A node signals a need. Query, command, or notify.
- **Layer 2 — Capability:** Nodes match intent against their abilities.
- **Layer 3 — Handoff:** Direct connection established. Intermediary drops out.
- **Layer 4 — Exchange:** The actual work — conversations, commands, streaming.

Everything that exists today (v1 spec, SDK, sessions, streaming) is Layer 4. The layers below are next.

**Bridges** bring existing ecosystems in: Yodel-to-HomeAssistant, Yodel-to-MQTT, Yodel-to-Alexa. The devices keep speaking what they speak. The bridge translates. From Yodel's perspective, they're suddenly yodelers.

MCP connects agents to tools. A2A connects agents to each other. **Open Yodel connects humans to agents — over any transport.**

## What can you do with it?

Yodel carries your intent to an agent. What that agent does with it is entirely up to you.

**Home & automation**
> *"Mow a heart into the lawn"* → Your family agent triggers the robot mower via MCP. You get a confirmation — or a photo — when it's done.

**Smart home**
> *"It's getting cold upstairs"* → Your agent reads the thermostat, adjusts the temperature, dims the lights. One voice command, done.

**Infrastructure**
> *"Restart the VPS"* → Your agent SSHes in, reboots the server, and streams the confirmation back to you in real time.

**Media & surveillance**
> *"Make a timelapse from today's garden cam"* → Your agent pulls the footage, runs it through ffmpeg, and sends you the video.

**IoT without cloud**
> *"Lights on."* → An ESP32 on the lamp hears the Yodel over Bluetooth. No app, no cloud, no account. The light turns on.

**Cross-device**
> *"What was that recipe again?"* → You ask on your watch while cooking. The answer plays on the kitchen speaker — because that's the device that fits best right now.

**Anything with an API**
> If your agent can call it, Yodel can trigger it.

None of this lives inside the Yodel protocol — and that's intentional. Yodel carries your intent to the agent. The agent decides what to do. The protocol stays clean and backend-agnostic. The power is in what you build behind it.

```
You (voice / text / gesture)
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
| [**spec**](https://github.com/openyodel/spec) | Protocol specification + Vision | ✓ [v1.0-draft.1](https://github.com/openyodel/spec/releases/tag/v1.0-draft.1) |
| [**yodel-js**](https://github.com/openyodel/yodel-js) | TypeScript SDK (`@openyodel/sdk`) | ✓ [v0.1.0](https://www.npmjs.com/package/@openyodel/sdk) |
| [**yodel-agent-claude**](https://github.com/openyodel/yodel-agent-claude) | Claude agent server with native Yodel + OpenAI-compatible API | 🚧 In Progress |
| [**plugin-openyodel-hermes**](https://github.com/openyodel/plugin-openyodel-hermes) | AI agent platform adapter — connect any Yodel device to an AI agent | 🚧 In Progress |
| [**yodel-acoustic-pairing**](https://github.com/openyodel/yodel-acoustic-pairing) | Acoustic device discovery and pairing — research & spec | 🚧 In Progress |
| [**yodel-swift**](https://github.com/openyodel/yodel-swift) | iOS SDK (SwiftUI, Apple SpeechAnalyzer) | Planned |
| [**mcp**](https://github.com/openyodel/mcp) | MCP Server for Yodel discovery and connection | Planned |
| [**skill**](https://github.com/openyodel/skill) | Agent Skill for Yodel ([agentskills.io](https://agentskills.io) standard) | Planned |

## Roadmap

See [ROADMAP.md](https://github.com/openyodel/.github/blob/main/ROADMAP.md) for the full project roadmap.

| Milestone | What | Status |
|-----------|------|--------|
| **v1 Spec** | Protocol specification + OpenAPI 3.1 | ✓ Released |
| **TypeScript SDK** | `@openyodel/sdk` — client, session, discovery, types | ✓ v0.1.0 |
| **Service Discovery** | Well-known endpoint, mDNS, known hosts | ✓ In Spec + SDK |
| **iOS SDK** | SwiftUI client with Apple SpeechAnalyzer + WhisperKit | Planned |
| **v1 Demo** | End-to-end: speak into device → streamed response from LLM | Next |
| **Yodel Semantic Spec** | Layer 0–4, core fields, transport-agnostic foundation | Next |
| **First alternative binding** | Yodel-over-MQTT or Yodel-over-BLE as proof of concept | Next |
| **MCP Server** | Yodel discovery and connection as MCP tools | Planned |
| **Agent Skill** | Yodel connection as Agent Skill ([agentskills.io](https://agentskills.io)) | Planned |
| **Android SDK** | Kotlin/Compose client with Whisper.cpp | Planned |
| **Bridges** | Yodel-to-HomeAssistant, Yodel-to-MQTT | Planned |
| **v2 Spec** | WebSocket, bidirectional streaming, device mesh | Planned |

## Gateway

Open Yodel works directly with any OpenAI-compatible endpoint. A **Gateway** adds device management, agent configuration, and audio pairing on top — the control plane for which device talks to which agent, with which API key, through which backend.

**WIRE** will be the first proprietary gateway built on the Yodel protocol — a hosted platform with device pairing, agent management, and a web dashboard. But the protocol is open: anyone can build their own gateway, run it self-hosted, or integrate it into an existing platform.

## Contribute

Open Yodel grows when people build on it. Some starting points:

- **Build a bridge** — Yodel-to-HomeAssistant, Yodel-to-MQTT, Yodel-to-Alexa. Every bridge brings thousands of existing devices into the Yodel network.
- **Build an agent server** — A Yodel endpoint for Ollama, OpenAI, Mistral. The pattern is simple: Yodel in, provider API out.
- **Write an SDK** — Python, Go, Rust, C. Every SDK lowers the barrier for an entire developer community.
- **Specify a binding** — What does Yodel look like over BLE? Over LoRa? Over MQTT?
- **Build a beacon** — An ESP32 that yodels 7 bytes into the air. A Raspberry Pi that yodels over MQTT. Every beacon grows the presence network.
- **Contribute to the spec** — The [Vision](https://github.com/openyodel/spec/blob/main/VISION.md) has many open questions. Pick one.

## Design principles

- **Semantic, not structural** — Yodel defines meaning, not format. The same semantics encode as JSON over HTTP or as 7 bytes over Bluetooth.
- **OpenAI-compatible baseline** — v1 works with any `/v1/chat/completions` endpoint out of the box.
- **Extending, not replacing** — optional headers and metadata, never breaking compatibility.
- **Transport-agnostic** — HTTPS + SSE today. BLE, LoRa, MQTT, ultrasound — any medium that can carry 7 bytes.
- **Backend-agnostic** — your backend, your model, your rules.
- **Native clients, shared protocol** — no cross-platform frameworks. Each platform gets a native SDK.

## What Open Yodel is NOT

- Not a transport protocol — it's a message semantics that rides on any transport
- Not a replacement for MCP (tool integration) or A2A (agent-to-agent)
- Not a competitor to Pipecat or LiveKit — those are server-side frameworks
- Not tied to any specific AI provider
- Not a framework — it's a protocol and a set of SDKs

## License

MIT
