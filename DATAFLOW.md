# AYU — End-to-End Data Flow

> A single concrete scenario: a gym session becomes a check-in, a coaching note, and a calibrated goal weight — without the user typing anything.

---

## The Scenario

A user is at the gym. They have one active goal: "Body & Fitness — 5×5 strength program."

---

## Step 1: Aura Sees the World

```
Camera → VisionAnalyzer (Gemini 2.5 Flash)
  "barbell rack detected. Plates approximately 80kg visible.
   User in lifting stance."

Mic → SpeechClient (Groq Whisper)
  User says: "leg day, let's go"

GPS → SensorFusion
  location: lat=45.4654, lon=9.1866
  → reverse geocode: "Gold's Gym, Milan"

IMU → SensorFusion → AthleticMetrics
  activity: WALKING (gym floor movement)

Accessibility → foreground app: Strava
```

---

## Step 2: WikiIngester Creates a Page

```yaml
slug:       gym-session-leg-day-20260518-001
title:      Gym session — leg day
source:     vision
confidence: 0.91
tags:       [fitness, training, "Body & Fitness"]
location:   45.4654,9.1866
activity:   WALKING
updated:    1716057600000
body:       >
  Barbell rack, approximately 80kg. User says "leg day, let's go."
  Strava open. Location: Gold's Gym, Milan.
```

Jaccard dedup check: this body is ≥40% different from the last 5 pages → not duplicate → written to `filesDir/wiki/`.

`onPageIngested` callback fires → PraxisEventBridge.

---

## Step 3: Rachmaninov Scores the Page

```
text = "gym session leg day barbell rack 80kg Strava Gold's Gym"
     + contextTokens(): ["gold's", "gym", "walking", "leg", "day", "barbell", "rack", "strava"]
isRich = true (source=vision)

for goal "Body & Fitness":
  def = DomainDef(
    ayuDomain=HEAL, defaultMode=LIFT, scoreAxis=PHYSICAL,
    unit=reps,
    contextHints=["gym", "workout", "fitness", "barbell", ...]
  )

  hintHits   = 4 ("gym", "workout", "barbell", "fitness")   → 4 × 0.20 = 0.80
  nameHits   = 2 ("body", "fitness")                        → 2 × 0.25 = 0.50
  visualHits = 3 ("barbell", "rack", "gym")                 → 3 × 0.35 = 1.05

  raw score = 0.80 + 0.50 + 1.05 = 2.35  (normalized: 0.78)
  × isRich:  0.78 × 1.1 = 0.858

  signalTypes = 3 (hint + name + visual)

threshold=0.50 ✓  signalTypes≥2 ✓  DebounceLedger.allow("body-fitness", 30min) ✓

→ MATCH: score=0.858, goal="Body & Fitness"
```

---

## Step 4: PromptOverlay

```
┌─────────────────────────────────────────────┐
│  ◉ gym session → Body & Fitness             │
│  Gold's Gym · leg day · barbell detected    │
│                                             │
│  [✓ Check in]          [Dismiss]            │
└─────────────────────────────────────────────┘
```

User taps ✓. One tap.

---

## Step 5: PraxisApiClient Posts Check-in

```
POST /api/checkins
  Authorization: pk_live_aura-field (PAT — actor_type='aura')
  {
    goalId: "body-fitness-uuid",
    progressDelta: 0.05,
    grade: null,      ← no grade yet; awaits partner review
    notes: "gym session — leg day. barbell rack visible.",
    actorType: "aura",
    agentName: "aura-field"
  }

PATCH /api/goals/body-fitness-uuid
  { progress: 0.52 }  ← was 0.47
```

GoalHudStrip in Aura HUD updates: `BODY FITNESS 52%`

---

## Step 6: Axiom Nightly Scan

That night:

```
AxiomProgressEstimationService:
  reads today's check-in → progress delta +0.05
  cross-references tracker (steps: 8,400) → consistent with gym visit

AxiomLearningService:
  leg day check-in is 4th in a row this week
  consistency this week: 4/4 → streak rising

AxiomDailySummaryService:
  "4-session week for Body & Fitness. Barbell work consistent.
   Consider adding accessory work — current logs show squat focus only."

AxiomPersonaService:
  grade history: OUTCOME=0.8, EFFORT=0.9, CONSISTENCY=0.85
  → W_j remains at 1.1 (MEDIOCRE status — not yet SUCCEEDED threshold)
```

---

## Step 7: Morning Brief (Rachmaninov.proposeDay)

```
Context tokens at 7am: ["stationary", "home", "morning"]
ACT stats: consistency=0.92, no stagnation

rankByContext(goals):
  Body & Fitness ranks #1 (gym-related tokens match)

buildPrompt → LLM call (Gemini 2.5 Flash):
  "Given domain=HEAL/LIFT, unit=reps, consistency=0.92,
   write one crisp action sentence for today."

LLM returns: "Do 5×5 barbell back squat at 82.5kg (add 2.5kg from last session)"
Ontology overrides: domain=HEAL, mode=LIFT, scoreAxis=PHYSICAL

ProposedAction:
  goalId:     "body-fitness-uuid"
  actionText: "Do 5×5 barbell back squat at 82.5kg (add 2.5kg from last session)"
  durationMin: 45
  scope:       PERSONAL
  unit:        reps
```

---

## Step 8: User Opens PraxisScreen in Aura

```
┌─────────────────────────────────────────────────┐
│  PRAXIS                                         │
│                                                 │
│  ● Body & Fitness         52%  ████████░░░      │
│    [LIFT] 5×5 squat 82.5kg · 45min · PERSONAL  │
│                                                 │
│  Streak: 4 days  ·  PP: 1,240  ·  LVL 7        │
└─────────────────────────────────────────────────┘
```

The user did not type anything. The entire loop — observe, score, check-in, brief, propose — ran passively.

---

## The Full Loop (Collapsed)

```
WORLD          AURA                RACHMANINOV         PRAXIS
─────          ────                ───────────         ──────

Gym visit
  │
  ├─ camera    VisionAnalyzer
  │  barbell ──► "barbell rack"     VISUAL_HINTS
  │                                 "Body & Fitness" ──► score=0.858
  │                                                      signalTypes=3
  │                                                      DebounceLedger.allow()
  │                                                      PromptOverlay
  │                                                           │ tap
  ├─ mic       WikiIngester     PraxisEventBridge             │
  │  "leg day"──► WikiPage ───────────────────────────► POST /checkins
  │                                                     PATCH /goals
  │                                                           │
  │             GoalHudStrip                          Axiom nightly:
  │             "BODY FITNESS 52%"                    summary + ACT stats
  │                                                           │
  └─ morning    PraxisScreen                          proposeDay():
     opens   ◄───────────────────────────────────     "5×5 squat 82.5kg"
```

One gym visit. Zero manual input. Full causal chain recorded.
