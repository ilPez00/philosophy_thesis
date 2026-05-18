# AURA — The Observer

> *"The camera is not a tool. It is the eye that never blinks."*

---

## 1. The Problem

Every phone interface is, philosophically, a *distraction device* — a sequence of applications competing for attention, each demanding the user enter its context, forget everything else, and interact on its terms. The human becomes servant of the grid.

Aura proposes the inverse: the phone becomes *transparent*. The world remains primary. The device becomes a perceptual augmentation layer — seeing what the user sees, hearing what the user hears, quietly building context, never demanding attention it hasn't earned.

This is not UX improvement. It is a claim about the proper relationship between tool and agent.

---

## 2. Extended Mind

Andy Clark and David Chalmers (1998) asked: where does the mind end?

Their answer: at the boundary of what the agent *reliably and fluently uses to think*. If a notebook functions as memory — consulted, trusted, used for reasoning — then it is, functionally, part of the cognitive system.

Aura's **wiki** is this argument implemented in software.

```
Traditional model:
  Brain ──► Phone (passive tool, queried on demand)

AYU model:
  Brain ──────────────────────────────► Intent
    ↑                                     │
  Wiki ◄── Phone (active observer,        │
  (external  always-on, enriches        Action
   memory)   every query with context)
```

Every observation is distilled into a wiki page. Not archived — *distilled*. A background compression model (on-device, idle time) reads raw signal, extracts the semantically meaningful fraction, destroys the rest.

---

## 3. Full Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│  SENSORS                                                                │
│                                                                         │
│  Camera ──► VisionAnalyzer ──► Gemini / GPT-4o                          │
│  Mic    ──► SpeechClient   ──► Deepgram / AssemblyAI / Groq Whisper      │
│  GPS    ──► LocationProvider ──► SensorFusion                           │
│  IMU    ──► SensorFusion ──► AthleticMetrics                            │
│             (activity: RUNNING / WALKING / DRIVING / GYM / ...)         │
│  Screen ──► AuraAccessibilityService ──► foreground app + text          │
│  Steps  ──► StepCounter ──► SensorFusion                                │
└─────────────────────────────────────────────────────────────────────────┘
                                 │ all signals
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  CONTEXT BUFFER                                                         │
│                                                                         │
│  gps · scene · audio · motion · locationName · motionActivity           │
│  accelerometer · gyroscope · light · proximity · stepCount              │
│                                                                         │
│  contextTokens() ──► flat List<String>                                  │
│    ["gold's", "gym", "walking", "barbell", "rack", "strava"]            │
└──────────────────────────────┬──────────────────────────────────────────┘
                               │
              ┌────────────────┴────────────────┐
              ▼                                 ▼
┌─────────────────────────┐       ┌─────────────────────────────────────┐
│  WIKI INGESTER           │       │  AURA SESSION                       │
│                          │       │                                     │
│  ingestAudio(transcript) │       │  systemPrompt                       │
│  ingestVision(scene)     │       │  + [MEMORY] (entity memory)         │
│  ingestNote(text)        │       │  + [WIKI KNOWLEDGE]                 │
│  ingestActivity(summary) │       │    WikiContext.retrieve(query, 4)   │
│                          │       │  + user query                       │
│  Jaccard dedup vs last 5 │       │         │                           │
│  Min length: 30 chars    │       │         ▼                           │
│  Tags auto-extracted     │       │  AgentLoop                          │
│                          │       │  ├── OpenAiBackend (35+ providers)  │
└──────────┬───────────────┘       │  ├── ToolRegistry (25+ tools)       │
           │ WikiPage              │  └── TtsEngine (ElevenLabs / TTS)   │
           ▼                       └─────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│  PRAXIS EVENT BRIDGE                                                 │
│                                                                      │
│  bestMatch(page, activeGoals) → score, signalTypes                   │
│  threshold=0.50 AND signalTypes≥2 AND DebounceLedger.allow(30min)    │
│  → PromptOverlay: "gym session → Body & Fitness [✓ Check in]"        │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 4. The Screen as Context, Not Destination

```
BEFORE AURA:

  ┌──────────────────────────────────────┐
  │  [WALLPAPER — static image]          │
  │                                      │
  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐         │
  │  │ 📱 │ │ 📷 │ │ 🗺️ │ │ 🎵 │         │  ← destination grid
  │  └────┘ └────┘ └────┘ └────┘         │    (each app = context switch)
  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐         │
  │  │ 💬 │ │ 📧 │ │ 🌐 │ │ ⚙️ │         │
  │  └────┘ └────┘ └────┘ └────┘         │
  └──────────────────────────────────────┘
  User attention: phone surface (dead, static)

AFTER AURA:

  ┌──────────────────────────────────────┐
  │  [LIVE CAMERA FEED — always on]      │  ← world is primary
  │                                      │
  │  ┌──────────────────────────────┐    │
  │  │ HUD: PASSIVE · FITNESS 52% ● │    │  ← minimal overlay
  │  └──────────────────────────────┘    │
  │                                      │
  │  [widget strip]  battery · steps     │  ← live data, not icons
  │                                      │
  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐         │
  │  │app │ │app │ │app │ │app │         │  ← apps below the world
  │  └────┘ └────┘ └────┘ └────┘         │
  │  [applet bar]                        │
  └──────────────────────────────────────┘
  User attention: world (through the phone)
```

---

## 5. Active vs Passive Mode

```
                    ┌─ ACTIVE MODE ──────────────────────────────┐
User in field ────► │                                            │
                    │  VAD auto-record (Voice Activity Detection) │
(moving, talking,   │  120fps AR rendering                        │
 outside)           │  Auto-TTS response on every reply          │
                    │  Vision cycle: every 1–5 seconds           │
                    │  Green perimeter glow                       │
                    │  GPS polling: high frequency                │
                    └────────────────────────────────────────────┘

                    ┌─ PASSIVE MODE ──────────────────────────────┐
User at desk ─────► │                                            │
                    │  Tap to capture (camera or mic)             │
(stationary,        │  60-second silent context refresh           │
 working, quiet)    │  No auto-TTS                                │
                    │  No automatic AR overlay                    │
                    │  GPS polling: low frequency                 │
                    └────────────────────────────────────────────┘

Swipe: VISOR ◄──► PRAXIS ◄──► SETTINGS ◄──► MEMORY
```

---

## 6. Second Brain — WikiStore

```
LIFECYCLE OF AN OBSERVATION:

  Raw signal (audio transcript / vision description / note)
         │
         ▼
  WikiIngester.ingest*():
    if len(text) < 30: skip
    if Jaccard(text, last5) > 0.7: skip (duplicate)
    tags = extractTags(text)
      ["conversation","shopping","reminder","location",
       "health","person","X"] ← auto-detected by keyword match
         │
         ▼
  WikiPage:
    ┌─────────────────────────────────────────┐
    │ slug:       gym-session-20260518-001     │
    │ title:      Gym session — leg day        │
    │ source:     vision                       │
    │ confidence: 0.91                         │
    │ tags:       [fitness, training, ...]     │
    │ location:   45.4654,9.1866               │
    │ activity:   WALKING                      │
    │ updated:    1716057600000                │
    │ body:       "Barbell rack, 80kg..."      │
    └─────────────────────────────────────────┘
         │
         ▼
  WikiStore.upsert(page) → filesDir/wiki/{slug}.md
         │
         ├──► onPageIngested callback → PraxisEventBridge
         └──► WikiContext cache invalidated
```

### WikiContext.retrieve(query, n=4)

```python
def retrieve(query: str, n: int = 4) -> str:
    pages = wiki_store.all()
    scores = []
    for page in pages:
        score = 0
        query_tokens = tokenize(query)
        for token in query_tokens:
            if token in page.title:
                score += 3      # title match = strong signal
            if token in page.body:
                score += 1
            if token in page.tags:
                score += 2
        if score > 0:
            scores.append((score, page))
    
    top = sorted(scores, reverse=True)[:n]
    
    return format_as_block(top)
    # → "[WIKI KNOWLEDGE]\n2026-05-18 | gym-session | vision\n..."
```

---

## 7. MemoryCompressor — Background Distillation

```
COMPRESSION PIPELINE:

  Raw wiki pages
  (source: audio, vision, conversation)
  older than 2 hours
         │
         │ trigger: WorkManager
         │ constraints: IDLE + BATTERY_NOT_LOW + CONNECTED
         │ interval: 3 hours
         ▼
  MemoryCompressor.doWork():

    batch = wiki_store.recentBySource(["audio","vision","conversation"])
              .filter(age > 2h)
              .take(8)

    prompt = """
    Compress these observations into key facts.
    Output JSON: [{fact, confidence, tags, entities}]
    Observations:
    {batch}
    """

    result = llm.complete(prompt)
    facts = json.parse(result)

    for fact in facts:
        wiki_store.upsert(WikiPage(
            source = "compressed",
            confidence = fact.confidence,
            body = fact.fact,
            tags = fact.tags,
        ))

    for original in batch:
        wiki_store.delete(original.slug)

  RESULT:
    8 raw pages (avg 200 chars each) → 1 compressed page (avg 150 chars)
    Storage: ~93% reduction
    Information: ~70% retained (key facts only)
```

```
BEFORE compression (8 pages, 1,600 chars total):
  "User said protein shake after gym"
  "Barbell visible, user lifting"
  "User mentioned leg day"
  "Strava app open during workout"
  ...

AFTER compression (1 page, 120 chars):
  fact: "Regular gym sessions, leg day focus, barbell/squat training,
         tracks via Strava. Post-workout: protein shake."
  confidence: 0.88
  tags: [fitness, training, strava, nutrition]
```

---

## 8. AI Provider Chain

```
PROVIDER SELECTION ALGORITHM:

  query arrives
       │
       ▼
  for provider in PRIORITY_ORDER:
      key = KeyManager.getKey(provider)
      if key == null and not provider.isKeyless: continue
      usage = KeyManager.usage[key]
      if usage.dead: continue
      if usage.cooldownUntil > now(): continue

      → ATTEMPT provider
      ├── success: KeyManager.reportSuccess(provider, key)
      │           return response
      └── error:
          ├── 401/403 → KeyManager.reportError("KEY_INVALID")
          │            → usage.dead = true
          ├── 429     → KeyManager.reportError("RATE_LIMIT")
          │            → cooldown = min(5s << errors, 120s)
          └── other   → KeyManager.reportError("GENERIC")
                       → cooldown = 5s

PRIORITY ORDER:
  Tier 1 (keyed):    gemini → vertex → groq → deepseek → openrouter
  Tier 2 (keyless):  gptoss → g4f/auto → unfilteredapi → llm7 → ovhcloud
  Tier 3 (keyed):    cerebras → mistral → github_models → nvidia_nim
                     → huggingface → xai → together → sambanova
                     → cohere → aimlapi → fireworks → ...

35+ providers. Silent failover. User never sees provider switches.
```

---

## 9. Tool System

Aura's agent can call 25+ tools. Categories:

```
FILESYSTEM          NETWORK             DEVICE
───────────         ───────────         ──────────
read_file           web_search          control_ui
write_file          get_weather         get_notifications
list_dir            find_places         reply_notification
run_shell           reverse_geocode     phone_task
read_source         get_navigation      phone_automation
write_source        translate_text
                    notify

MEMORY              DEV                 MESH
───────────         ───────────         ──────────
memory_save         dev_task            mesh_peers
memory_search       dev_status          mesh_delegate
import_uberwiki     dev_task_github
```

### Tool Execution Flow

```python
def execute(name: str, args: dict) -> str:
    # All tools follow:
    #   1. Validate args (return "Error: X required" if missing)
    #   2. Execute (may call network, device, filesystem)
    #   3. Return Compress.compress(result)
    #      ↑ LZ-string compression reduces tokens in next context window

    if name == "find_places":
        q = args.get("query") or return "Error: query required"
        gps = AuraApp.smartPhoneContext.sensorData.location
        lat = gps?.latitude or 0.0
        lon = gps?.longitude or 0.0
        return net.findPlaces(q, lat, lon)   # Google Places → Nominatim

    if name == "reverse_geocode":
        lat = args["lat"] or return "Error: lat required"
        lon = args["lon"] or return "Error: lon required"
        return net.reverseGeocode(lat, lon)  # Google Geocoding → Nominatim

    if name == "get_weather":
        gps = AuraApp.smartPhoneContext.sensorData.location
        lat = args.get("lat") or gps?.latitude or 0.0
        lon = args.get("lon") or gps?.longitude or 0.0
        return net.getWeather(lat, lon)      # Open-Meteo → OWM

    if name == "web_search":
        q = args["query"]
        return net.webSearch(q)              # Tavily → Brave → Serper → DDG

    if name == "notify":
        msg = args["message"]
        title = args.get("title", "Aura")
        return net.notify(msg, title)        # ntfy.sh → Telegram → Gotify
```

---

## 10. Lattice — Physical World Extension

Aura can dispatch physical jobs to any device registered in the Praxis Lattice:

```
VOICE COMMAND ROUTING:

  "dispatch print_file to workshop-cnc"
          │
          ▼
  VoiceCommandRouter.parse(utterance):

    DISPATCH_RE = r'^(?:dispatch|send|run|execute)\s+(.+?)\s+(?:to|on)\s+(.+)$'

    match: jobType="print_file", deviceSlug="workshop-cnc"
    → underscore conversion: "print file" → "print_file"

    return VoiceCmd.Dispatch(jobType="print_file", deviceSlug="workshop-cnc")

          │
          ▼
  VisorScreen handles VoiceCmd.Dispatch:

    devices = PraxisApiClient.listDevices()
    target = devices.find(slug contains "workshop-cnc"
                          OR name contains "workshop-cnc")

    if target == null: reply "Device 'workshop-cnc' not in lattice"
    else:
      jobId = PraxisApiClient.submitJob(target.id, "print_file")
      reply "◈ print_file → Workshop CNC"

SUPPORTED TRIGGER WORDS:
  dispatch / send / run / execute

SYNTAX:
  <trigger> <job_type> to <device>
  <trigger> <job_type> on <device>
```

---

## 11. Privacy Architecture

```
DATA CLASSIFICATION:

  ┌──────────────────────────────────────────────────────────┐
  │  NEVER LEAVES DEVICE                                     │
  │                                                          │
  │  ● Raw audio transcripts                                 │
  │  ● Camera frames + scene descriptions                    │
  │  ● GPS coordinates                                       │
  │  ● Wiki pages (all sources)                              │
  │  ● Entity memory                                         │
  │  ● Psychological profile (Big Five, archetypes)          │
  │  ● Compressed distillates                                │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  PER-GOAL OPT-IN (user explicitly enables per goal)      │
  │                                                          │
  │  ● Goal name + progress %                                │
  │  ● Check-in grade + notes                                │
  │  ● Tracker data (steps, weight, etc.)                    │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  ANONYMIZED ONLY (community wiki — no identifiers)       │
  │                                                          │
  │  ● will → action → effect flows                          │
  │  ● domain + mode + grade + outcome                       │
  │  ● Activity type (not GPS coordinates)                   │
  └──────────────────────────────────────────────────────────┘
```

The phone knows everything. The cloud knows what the user chose to share.

---

## 12. Aesthetic Specification

From ayu.md §1.2: *"iPhone fluidity + Anduril Lattice density."*

```
PRINCIPLES:
  120fps               seamless transitions, no jank
  Blur + translucency  camera feed breathes beneath UI
  Information density  HUD does not hide data to be pretty
  Glow effects         tapped object gets radial glow
  Everything earns its place
                       if element neither informs nor enables: remove

COLOUR SYSTEM:
  Active mode    green perimeter glow (#00FF88)
  Passive mode   no border
  Accent         #00FF88 (monochrome green on dark)
  Background     #0A0A0A (near-black, not pure black)
  Text           #DDDDDD (off-white, reduces eye strain)
  Dim            #4A4A4A (secondary info)
  Font           Monospace (data density, terminal aesthetic)

HUD DENSITY SPECTRUM:
  Minimal (passive, desk):
    [●] PASSIVE  ·  FITNESS 52%

  Dense (active, field):
    [●] ACTIVE  ·  FITNESS 52%  ·  GPS ✓  ·  4.7 km/h  ·  8,240 steps
    [barbell rack detected — leg press visible]
    [WIKI: 847 pages · today: 12 added]
```

---

## 13. Technical Stack

| Layer | Technology |
|---|---|
| Language | Kotlin |
| UI | Jetpack Compose |
| Camera | CameraX + ML Kit ObjectDetector |
| Vision AI | Gemini Vision API / GPT-4o |
| STT | Deepgram / AssemblyAI / Groq Whisper / Gladia / Speechmatics |
| TTS | ElevenLabs / Fish Audio / Android TTS |
| LLM inference | OpenAiBackend (35+ providers, KeyManager rotation) |
| Wiki | Markdown + YAML frontmatter, local filesystem |
| Background | WorkManager (MemoryCompressor, DriveSyncWorker, CompressionScheduler) |
| Cloud sync | Google Drive REST API (`drive.appdata`) |
| Navigation | OSRM (turn-by-turn routing, free, no key) |
| Geocoding | Google Places → Nominatim fallback |
| Maps | Google Maps API → Nominatim OSM fallback |
| Search | Tavily → Brave → Serper → DuckDuckGo |
| Notifications | ntfy.sh → Telegram → Gotify |
| Lattice | PraxisApiClient (listDevices, submitJob, listJobs) |
| Memory | WikiStore (filesystem) + LocalMemoryClient (SQLite) |

Source: `https://github.com/ilPez00/aura`
