# Open Yodel — Project Roadmap

**Status:** Active development
**Last updated:** 2026-03-09

---

## Phase 1 — Foundation (current)

- ✅ Yodel Protocol Spec v1 (v1.0-draft.1 released)
- ✅ OpenAPI 3.1 Definition
- ✅ GitHub-Organisation + Repos aufgesetzt
- 🔲 Issue #4: Entscheidung über `yodel.context`-Feld
- 🔲 iOS SDK (openyodel/swift) — SwiftUI Client mit Apple SpeechAnalyzer + WhisperKit
- 🔲 PWA SDK (openyodel/js) — React/TypeScript Client mit Web Speech API
- 🔲 Service Discovery Spec (Well-Known Endpoint, Discovery-Hierarchie, Known Hosts)
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
