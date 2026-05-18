# RACHMANINOV — The Bridge

> *"Language, logistics, coordination, API calls." — ayu.md §4.3*  
> *"Will, action, experience, the a priori." — the Human*

---

## 1. Why the Name

Sergei Rachmaninoff composed at the hinge between the Romantic and the Modern. His music did not belong fully to either tradition — it metabolized one and fed the other. It was, structurally, a *bridge*.

This component does the same. It stands between raw observation (Aura) and structured social commitment (Praxis). It takes the noise of a life — a photo of a barbell, a word spoken, a GPS trace, a YouTube title — and renders it legible in a shared vocabulary of human action.

Without Rachmaninov, Aura and Praxis are isolated. With it, every observation becomes a potential check-in against a goal.

---

## 2. Position in the Stack

```
AURA (observer)
  │
  │  produces: WikiPage{source, body, tags, location, activity}
  ▼
RACHMANINOV (bridge)
  │
  │  provides: PraxisOntology, VISUAL_HINTS, scoring algorithm
  │  consumes: WikiPage + contextTokens + activeGoals
  │  produces: bestMatch(goal, score), ProposedAction
  ▼
PRAXIS (arena)
  │
  │  consumes: check-in (goalId, progressDelta, grade, actorType)
  │  produces: W_j adjustment, streak, Axiom coaching
  ▼
USER (agent)
  │
  │  receives: PromptOverlay (one tap check-in), morning brief
  └──► takes action → loop begins again
```

---

## 3. The Core Problem: Bridging Signal to Intent

A person is at a gym. Their phone sees a barbell. Their voice says "protein shake." GPS places them near a fitness center. Strava is open.

The question Rachmaninov answers: **which goal does this serve, and how strongly?**

This is not pattern-matching. It is semantic alignment between:
- An **observed world state** (image, audio, location, motion, app)
- A **declared intent** (goal name, domain, progress, history)

```
SIGNAL SPACE                    INTENT SPACE
────────────                    ────────────
vision: "barbell rack"          goal: "5×5 strength program"
audio:  "leg day"               domain: "Body & Fitness"
GPS:    "Gold's Gym, Milan"     HEAL / LIFT / reps
IMU:    WALKING                 W_j = 1.1
app:    Strava
                │
                ▼
     RACHMANINOV SCORING
     score = 0.858, signalTypes = 3
                │
                ▼
     PromptOverlay → one tap
```

---

## 4. The AYU Action Schema

Every human action decomposes into 5 orthogonal dimensions:

```
DIMENSION       VALUES
─────────────   ──────────────────────────────────────────────────────
ActionDomain    FABRICATE | STUDY | CONSTRUCT | BOND | HEAL
ActionMode      LIFT | WALK | WORK | LEARN | CODE | CREATE | REST
ActionScope     PERSONAL | COLLABORATIVE | COMMUNAL
ActionTrigger   PROPOSAL | SELF | EXTERNAL
OutcomeType     COMPLETION | PROGRESS | DISCOVERY | MAINTENANCE
GradeRationale  COMPLETENESS | EFFORT | OUTCOME | CONSISTENCY
```

These are not arbitrary. The schema reflects a view of human engagement:

```
ActionDomain — what kind of engagement with the world
  FABRICATE  making things that generate economic value
  STUDY      acquiring knowledge and capability
  CONSTRUCT  building systems, relationships, environments
  BOND       deepening connection with others
  HEAL       maintaining and restoring biological/psychological health

ActionMode — the register of the body/mind during action
  LIFT / WALK → physical exertion modes
  WORK / CODE → productive output modes
  LEARN / CREATE → generative cognitive modes
  REST → recovery and integration
```

---

## 5. The 14 Goal Domains

Maslow's hierarchy, operationalized into 14 trackable `DomainDef` entries:

```
MASLOW LEVEL 1 — Physiological
  ┌─────────────────────────────────────────────────────────────────┐
  │ Body & Fitness     HEAL/LIFT    PHYSICAL       unit: reps       │
  │ Rest & Recovery    HEAL/REST    PHYSICAL       unit: hours      │
  │ Mental Balance     HEAL/REST    PSYCHOLOGICAL  unit: min        │
  └─────────────────────────────────────────────────────────────────┘

MASLOW LEVEL 2 — Safety
  ┌─────────────────────────────────────────────────────────────────┐
  │ Environment & Home CONSTRUCT/CREATE PSYCHOLOGICAL unit: tasks   │
  │ Health & Longevity HEAL/WALK    PHYSICAL       unit: steps      │
  │ Financial Security FABRICATE/WORK ECONOMIC     unit: €         │
  └─────────────────────────────────────────────────────────────────┘

MASLOW LEVEL 3 — Belonging
  ┌─────────────────────────────────────────────────────────────────┐
  │ Friendship & Social BOND/REST   PSYCHOLOGICAL  unit: contacts   │
  │ Romance & Intimacy  BOND/REST   PSYCHOLOGICAL  unit: quality-t  │
  │ Community           BOND/CREATE PSYCHOLOGICAL  unit: hours      │
  └─────────────────────────────────────────────────────────────────┘

MASLOW LEVEL 4 — Esteem
  ┌─────────────────────────────────────────────────────────────────┐
  │ Career & Craft     FABRICATE/WORK INTELLECTUAL unit: deliverbl  │
  │ Wealth & Assets    FABRICATE/WORK ECONOMIC     unit: €         │
  │ Gaming & Esports   STUDY/CREATE  INTELLECTUAL  unit: hours      │
  └─────────────────────────────────────────────────────────────────┘

MASLOW LEVEL 5 — Transcendence
  ┌─────────────────────────────────────────────────────────────────┐
  │ Impact & Legacy    CONSTRUCT/CREATE INTELLECTUAL unit: projects │
  │ Spirit & Purpose   STUDY/LEARN   PSYCHOLOGICAL  unit: pages     │
  └─────────────────────────────────────────────────────────────────┘

SCORE AXES (for aggregate life-balance metrics):
  PHYSICAL · ECONOMIC · INTELLECTUAL · PSYCHOLOGICAL
```

---

## 6. Visual Hints Map

130+ observable keywords, each mapped to a domain. The bridge from raw perception to semantic category.

```
VISION / AUDIO SIGNAL                  DOMAIN MATCHED
──────────────────────────────────────  ───────────────────
"dumbbell" / "barbell" / "squat"        Body & Fitness
"meal" / "plate" / "cooking" / "diet"   Health & Longevity
"book" / "reading" / "notes" / "study"  Spirit & Purpose
"laptop" / "terminal" / "coding"        Career & Craft
"receipt" / "invoice" / "wallet"        Financial Security
"guitar" / "canvas" / "painting"        Impact & Legacy
"candle" / "breathing" / "therapy"      Mental Balance
"chart" / "trading" / "crypto"          Wealth & Assets
"bed" / "pillow" / "resting"            Rest & Recovery
"controller" / "headset" / "console"    Gaming & Esports
"friends" / "dinner" / "social"         Friendship & Social
"apartment" / "cleaning" / "repair"     Environment & Home
"medicine" / "doctor" / "sleep"         Health & Longevity
"partner" / "date" / "relationship"     Romance & Intimacy
```

These are design decisions about what counts as evidence. Handcrafted, versioned, reviewed.

---

## 7. Scoring Pipeline

```
INPUT: WikiPage + contextTokens + List<ActiveGoal>

def bestMatch(page, goals):
    text = page.title + " " + page.body + " " + page.tags.join(" ")
           + " " + contextTokens().join(" ")
    isRich = page.source in ["vision", "audio", "screen"]
    best = null
    bestScore = 0.0

    for goal in goals:
        def = resolveDomain(goal.domain)
        if def == null: continue

        # Three independent signal types
        hintHits   = count(h for h in def.contextHints if h in text)
        nameHits   = count(t for t in tokenize(goal.name) if t in text)
        visualHits = count(v for v in VISUAL_HINTS[def.domain] if v in text)
                     if isRich else 0

        # Weighted sum
        rawScore = hintHits × 0.20 + nameHits × 0.25 + visualHits × 0.35

        # Source bonuses
        if page.source == "sensor": rawScore × 1.30
        if isRich:                  rawScore × 1.10

        # Count distinct signal types (anti-noise guard)
        signalTypes = (hintHits > 0) + (nameHits > 0) + (visualHits > 0)

        if rawScore > bestScore:
            best = (goal, rawScore, signalTypes)
            bestScore = rawScore

    if best == null: return null
    goal, score, signalTypes = best

    # Acceptance criteria
    if score >= 0.50          # minimum evidence
    and signalTypes >= 2      # multi-signal convergence required
    and DebounceLedger.allow(goal.id, windowMin=30):  # 30-min cooldown
        return MatchResult(goal=goal, score=score)

    return null
```

The multi-signal requirement (`signalTypes ≥ 2`) is critical. A single keyword appearing in a 200-word body should not trigger a goal check-in. Convergent evidence — two or three independent signal channels pointing at the same domain — is required.

---

## 8. Context Tokens

```
LIVE SENSOR STATE → FLAT STRING TOKENS:

contextTokens():
  locationName.words()             ["gold's", "gym"]
  sensorMetrics.activity.label     "walking"
  audio.removePrefix("Audio: ")    ["protein", "shake", "post-workout"]
  scene.words()                    ["barbell", "rack", "plates", "gym"]
  foregroundApp.words()            ["strava"]

  → ["gold's", "gym", "walking", "protein", "shake",
     "post-workout", "barbell", "rack", "plates", "strava"]

These tokens are appended to page text before scoring.
Result: full life context participates in every domain match.

WHY THIS MATTERS:
  photo of a meal:
    text only: "food, plate, vegetables" → score=0.22 (Health & Longevity)
    + context: "gym" in locationName, "post-workout" in audio
    → score=0.71 (MATCHED — multi-signal convergence)
```

---

## 9. The Day Proposal Engine

```
Rachmaninov.proposeDay(goals, contextTokens):

  STEP 1: ACT stats (Adaptive Consistency Tracker)
  ─────────────────────────────────────────────────
  actStats = RachmaAct.computeStats(wikiStore)
    for each goal, last 90 days:
      avgGrade      = mean(check_in.grade)
      consistency   = days_with_checkin / 90
      trend         = mean(last7d.grade) - mean(prev7d.grade)
      stagnating    = consistency < 0.20 AND trend < 0

  actInsight = format:
    "Stagnating goals: body-fitness (0 check-ins/7d, trend -0.12)"
    "Thriving goals: career-craft (consistency=0.85, trend +0.08)"

  STEP 2: Context-ranked goals
  ─────────────────────────────
  ranked = rankByContext(goals, contextTokens)
    score_j = count(token in contextTokens for token in goal.name.tokens
                    + def.contextHints)
    sort descending by score_j

  STEP 3: Prompt construction
  ────────────────────────────
  enriched = for each goal:
    def = resolveDomain(goal.domain)
    "[{goal.name}] domain={def.ayuDomain}/{def.defaultMode}
     axis={def.scoreAxis} unit={def.unit}
     progress={goal.progress*100:.0f}%
     {stagnating: '⚠ STAGNATING' if goal.id in actStats.stagnatingGoalIds}"

  prompt = """
  You are Rachmaninov — action-proposal engine. Not an assistant.
  Context signals: {contextTokens.join(", ")}
  Today: {date}
  {actInsight}
  Goals:
  {enriched}

  Output ONLY a JSON array. No markdown, no prose.
  Fields: goalId, actionText (ONE crisp sentence), durationMin, scope.
  Bad: "work on fitness goals"
  Good: "Do 5×5 barbell squat at 82.5kg — add 2.5kg from last session"
  """

  STEP 4: LLM fills language; ontology owns semantics
  ─────────────────────────────────────────────────────
  slots = llm.complete(prompt)
  # parse JSON: [{goalId, actionText, durationMin, scope}]

  for slot in slots:
      goal = goalMap[slot.goalId]
      def  = resolveDomain(goal.domain)
      yield ProposedAction(
          goalId      = goal.id,
          goalName    = goal.name,
          domain      = def.ayuDomain,    # ontology owns this — LLM cannot override
          mode        = def.defaultMode,  # ontology owns this
          scope       = slot.scope,       # LLM fills this
          durationMin = slot.durationMin, # LLM fills this (5–180 range enforced)
          actionText  = slot.actionText,  # LLM fills this
          scoreAxis   = def.scoreAxis,    # ontology owns this
          unit        = def.unit,         # ontology owns this
      )

  STEP 5: Stagnation override
  ────────────────────────────
  if goal.id in actStats.stagnatingGoalIds:
      slot.mode        = REST            # lighter session
      slot.durationMin = 15              # shorter
      slot.actionText  = "Recover: light session for {goal.name}
                          — reset and rebuild consistency"
```

---

## 10. ACT — Adaptive Consistency Tracker

```
RachmaAct.computeStats(wikiStore) → ActStats:
  
  For all wiki pages with source="conversation"|"audio"|"vision":
    extract grade if present (from check-in notes pattern)
    group by goalId

  ActStats:
    avgGradeByGoal:     { goalId → float }
    consistencyByGoal:  { goalId → float }   (days with entry / 90)
    trendByGoal:        { goalId → float }   (last7d - prev7d slope)
    stagnatingGoalIds:  Set<goalId>          (consistency<0.20 AND trend<0)
    globalHealthScore:  float                (weighted avg consistency)
    activeDayStreak:    int                  (consecutive days with any entry)

actInsight(stats) → string for LLM prompt:
  if stagnatingGoalIds.isEmpty:
      "All goals showing positive consistency."
  else:
      "⚠ Stagnating: {stagnatingGoalIds.join(', ')}.
       Consider REST sessions. Rebuild consistency before pushing harder."
```

---

## 11. Live Ontology Sync

```
SYNC PROTOCOL:

  Praxis server exposes:
    GET /api/rachmaninov/ontology
    → HTTP 200: { version, updatedAt, ontology{}, visualHints{} }
    → HTTP 304: Not Modified (ETag match)
    → Cache-Control: max-age=3600

  RachmaninoffSync.sync(context):
    1. load cachedEtag from filesDir/rachmaninov_ontology.json
    2. GET /api/rachmaninov/ontology
       headers: { "If-None-Match": cachedEtag }

    if 304: load from cache (no network parsing overhead)
    if 200: parse JSON
            write to filesDir/rachmaninov_ontology.json
            update liveOntology StateFlow
            update liveVisualHints StateFlow
    if error: fall back to bundled PraxisOntology.kt constants
              (always available, compiled into APK)

  PraxisEventBridge reads:
    RachmaninoffSync.liveOntology.value
    RachmaninoffSync.liveVisualHints.value

  RESULT:
    Domain changes (new keywords, weight adjustments) ship
    server-side without requiring an app release.
    ETag prevents redundant parsing on unchanged ontology.
```

---

## 12. The Full Rachmaninov Vision (Unbuilt)

Current implementation: scoring, proposals, ACT, live sync.  
Full vision: inference router + privacy orchestrator.

```
┌─────────────────────────────────────────────────────────────────┐
│  RACHMANINOV ENGINE (planned)                                   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  TASK SENSITIVITY CLASSIFIER                            │   │
│  │                                                         │   │
│  │  LOW:  "What's the weather?"                            │   │
│  │        → cloud model (Gemini/GPT-4o)                    │   │
│  │                                                         │   │
│  │  MED:  "What did I say to my partner last week?"        │   │
│  │        → anonymize PII → cloud model                    │   │
│  │                                                         │   │
│  │  HIGH: "Analyze my depression patterns"                 │   │
│  │        → local model (Gemma, on-device)                 │   │
│  │        → never leaves device                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ANONYMIZATION PIPELINE                                 │   │
│  │                                                         │   │
│  │  personal wiki                                          │   │
│  │     │ strip: names, GPS coords, phone numbers,          │   │
│  │     │        medical details, financial specifics       │   │
│  │     ▼                                                   │   │
│  │  goal wiki (per-user, Praxis DB, private)               │   │
│  │     │ strip: biometrics, exact location                 │   │
│  │     ▼                                                   │   │
│  │  community wiki (global, anonymized)                    │   │
│  │     domain + mode + grade + outcome ONLY                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  COMMUNITY TRAINING LOOP                                │   │
│  │                                                         │   │
│  │  Axiom reads community wiki:                            │   │
│  │    "users matching archetype=Hero, domain=Career,       │   │
│  │     who do focused 90min deep-work sessions 3×/week     │   │
│  │     outperform random-schedule workers by 0.3σ"         │   │
│  │                                                         │   │
│  │  Writes back compressed insight (no individual data)    │   │
│  │  Injected into next coaching session                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

PHILOSOPHICAL CLAIM:
  Privacy is not a constraint on capability.
  Privacy IS the constitutive architecture.

  A user who cannot trust that their most sensitive moments
  stay local will not use the system honestly.
  Honest use is the only use that produces valuable data.
  Therefore: privacy maximization = capability maximization.
```

---

## 13. DomainDef Structure

```typescript
interface DomainDef {
    ayuDomain:    ActionDomain;   // FABRICATE|STUDY|CONSTRUCT|BOND|HEAL
    defaultMode:  ActionMode;     // LIFT|WALK|WORK|LEARN|CODE|CREATE|REST
    scoreAxis:    ScoreAxis;      // PHYSICAL|ECONOMIC|INTELLECTUAL|PSYCHOLOGICAL
    unit:         string;         // reps|hours|steps|€|deliverables|pages|...
    contextHints: string[];       // domain-specific keyword list for scoring
}

const PRAXIS_ONTOLOGY: Record<string, DomainDef> = {
    "Body & Fitness":      { ayuDomain: HEAL,      defaultMode: LIFT,   ... },
    "Career & Craft":      { ayuDomain: FABRICATE, defaultMode: WORK,   ... },
    "Spirit & Purpose":    { ayuDomain: STUDY,     defaultMode: LEARN,  ... },
    "Financial Security":  { ayuDomain: FABRICATE, defaultMode: WORK,   ... },
    "Friendship & Social": { ayuDomain: BOND,      defaultMode: REST,   ... },
    "Mental Balance":      { ayuDomain: HEAL,      defaultMode: REST,   ... },
    // ... 14 total
};
```

Mirrored in Kotlin (`PraxisOntology.kt`) and TypeScript (`PraxisOntology.ts`). Single source of truth versioned in Praxis server, synced to Aura clients.
