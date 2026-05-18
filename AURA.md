# AURA — The Observer

> *"The camera is not a tool. It is the eye that never blinks."*

---

## 1. The Problem

Every phone interface is, philosophically, a *distraction device* — a sequence of applications competing for attention, each demanding the user enter its context, forget everything else, and interact on its terms. The human becomes a servant of the grid.

Aura proposes the inverse: the phone becomes *transparent*. The world remains primary. The device becomes a perceptual augmentation layer — seeing what the user sees, hearing what the user hears, quietly building context, never demanding attention it hasn't earned.

This is not UX improvement. It is a claim about the proper relationship between tool and agent.

---

## 2. Extended Mind

Andy Clark and David Chalmers (1998) asked: where does the mind end?

Their answer: at the boundary of what the agent *reliably and fluently uses to think*. If a notebook functions as memory — consulted, trusted, used for reasoning — then it is, functionally, part of the cognitive system.

Aura's **wiki** is this argument implemented in software.

Every observation — a meal photographed, a word spoken, a GPS trace, a screen glanced at — is distilled into a structured wiki page. Not archived. Distilled. A background compression model (running on-device during idle time) reads raw signal, extracts the semantically meaningful fraction, destroys the rest.

The result is a **growing compressed life-log** that:
- Fits in local storage
- Is queryable before every AI call (semantic keyword search)
- Grows unboundedly in coverage while staying bounded in size
- Belongs entirely to the user — never transmitted without consent

When Aura answers a question, it first retrieves the 4 most relevant wiki pages from this personal knowledge base and injects them before the LLM call. The AI answers with the user's own history as context.

This is not "memory." It is extended mind, implemented.

---

## 3. Architecture

```
SENSORS
  Camera → VisionAnalyzer (Gemini/GPT-4o)          — what am I looking at?
  Mic    → SpeechClient (Deepgram/Whisper/Groq)     — what am I saying?
  GPS    → SensorFusion                             — where am I?
  IMU    → SensorFusion → AthleticMetrics           — what is my body doing?
  Screen → AccessibilityService                     — what app am I using?
         ↓
CONTEXT BUFFER
  locationName · scene · audio · motion · activity · foregroundApp
  → flat contextTokens[] for downstream domain matching
         ↓
         ├──→ WIKI INGESTER → WikiStore (filesDir/wiki/)
         │       ingestAudio / ingestVision / ingestNote
         │       Jaccard dedup vs last 5 pages
         │       → WikiPage (slug, title, body, source, confidence, tags, GPS, activity)
         │
         └──→ AURA SESSION
                WikiContext.retrieve(query, 4) → [WIKI KNOWLEDGE] block
                LocalMemoryClient (entity + conversation memory)
                AgentLoop → OpenAiBackend (35+ provider fallback chain)
                TtsEngine (ElevenLabs → Android TTS)
```

---

## 4. The Screen as Context, Not Destination

Default state in Aura is the **live camera feed**.

Information overlays it the way a thermal map overlays terrain — present when needed, absent when not. There are no application icons on a blank background. There is no home screen in the traditional sense. There is the world, and there is the agent's current context projected onto it.

### 4.1 HUD Layer

A minimal heads-up display persists at the top of the screen:
- Current mode (ACTIVE / PASSIVE)
- Top goal by context relevance (colour-coded dot + name + progress %)
- Sensor readings (GPS lock, battery, step count)

### 4.2 Active vs Passive Mode

| Mode | Behaviour |
|---|---|
| **Active** | Voice Activity Detection auto-triggers recording. 120fps AR. Auto TTS response. Vision cycle every 1–5s. Green perimeter glow. |
| **Passive** | Tap to capture. 60s silent context refresh. Manual record only. No visual noise. |

Active mode is the field mode — outdoor, moving, talking. Passive mode is the default — ambient awareness without intrusion.

### 4.3 The Launcher

Aura is a full Android launcher (`CATEGORY_HOME`). It replaces the default home screen.

```
┌──────────────────────────────────┐
│  [camera feed — always live]     │
│  ┌──────────────────────────┐    │
│  │  HUD: MODE · GOAL · GPS  │    │
│  └──────────────────────────┘    │
│                                  │
│  [widget strip — live sensors]   │  battery, steps, speed, altitude, cost
│                                  │
│  [app grid — draggable, folders] │
│                                  │
│  [applet bar]                    │  Maps, Praxis, Calendar, Terminal...
└──────────────────────────────────┘
```

Apps are plumbing, not destinations. They live in the grid at the bottom. The screen belongs to the world.

---

## 5. Privacy Architecture

Every piece of data in Aura is local-first:

- Wiki pages → device filesystem (`filesDir/wiki/`)
- Entity memory → device SQLite
- Conversation history → device memory

Data flows to Praxis only per **explicit per-goal opt-in**, and only the goal-relevant fraction. Personal identifiers are stripped by Rachmaninov before any community-level aggregation.

The compression loop (MemoryCompressor, 3h idle) runs on-device. Raw audio transcripts, vision frames, and conversation logs are destroyed after distillation. Only the distillate persists.

**The phone knows everything. The cloud knows what the user chose to share.**

---

## 6. The Second Brain — Technical Specification

### WikiPage schema
```yaml
slug:       gym-session-leg-day-20260518
title:      Gym session — leg day
source:     vision | audio | screen | conversation | compressed
confidence: 0.87
tags:       [fitness, training, "Body & Fitness"]
location:   45.4654,9.1866
activity:   WALKING
updated:    1716057600000
body:       >
  User at barbell squat rack. Plates ~80kg visible. Post-workout
  protein shake mentioned. Strava open in foreground.
```

### WikiStore operations
- `upsert(page)` — writes to `filesDir/wiki/{slug}.md`
- `search(query, n)` — keyword scoring (title match: +3pts, body: +1pt)
- `recentBySource(source, n)` — ordered by `updated` descending
- `ensureUniqueSlug(slug)` — appends `-2`, `-3`, etc.

### WikiContext
At every LLM call: `retrieve(query, 4)` → formats top-4 pages as:
```
[WIKI KNOWLEDGE]
2026-05-18 | gym-session | vision
Tags: fitness, training
User at squat rack. ~80kg. Post-workout shake...
---
```
This block is injected after `[MEMORY]` and before the user's query in the effective system prompt.

### MemoryCompressor
`CoroutineWorker`, scheduled every 3h on idle + battery-not-low:
1. Reads all wiki pages older than 2h (source: audio/vision/conversation), batch 8
2. Calls LLM: "distill to `[{fact, confidence, tags, entities}]`"
3. Writes one compressed page (source: compressed)
4. Deletes originals

Result: wiki grows in coverage, stays bounded in storage.

---

## 7. The Lattice — Physical World Extension

Aura can dispatch physical jobs to any device registered in Praxis Lattice (see PRAXIS.md):

**Voice command:** `"dispatch print_file to workshop-cnc"`

```
VoiceCommandRouter.parse()
  → VoiceCmd.Dispatch(jobType="print_file", deviceSlug="workshop-cnc")
  → PraxisApiClient.listDevices() → finds device by slug
  → PraxisApiClient.submitJob(deviceId, "print_file")
  → VisorScreen: shows "◈ print_file → Workshop CNC"
```

The phone becomes the command interface for a physical network. Aura sees the world; it can also *act* in it.

---

## 8. AI Provider Chain

Aura works with zero API keys. Provider fallback (implemented in `OpenAiBackend.kt`):

```
Tier 1 — keyed (configure in Settings)
  Gemini 2.5 Flash · Groq Llama 3.3 70B · DeepSeek · OpenRouter

Tier 2 — keyless free proxies (zero config)
  gptoss · g4f/auto · unfilteredapi · llm7 · ovhcloud · public Ollama

Tier 3 — additional keyed
  Cerebras · Mistral · GitHub Models · NVIDIA NIM · HuggingFace
  xAI · Together · SambaNova · Cohere · AIMLAPI · Fireworks

Vision
  Gemini 2.5 Flash → GPT-4o (OpenRouter) → Gemini 1.5 Pro
```

35+ providers. If the best option fails, the next is tried silently. The user never sees a provider switch.

---

## 9. Technical Stack

| Component | Technology |
|---|---|
| Language | Kotlin |
| UI | Jetpack Compose |
| AI inference | OpenAiBackend (35+ providers, KeyManager rotation) |
| Vision | CameraX + Gemini Vision API |
| STT | Deepgram / AssemblyAI / Groq Whisper |
| TTS | ElevenLabs / Android TTS |
| Wiki | Markdown + YAML frontmatter, local filesystem |
| Background work | WorkManager (MemoryCompressor, DriveSyncWorker) |
| Cloud sync | Google Drive REST API (`drive.appdata` scope) |
| Navigation | GPS + SensorFusion + OSRM routing |
| Maps | Google Places → Nominatim fallback |
| Search | Tavily → Brave → Serper → DuckDuckGo |

Source: `https://github.com/ilPez00/aura`
