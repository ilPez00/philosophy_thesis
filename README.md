# AYU — A Technical Philosophy of Intentional Living

**Author:** Giovanni Pezzin  
**Presented to:** Prof. Massimiliano Badino  
**Date:** May 2026

---

> *"Destroy the phone interface. Make the screen transparent."*

---

## What This Is

AYU is a three-layer computational system designed around a single thesis:

> **Passive observation → causal understanding → intentional action → measurable life improvement.**

It is not a productivity app. It is not a journal. It is an attempt to make the gap between *wanting* and *doing* legible — first to a machine, then to the person living it.

```
┌──────────────────────────────────────────────┐
│  AURA          — the observer                │
│  Always-on field sensor. AR launcher.        │
│  Sees what you see. Remembers what matters.  │
├──────────────────────────────────────────────┤
│  RACHMANINOV   — the bridge                  │
│  Shared ontology of human action.            │
│  Scores observations. Routes inference.      │
├──────────────────────────────────────────────┤
│  PRAXIS        — the arena                   │
│  Social commitment OS.                       │
│  Federated will. Axiom AI. Community data.   │
└──────────────────────────────────────────────┘
```

The full schema of a human action: `desire → intent → method → action → effect`

Aura captures the *method* and *action*. Rachmaninov classifies them. Praxis validates the *effect* and adjusts the *intent*.

---

## Documents

| Document | What It Covers |
|---|---|
| [AURA.md](AURA.md) | The observer. Extended mind. AR launcher. Wiki second brain. Local-first AI. |
| [RACHMANINOV.md](RACHMANINOV.md) | The bridge. Action ontology. 14 domains. Scoring pipeline. Privacy routing. |
| [PRAXIS.md](PRAXIS.md) | The arena. Social accountability. Axiom AI. Lattice physical network. |
| [DATAFLOW.md](DATAFLOW.md) | End-to-end: gym session → vision → scoring → check-in → Axiom brief. |
| [BUSINESS.md](BUSINESS.md) | Monetization, roadmap, and the competitive moat. |

---

## Philosophical Lineage

| Tradition | How It Appears |
|---|---|
| **Extended Mind** (Clark & Chalmers 1998) | Aura's wiki is memory outside the skull. Consulted, trusted, used for reasoning = part of the cognitive system. |
| **Aristotelian Praxis** | Action as its own end. The commitment to show up is the practice. The outcome is evidence of it, not its purpose. |
| **Husserlian epoché** | The AR overlay brackets the device interface. The world becomes primary; information is overlaid on it like a thermograph. |
| **Popperian falsifiability** | A goal that has never produced a check-in is not a goal — it is an intention. VALIDATED status requires external grading. |
| **Ariely on commitment devices** | Financial stakes increase follow-through ~40%. Bets are not gamification — they are skin in the game. |
| **Federated will** | Each person's goals are sovereign. The community validates effort, not outcomes. No central authority judges what is worth pursuing. |
| **Causal AI** | Current AI systems answer questions. AYU builds the *causal model of a person's life* before any question is asked. |

---

## Core Principles

**Intent over Application.** The user wants to "send a message to Mom," not "open WhatsApp." Aura abstracts intent; apps become invisible plumbing.

**The Screen as Context, Not Destination.** Default state is the camera feed. Information overlays it like heat on a thermograph — relevant, immediate, fading when not needed.

**Capture Everything, Distill to Essence.** Every tap, word, glance is potential signal. Raw data is ephemeral. The wiki is permanent. The gap is the compression model.

**Privacy is the Default, Sharing is Deliberate.** All data local by default. Data flows to Praxis only per explicit per-goal opt-in, stripped of personal identifiers. The community wiki receives only anonymized `will→action→effect` flows.

---

## Privacy Architecture

```
ON DEVICE (Aura — never leaves without consent)
  ├── Raw audio transcripts
  ├── Camera frames + scene descriptions
  ├── GPS traces
  ├── Wiki pages (all sources)
  ├── Entity memory
  └── Compressed distillates

PER-GOAL OPT-IN (user explicitly enables per goal)
  ├── Goal name + progress %
  ├── Check-in mood/grade
  └── Tracker data (steps, etc.)

ANONYMIZED ONLY (community wiki)
  ├── will→action→effect flows (no names, places, IDs)
  ├── domain + mode + grade + outcome
  └── anonymized biometrics (activity type, not location)

NEVER SHARED
  ├── Raw audio/video
  ├── GPS coordinates
  ├── Personal wiki pages
  └── Psychological profile (Big Five, archetypes)
```

The psychological profile (Big Five, Jungian archetypes, DSM-5 contextual) is always private. Used only for per-user coaching personalization. Never transmitted.

---

## Current Implementation Status

All three layers are fully implemented and running:

**Aura** — Android app (Kotlin/Jetpack Compose)
- Full Android launcher (`CATEGORY_HOME`)
- AR HUD with live vision analysis (Gemini/GPT-4o)
- Wiki second brain (local filesystem, background compression)
- 35+ LLM provider fallback chain (works keyless)
- Voice dispatch to Lattice devices
- Google Drive sync

**Rachmaninov** — shared semantic layer
- 14 domains, 130+ visual hints, full ayu action schema
- Live sync endpoint (ETag cache, bundled fallback)
- ACT feedback loop (stagnation detection)
- Day proposal engine (LLM fills language; ontology owns semantics)

**Praxis** — full-stack webapp
- React/Express/Supabase, deployed Railway + Vercel
- Axiom AI agent (nightly scan + on-demand coaching + tool calls)
- Duels, sparring, team challenges, accountability bets (Stripe)
- Lattice physical device network (CNC mills, printers, computers)
- MCP server (Claude Desktop integration)
- PAT system (any external agent can act on a user's behalf)

Source repositories:
- Aura: `https://github.com/ilPez00/aura`
- Praxis: `https://github.com/ilPez00/praxis-backend`
- Specs: `https://github.com/ilPez00/philosophy_thesis`
