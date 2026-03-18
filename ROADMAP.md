# Open Yodel — Project Roadmap

**Status:** Active development
**Last updated:** 2026-03-14

---

## Phase 1 — Foundation (current)

- ✅ Yodel Protocol Spec v1 (v1.0-draft.1 released)
- ✅ OpenAPI 3.1 Definition
- ✅ GitHub-Organisation + Repos aufgesetzt
- ✅ `yodel.context`-Feld entfernt — §12.3 Forward Compatibility reicht ([#4](https://github.com/openyodel/spec/issues/4))
- ✅ Service Discovery Spec (Well-Known Endpoint, Discovery-Hierarchie, Known Hosts)
- ✅ Erster Live-Test: curl → OpenCode → Claude (Yodel-Headers, Session-ID, Proof of Concept)
- ✅ Spec: Yodel-Headers sind transport-agnostic, Body ist backend-spezifisch — §6.1 ([#5](https://github.com/openyodel/spec/issues/5))
- ✅ Spec: Session-ID-Mapping Backend → X-Yodel-Session — §8 + OpenAPI ([#6](https://github.com/openyodel/spec/issues/6))
- ✅ TypeScript SDK (openyodel/yodel-js) — YodelClient, YodelSession, DiscoveryClient, STT/TTS-Interfaces, Types, Tests
- ✅ Spec: Non-streaming Backends — bewusst out of scope in §5.2 ([#7](https://github.com/openyodel/spec/issues/7))
- ✅ NPM: `@openyodel/sdk` v0.1.0 publiziert
- 🔲 iOS SDK (openyodel/yodel-swift) — SwiftUI Client mit Apple SpeechAnalyzer + WhisperKit
- 🔲 Erster End-to-End-Demo: Mikrofon → LLM → gestreamte Antwort (30-Sekunden-Video)

---

## Phase 2 — Ecosystem

- 🔲 Netzwerk-Discovery (mDNS/DNS-SD `_yodel._tcp`) — Zero-Config in vertrauenswürdigen Netzwerken
- 🔲 MCP Server (openyodel/mcp) — Yodel-Discovery und -Verbindung als MCP-Tools
- 🔲 Agent Skill (openyodel/skill) — Yodel-Verbindung als Agent Skill (SKILL.md, [agentskills.io](https://agentskills.io)-Standard)
- 🔲 Android SDK (Kotlin/Compose mit Whisper.cpp)
- 🔲 Community Feedback einarbeiten, Spec stabilisieren
- 🔲 v1.0 Final Release

---

## Phase 3 — Next Generation

- 🔲 v2 Spec: WebSocket, bidirektionales Streaming, Device Mesh
- 🔲 CarPlay / Android Auto Integration
- 🔲 Home Assistant / Node-RED / MQTT Integration
- 🔲 Dedicated Hardware Referenz-Design

---

Open Yodel ist ein unabhängiges Open-Source-Protokoll. Gateways wie [WIRE](https://github.com/moongrabber) bauen auf dem Protokoll auf, treiben es aber nicht.
