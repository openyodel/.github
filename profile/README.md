# Yodel

**The open voice protocol for AI agents.**

Yodel connects humans to AI agents — by voice, on any device, with any backend. No vendor lock-in.

Think of it as a walkie-talkie for your AI agents: you speak, your agent responds. Whether you're running Ollama locally, hitting the Claude API, or hosting your own vLLM server — Yodel doesn't care. It's the last mile between your device and your AI.

## How it works

Yodel builds on the OpenAI-compatible `/v1/chat/completions` format and extends it with optional voice-specific features: TTS control, device metadata, session management. A backend that doesn't know Yodel still works — the extensions are fully optional.

```
You (Voice) ──[ Yodel ]──> Any AI Backend
                              ├── Ollama
                              ├── LiteLLM / vLLM
                              ├── Claude API
                              ├── OpenAI API
                              └── Your own server
```

## Repos

| Repo | What | Status |
|---|---|---|
| [**spec**](https://github.com/openyodel/spec) | Protocol specification | In progress |
| [**swift**](https://github.com/openyodel/swift) | iOS SDK (SwiftUI, Apple SpeechAnalyzer) | In progress |
| [**js**](https://github.com/openyodel/js) | Web/PWA SDK (Web Speech API) | In progress |

## Design principles

- **OpenAI-compatible as baseline** — any `/v1/chat/completions` endpoint works out of the box
- **Extending, not replacing** — Yodel adds optional headers and metadata, never breaks compatibility
- **Transport-agnostic** — HTTPS + SSE today, WebSocket for bidirectional communication later
- **Backend-agnostic** — your backend, your model, your rules
- **Native clients, shared protocol** — no cross-platform frameworks, each platform gets a native SDK

## What Yodel is NOT

- Not a replacement for MCP (tool integration) or A2A (agent-to-agent) — Yodel is transport-layer only
- Not tied to any specific AI provider
- Not a framework — it's a protocol and a set of SDKs

## License

MIT
