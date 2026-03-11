# Open Yodel — SDK Design Guide

**Version:** 0.2-draft
**Date:** 2026-03-11
**Status:** Draft
**Location:** This document lives in `openyodel/.github` or `openyodel/spec` — it is the reference for all SDK implementations.

---

## 1. Purpose

This document defines the shared architecture, terminology, types, and conventions for all Open Yodel SDKs. Each platform-specific SDK (JS, Swift, Kotlin, Embedded) implements this design in the idioms of its respective language.

**Goal:** A developer who knows one SDK can immediately navigate any other. Not identical code, but identical concepts.

---

## 2. Design Principles

### 2.1 Cross-Platform

1. **The protocol is the truth.** Everything derives from the [Yodel Protocol Specification](https://github.com/openyodel/spec). SDKs don't interpret the spec — they implement it.
2. **Same concepts, native idioms.** The architecture (Client, Session, Provider) is the same everywhere. The implementation leverages each platform's strengths (Swift Concurrency, Kotlin Coroutines, JS async/await).
3. **Progressive disclosure.** The simplest use case (send text, stream response) requires minimal code. Advanced features (STT, TTS, Discovery, Gateway) are opt-in.
4. **No vendor lock-in.** Every SDK works out-of-the-box with any OpenAI-compatible endpoint and can communicate with any Yodel-compatible endpoint — direct or through a gateway — via configuration.
5. **Modular.** STT and TTS are swappable providers, not baked into the core. The core is pure HTTP + SSE.
6. **Provider pattern.** STT and TTS engines are abstracted behind a shared interface. Platforms supply their own implementations.

### 2.2 Per SDK

7. **Platform-native.** No cross-platform frameworks. Swift for Apple, Kotlin for Android, TypeScript for Web. Each SDK uses native APIs and toolchains.
8. **Platform package manager.** npm/pnpm for JS, Swift Package Manager for Swift, Gradle/Maven for Kotlin.
9. **Platform conventions.** Naming follows the conventions of the target language. Yodel terminology stays consistent across all SDKs (see section 5).

---

## 3. Architecture

Every SDK consists of the same logical layers. Not every layer needs to be a separate package/module — that's platform-specific. But the layers exist conceptually in every SDK.

```
┌─────────────────────────────────────────────────────────────┐
│                    Application (App / PWA / CLI)             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│   │ STT Provider │  │ Yodel Client │  │  TTS Player  │     │
│   │  (optional)  │  │    (core)    │  │  (optional)  │     │
│   └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│          │                 │                  │              │
│          │  provides text  │  sends/receives  │  plays audio │
│          └────────────────►│◄─────────────────┘              │
│                            │                                │
│                    ┌───────┴───────┐                        │
│                    │    Session    │                        │
│                    │  (optional)   │                        │
│                    └───────┬───────┘                        │
│                            │                                │
│                    ┌───────┴───────┐                        │
│                    │   Discovery   │                        │
│                    │  (optional)   │                        │
│                    └───────┬───────┘                        │
│                            │                                │
├────────────────────────────┼────────────────────────────────┤
│                     Yodel Protocol                          │
│              HTTPS + SSE │ X-Yodel-* Headers                │
├────────────────────────────┼────────────────────────────────┤
│                     Network / Transport                     │
└─────────────────────────────────────────────────────────────┘
```

### 3.1 Yodel Client (Core)

The core of every SDK. Accepts text, builds a Yodel request, sends it to the endpoint, parses the SSE stream, and returns the response.

**Responsibilities:**
- Build Yodel request (headers + body + optional `yodel` block)
- HTTP POST to the configured endpoint (default: `/v1/chat/completions`, or a gateway proxy endpoint like `/v1/yodel/{agent_slug}`, or any custom endpoint)
- Parse SSE stream (content chunks, yodel event, `[DONE]`)
- Error handling (HTTP errors + stream errors)
- Hold configuration (endpoint, model, API key, agent, device)

**Not responsible for:**
- Speech recognition (STT) — that's a provider
- Speech output (TTS) — that's a player
- Session history — that's the session layer

### 3.2 Session

Manages conversation history for persistent mode.

**Responsibilities:**
- Build and maintain the message array
- Insert system prompt as the first message, **if configured** (many agents have their own system prompt on the backend — the SDK does not require one)
- Append assistant responses to the history after receiving them
- Capture session ID from the yodel response event and include it in subsequent requests
- Export / import history (for client-side persistence)

**Rules:**
- In ephemeral mode, no history is retained — each request contains only the current user message (and the system prompt, if configured).
- In persistent mode, the session sends the full history with every request.
- The session is an optional wrapper around the client, not a mandatory component.

### 3.3 STT Provider

Abstracts platform-specific speech recognition.

**Responsibilities:**
- Microphone access and audio capture
- Local transcription (on-device)
- Report interim results (live while the user speaks) and final results
- Availability check (microphone permission, engine available?)
- Resource cleanup

**Providers per platform:**

| Platform | Primary | Fallback |
|----------|---------|----------|
| Web / JS | Web Speech API | Whisper WASM |
| iOS | Apple SpeechAnalyzer (iOS 26+) | WhisperKit |
| Android | — (Phase 2) | Whisper.cpp |
| Embedded | — (Phase 3) | Whisper.cpp |

### 3.4 TTS Player

Plays audio URLs delivered by the backend in the yodel response event.

**Responsibilities:**
- Fetch and play audio from a URL
- Playback control (play, pause, resume, stop)
- Volume control
- Format support (opus, mp3, wav, aac)

**Note:** TTS generation happens on the backend. The SDK only plays the finished audio.

### 3.5 Discovery

Automatically finds Yodel endpoints.

**Responsibilities:**
- Query the well-known endpoint (`/.well-known/yodel.json`)
- Parse and return the response as a typed object
- Read known hosts file (platform-specific format and storage location)
- mDNS/DNS-SD discovery (optional, Phase 2)

---

## 4. Component Overview per Platform

Each platform has its own package structure. The logical components are the same; the split into packages/modules is platform-specific.

### 4.1 JS / TypeScript (Web, PWA, Node, Electron)

```
@openyodel/sdk             Core: YodelClient, Session, Discovery, Types
@openyodel/web-speech      STT Provider: Web Speech API
@openyodel/whisper-wasm    STT Provider: Whisper WASM
@openyodel/tts             TTS Player: Web Audio API
@openyodel/gateway         Gateway integration (generic, works with any Yodel gateway)
```

Package manager: npm (distribution) / pnpm (development)
Repo: `openyodel/js`

### 4.2 Swift (iOS, macOS, watchOS, CarPlay)

```
OpenYodel                  Swift Package with targets:
  OpenYodelCore              Core: YodelClient, Session, Discovery, Types
  OpenYodelSpeech            STT Provider: Apple SpeechAnalyzer
  OpenYodelWhisper           STT Provider: WhisperKit (fallback)
  OpenYodelTTS               TTS Player: AVFoundation
  OpenYodelGateway           Gateway integration (generic)
```

Package manager: Swift Package Manager
Repo: `openyodel/swift`

### 4.3 Kotlin (Android, Android Auto)

```
io.openyodel.sdk           Core: YodelClient, Session, Discovery, Types
io.openyodel.stt.whisper   STT Provider: Whisper.cpp
io.openyodel.tts           TTS Player: Android MediaPlayer
io.openyodel.gateway       Gateway integration (generic)
```

Package manager: Gradle (Maven Central)
Repo: `openyodel/kotlin`

### 4.4 Embedded / C (Hardware, IoT) — Phase 3

```
openyodel-core             Core: Yodel Client (C, minimal)
openyodel-stt              STT: Whisper.cpp
```

Repo: `openyodel/embedded`

**Note:** The decision between C99, C++, or a C core with C++ wrapper is deferred to Phase 3. For now, all shared types (section 6) MUST be representable in C99. This prevents defining types that cannot be mapped to the embedded context later (e.g., no nested optionals, no generic containers).

---

## 5. Shared Terminology (Naming Convention)

These terms are **identical across all SDKs**. They derive directly from the Yodel Spec and MUST NOT be renamed, abbreviated, or "improved."

### 5.1 Classes / Types

| Concept | Name in all SDKs | Origin |
|---------|-------------------|--------|
| The main client | `YodelClient` | — |
| Conversation history | `YodelSession` | Spec §8 |
| SSE stream object | `YodelStream` | — |
| A chunk in the stream | `YodelStreamChunk` | Spec §7.1.1 |
| Yodel event in stream | `YodelResponseEvent` | Spec §7.1.2 |
| Error | `YodelError` | Spec §9 |
| Discovery response | `YodelDiscovery` | Spec §13.2 |
| STT abstraction | `STTProvider` (interface) | — |
| TTS playback | `TTSPlayer` | — |
| TTS configuration | `TTSConfig` | Spec §6.4.2 |
| Device metadata | `DeviceConfig` | Spec §6.4.3 |
| Agent configuration | `AgentConfig` | Spec §6.2 |
| Gateway client | `GatewayClient` | — |
| Gateway configuration | `GatewayConfig` | — |

### 5.2 Methods

| Action | Method name | Description |
|--------|-------------|-------------|
| Send message + receive stream | `chat()` | Core method. Accepts text, returns stream. |
| Create a session | `createSession()` | Creates a new `YodelSession`. |
| Perform discovery | `discover()` | Queries `/.well-known/yodel.json`. |
| Pull config (gateway) | `pullConfig()` | Fetches device configuration from a gateway. |
| Start STT | `start()` | On the STT provider. |
| Stop STT | `stop()` | On the STT provider. |
| Play audio | `play()` | On the TTS player. |

### 5.3 Events / Callbacks

| Event | Name | When |
|-------|------|------|
| Text delta received | `content` | On every content chunk in the stream |
| Yodel metadata received | `yodel` | When the yodel event arrives in the stream |
| Stream ended | `done` | After `[DONE]` |
| Error | `error` | On HTTP or stream error |
| STT interim result | `interim` | While the user is speaking |
| STT final result | `final` | When transcription is complete |

### 5.4 Stream Consumption

Every SDK MUST support two consumption patterns for `YodelStream`:

1. **Pull-based (iterator).** The consumer pulls chunks one at a time in a loop. This is the simple, linear pattern.
2. **Push-based (events/callbacks).** The consumer registers handlers for specific events. This gives more control and allows handling content, yodel events, and errors separately.

Both patterns MUST be available in every SDK. The implementation uses platform-native mechanisms:

| Pattern | TypeScript | Swift | Kotlin |
|---------|-----------|-------|--------|
| Pull (iterator) | `for await (const chunk of stream)` | `for try await chunk in stream` | `stream.collect { chunk -> }` |
| Push (events) | `stream.on("content", ...)` | `stream.onContent { ... }` | `stream.onContent { ... }` |

### 5.5 Platform-Specific Adaptations

The terms above are binding. The **casing** adapts to the platform:

| Concept | TypeScript | Swift | Kotlin |
|---------|-----------|-------|--------|
| Class | `YodelClient` | `YodelClient` | `YodelClient` |
| Method | `client.chat()` | `client.chat()` | `client.chat()` |
| Config field | `apiKey` | `apiKey` | `apiKey` |
| Enum value | `"ephemeral"` | `.ephemeral` | `SessionMode.EPHEMERAL` |
| Async | `await client.chat()` | `try await client.chat()` | `client.chat()` (suspend) |

---

## 6. Shared Types

These types are derived from the Yodel Spec and exist in every SDK — in the syntax of the respective language, but with identical structure and identical field names.

**Constraint:** All types MUST be representable in C99. This ensures future compatibility with the Embedded SDK (Phase 3). Concretely: no nested optionals, no generic containers, no inheritance hierarchies.

### 6.1 YodelClientConfig

Configuration for creating a `YodelClient`.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `endpoint` | String (URL) | Yes | Backend or gateway URL |
| `model` | String | Yes | Model identifier |
| `apiKey` | String | No | Backend API key |
| `agent` | AgentConfig | No | Agent configuration |
| `deviceId` | String (UUID) | No | Device identity for `X-Yodel-Device` |
| `tts` | TTSConfig | No | Default TTS configuration |
| `device` | DeviceConfig | No | Default device metadata |
| `timeout` | Integer (ms) | No | Request timeout. Default: 30000 |

### 6.2 AgentConfig

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `slug` | String | Yes | Agent slug for `X-Yodel-Agent` |
| `systemPrompt` | String | No | System prompt. If omitted, no system message is sent — the backend's own prompt (if any) takes effect. |
| `mode` | `"ephemeral"` \| `"persistent"` | No | Session mode. Default: `"ephemeral"` |
| `language` | String (BCP 47) | No | Language |

### 6.3 TTSConfig

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `requested` | Boolean | No | TTS requested? Default: `false` |
| `voice` | String | No | Voice (e.g., `"alloy"`) |
| `provider` | String | No | TTS provider (e.g., `"elevenlabs"`) |
| `format` | `"opus"` \| `"mp3"` \| `"wav"` \| `"aac"` | No | Audio format. Default: `"opus"` |

### 6.4 DeviceConfig

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | DeviceType | No | Device type |
| `capabilities` | DeviceCapability[] | No | Device capabilities |

**DeviceType:** `"ios"` | `"android"` | `"web"` | `"car"` | `"speaker"` | `"terminal"` | `"embedded"`

**DeviceCapability:** `"audio_out"` | `"audio_in"` | `"display"` | `"haptic"` | `"camera"`

### 6.5 ChatOptions

Optional parameters per `chat()` call that override client defaults.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `input` | `"voice"` \| `"text"` | No | Input source. Default: `"text"` |
| `tts` | TTSConfig | No | TTS for this request |
| `device` | DeviceConfig | No | Device metadata for this request |
| `inputLang` | String (BCP 47) | No | Input language |
| `temperature` | Number | No | Sampling temperature (OpenAI-compatible) |
| `maxTokens` | Integer | No | Max tokens (OpenAI-compatible) |

### 6.6 YodelStreamChunk

| Field | Type | Description |
|-------|------|-------------|
| `content` | String | Text delta |
| `role` | String? | `"assistant"` (first chunk only) |
| `finishReason` | String? | `"stop"` \| `"length"` \| `null` |

### 6.7 YodelResponseEvent

| Field | Type | Description |
|-------|------|-------------|
| `ttsUrl` | String? | URL to TTS audio |
| `sessionId` | String? | Server-assigned session ID |

### 6.8 YodelError

| Field | Type | Description |
|-------|------|-------------|
| `message` | String | Human-readable error text |
| `type` | String | Error type (from Spec §9) |
| `code` | String? | Machine-readable error code |
| `status` | Integer | HTTP status code |

### 6.9 YodelDiscovery

| Field | Type | Description |
|-------|------|-------------|
| `yodelVersion` | Integer | Highest supported protocol version |
| `endpoints` | Map<String, String> | Available endpoints |
| `capabilities` | String[] | Backend capabilities |
| `gateway` | String? | Gateway name (if present) |
| `agents` | DiscoveryAgent[] | Published agents |

---

## 7. STT Provider Interface

Every SDK defines an STT provider interface. Methods and events are the same everywhere; the implementation is platform-specific.

### 7.1 Interface Definition (language-agnostic)

```
interface STTProvider {
    /** Starts microphone capture and transcription */
    start() → async/throws

    /** Stops capture and transcription */
    stop() → async/throws

    /** Checks if the provider is available (permission, engine) */
    isAvailable() → async → Boolean

    /** Releases resources */
    dispose()

    // Events:
    on "interim" → (text: String)     // Interim result
    on "final"   → (text: String)     // Final result
    on "error"   → (error: Error)     // Error
}
```

### 7.2 Rules for Provider Implementations

1. **No provider is required.** An SDK works without any STT provider — it's then a text-only client.
2. **Providers are swappable.** The app chooses the provider, not the SDK.
3. **Providers are separate packages/modules.** They are only loaded when imported.
4. **`isAvailable()` checks everything.** Microphone permission, engine availability, platform support. Returns `false` if anything is missing.
5. **`start()` is idempotent.** Calling it twice does not start twice.
6. **`dispose()` is final.** After `dispose()`, the provider is no longer usable.

---

## 8. TTS Player Interface

### 8.1 Interface Definition (language-agnostic)

```
interface TTSPlayer {
    /** Fetches and plays audio from a URL */
    play(url: String) → async/throws

    /** Pauses playback */
    pause()

    /** Resumes playback */
    resume()

    /** Stops playback */
    stop()

    /** Volume (0.0 – 1.0) */
    volume: Number (read/write)

    /** Releases resources */
    dispose()

    // Events:
    on "started"  → ()                // Playback started
    on "finished" → ()                // Playback finished
    on "error"    → (error: Error)    // Error
}
```

---

## 9. Gateway Integration

Gateway integration is **optional** and exists as a **separate package/module** per platform. The SDK does not depend on any specific gateway implementation — it defines a generic interface that works with any Yodel-compatible gateway.

### 9.1 Gateway Client

The `GatewayClient` wraps a `YodelClient` and adds gateway-specific functionality: config pull, proxy routing, and secret rotation.

**Responsibilities:**
- Authenticate with the gateway using a device secret
- Pull device configuration (available agents, endpoints, models)
- Route `chat()` calls through the gateway proxy endpoint (`/v1/yodel/{agent_slug}`)
- Handle secret rotation

**Not responsible for:**
- Gateway-specific features (billing, account management, dashboards) — those are gateway UI concerns, not SDK concerns

### 9.2 GatewayConfig

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `endpoint` | String (URL) | Yes | Gateway URL |
| `deviceId` | String (UUID) | Yes | Device ID |
| `deviceSecret` | String | Yes | Device secret for authentication |

### 9.3 Gateway Methods

| Method | Description |
|--------|-------------|
| `GatewayClient(config)` | Create a gateway-connected client |
| `pullConfig()` | Fetch device configuration (available agents) |
| `chat(text, agentSlug, options?)` | Send message through gateway proxy |
| `rotateSecret()` | Rotate the device secret |

### 9.4 Proxy Routing

When connected through a gateway, `chat()` sends requests to `/v1/yodel/{agent_slug}` instead of `/v1/chat/completions`. The gateway injects API keys server-side — the device never sees backend credentials. This is handled transparently by the `GatewayClient`.

### 9.5 Gateway Packages per Platform

| Platform | Package |
|----------|---------|
| JS / TypeScript | `@openyodel/gateway` |
| Swift | `OpenYodelGateway` (target in `OpenYodel` package) |
| Kotlin | `io.openyodel.gateway` |
| Embedded | Not applicable (Phase 3+) |

---

## 10. Error Handling

### 10.1 Error Classification

All SDKs use the same error types — derived from the Yodel Spec §9.

| Error type | Description |
|------------|-------------|
| `authentication_error` | Invalid or missing API key / device secret |
| `authorization_error` | Valid key but insufficient permissions |
| `validation_error` | Invalid request body |
| `not_found_error` | Endpoint, model, or resource not found |
| `conflict_error` | Resource conflict (e.g., duplicate slug) |
| `backend_error` | Backend unreachable or returned an error |
| `rate_limit_error` | Too many requests |
| `stream_error` | Error during SSE stream (after HTTP 200) |
| `timeout_error` | Request timeout |
| `stt_error` | Speech recognition error |
| `tts_error` | Audio playback error |
| `network_error` | No network connection |

### 10.2 Rules

1. **Every error is a `YodelError`** (or a platform-specific subtype).
2. **HTTP errors before the stream** are thrown as exceptions/errors.
3. **Errors during the stream** (after HTTP 200) are emitted as `error` events.
4. **Error type and code** come from the backend response when available. Otherwise, the SDK sets sensible defaults.
5. **Never silent failures.** Every error is reported — either as an exception or as an event.

---

## 11. Yodel Protocol Compliance Levels

SDKs (including third-party implementations) declare which compliance level they implement. This helps developers choose an SDK and helps the community verify implementations.

### 11.1 Levels

| Level | Name | Requirements |
|-------|------|--------------|
| **1** | **Core** | `YodelClient` + `chat()` with SSE streaming. Text in, streamed text out. Yodel headers and `yodel` request block supported. Error handling per Spec §9. |
| **2** | **Session** | Level 1 + `YodelSession` with persistent mode support. Session ID handling from yodel response events. History management. |
| **3** | **Discovery** | Level 2 + `discover()` with well-known endpoint support. Known hosts file support. |
| **4** | **Voice** | Level 3 + at least one `STTProvider` implementation + `TTSPlayer` implementation. Full voice flow: microphone → transcription → Yodel request → streamed response → audio playback. |
| **5** | **Gateway** | Level 4 + `GatewayClient` with config pull, proxy routing, and secret rotation. |

### 11.2 Rules

- Levels are cumulative. Level 3 implies Level 2 implies Level 1.
- A SDK MUST declare its compliance level in its README.
- A SDK MAY skip levels (e.g., implement Level 1 + Level 5 without Level 4), but MUST document this explicitly.
- Third-party SDKs are encouraged to use the same compliance levels.

### 11.3 Compliance Badge

SDKs SHOULD display a compliance badge in their README:

```
Yodel Protocol v1 — Compliance Level 4 (Voice)
```

---

## 12. Versioning

### 12.1 SDK Versioning

All SDKs use [Semantic Versioning](https://semver.org/):

- **Major:** Breaking API changes
- **Minor:** New features, backward-compatible
- **Patch:** Bugfixes

### 12.2 Protocol Versioning

SDK version and Yodel protocol version are **independent**. An SDK at version 2.3.1 may implement Yodel Protocol v1.

Every SDK MUST document which Yodel protocol version it implements.

### 12.3 Compatibility Matrix

Every SDK maintains a compatibility matrix in its README:

```
| SDK Version | Yodel Protocol | Compliance Level |
|-------------|---------------|------------------|
| 0.1.x       | v1            | Level 1 (Core)   |
| 0.2.x       | v1            | Level 2 (Session) |
```

---

## 13. Documentation

### 13.1 Every SDK MUST Include

| Document | Description |
|----------|-------------|
| `README.md` | Overview, installation, quick start, compatibility matrix, compliance level |
| `CHANGELOG.md` | All changes per version (generated via Changesets or equivalent) |
| `CONTRIBUTING.md` | How to contribute |
| `LICENSE` | MIT |
| API documentation | Generated from code (TypeDoc, DocC, Dokka) |

### 13.2 Quick Start Format

Every SDK README begins with the same narrative arc:

1. Installation (1 command)
2. Simplest use case: text → stream (5 lines of code)
3. Voice use case: microphone → STT → stream → TTS (10–15 lines of code)
4. Further links (API docs, Design Guide, Spec)

---

## 14. Testing

### 14.1 Every SDK MUST Test

| Area | What |
|------|------|
| Yodel Client | Request construction (headers, body, `yodel` block) |
| Yodel Client | SSE parsing (content chunks, yodel event, `[DONE]`) |
| Yodel Client | Error handling (HTTP errors, stream errors) |
| Session | History management (ephemeral vs. persistent) |
| Session | Session ID capture from yodel event |
| Discovery | Well-known endpoint parsing |
| STT Provider | Mocking (no real microphone access in tests) |
| TTS Player | Mocking (no real audio playback in tests) |
| Gateway | Config pull, proxy routing (against mock server) |

### 14.2 Shared Test Fixtures

A shared set of test fixtures lives in `openyodel/spec/test-fixtures/`. These are JSON files that every SDK runs against:

- Correct header generation for various configurations
- Correct body construction with and without `yodel` block
- SSE parsing for various stream scenarios (normal, error mid-stream, missing yodel event, empty content)
- Error response parsing for all error types
- Discovery response parsing (direct backend, gateway, empty agents)

Each SDK includes the `spec` repo as a submodule or fetches the fixtures in CI.

---

## 15. CI/CD

### 15.1 Approach

Each SDK maintains its own CI/CD workflows. The build toolchains are too different across platforms (pnpm vs. Xcode vs. Gradle) for shared templates to add value.

### 15.2 Required CI Stages

Every SDK MUST implement these CI stages, regardless of the specific toolchain:

| Stage | Description |
|-------|-------------|
| **Lint** | Code style and static analysis |
| **Test** | Unit tests + fixture tests |
| **Build** | Compile and bundle |
| **Publish** | Release to package registry (npm, SPM, Maven Central) |

### 15.3 Conventions

- All SDKs use **Conventional Commits** for commit messages.
- All SDKs use a controlled release process (e.g., Changesets for JS, manual tagging for Swift/Kotlin).
- All SDKs run CI on every pull request and on every push to `main`.

---

## 16. Repository Structure

### 16.1 Per SDK

Each SDK lives in its own repo under `openyodel/`:

| Repo | SDK | Language |
|------|-----|----------|
| `openyodel/js` | JS/TypeScript SDK | TypeScript |
| `openyodel/swift` | iOS/macOS SDK | Swift |
| `openyodel/kotlin` | Android SDK | Kotlin |
| `openyodel/embedded` | Hardware/IoT SDK | C (Phase 3) |

### 16.2 Shared Resources

| Location | Content |
|----------|---------|
| `openyodel/spec` | Yodel Protocol Specification + OpenAPI |
| `openyodel/spec/test-fixtures/` | Shared test data (JSON) |
| `openyodel/.github` | This design guide, roadmap, org profile |
