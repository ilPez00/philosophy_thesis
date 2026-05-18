# RACHMANINOV — The Bridge

> *"Language, logistics, coordination, API calls." — ayu.md §4.3*  
> *"Will, action, experience, the a priori." — the Human*

---

## 1. Why the Name

Sergei Rachmaninoff composed at the hinge between the Romantic and the Modern. His music did not belong fully to either tradition — it metabolized one and fed the other. It was, structurally, a bridge.

This component does the same. It stands between raw observation (Aura) and structured social commitment (Praxis). It takes the noise of a life — a photo of a barbell, a word spoken, a GPS trace, a YouTube title — and renders it legible in a shared vocabulary of human action.

Without Rachmaninov, Aura and Praxis are isolated. With it, every observation becomes a potential check-in against a goal.

---

## 2. The Core Problem: Bridging Signal to Intent

A person is at a gym. Their phone sees a barbell. Their voice says "protein shake." GPS places them near a fitness center. Strava is open.

The question Rachmaninov must answer: **which goal does this serve, and how strongly?**

This is not a pattern-matching problem. It is a semantic alignment problem between:
- An **observed world state** (image, audio, location, motion, app context)
- A **declared intent** (goal name, domain, progress, history)

Rachmaninov answers it deterministically — no LLM inference needed for scoring — by maintaining a shared ontology that maps observable signals to action domains.

---

## 3. The Action Schema

Every human action, in this system, is decomposed into five dimensions:

```
ActionDomain    FABRICATE | STUDY | CONSTRUCT | BOND | HEAL
ActionMode      LIFT | WALK | WORK | LEARN | CODE | CREATE | REST
ActionScope     PERSONAL | COLLABORATIVE | COMMUNAL
ActionTrigger   PROPOSAL | SELF | EXTERNAL
OutcomeType     COMPLETION | PROGRESS | DISCOVERY | MAINTENANCE
GradeRationale  COMPLETENESS | EFFORT | OUTCOME | CONSISTENCY
```

These are not arbitrary. They derive from a view of human flourishing that:
- Treats production, learning, building, relating, and healing as the five fundamental modes of engagement with the world
- Distinguishes the register in which an action occurs (personal, social, civic)
- Separates *how* an action was triggered from *what kind of outcome it produced*

This schema is the common language. Aura scores in it. Praxis tracks in it. Rachmaninov owns it.

---

## 4. The 14 Goal Domains

Maslow's hierarchy of needs, operationalized into 14 trackable domains. Each domain maps to one `DomainDef` containing: AYU action primitive, default mode, score axis, unit of measurement, and context keywords.

```
Level 1 — Physiological
  Body & Fitness       HEAL/LIFT      · physical      · reps
  Rest & Recovery      HEAL/REST      · physical      · hours
  Mental Balance       HEAL/REST      · psychological · min

Level 2 — Safety
  Environment & Home   CONSTRUCT/CREATE · psychological · tasks
  Health & Longevity   HEAL/WALK      · physical      · steps
  Financial Security   FABRICATE/WORK · economic      · €

Level 3 — Belonging
  Friendship & Social  BOND/REST      · psychological · contacts
  Romance & Intimacy   BOND/REST      · psychological · quality-time
  Community            BOND/CREATE    · psychological · hours

Level 4 — Esteem
  Career & Craft       FABRICATE/WORK · intellectual  · deliverables
  Wealth & Assets      FABRICATE/WORK · economic      · €
  Gaming & Esports     STUDY/CREATE   · intellectual  · hours

Level 5 — Transcendence
  Impact & Legacy      CONSTRUCT/CREATE · intellectual · projects
  Spirit & Purpose     STUDY/LEARN    · psychological · pages
```

Four score axes: **PHYSICAL · ECONOMIC · INTELLECTUAL · PSYCHOLOGICAL**

These axes allow cross-domain comparison and aggregate life-balance metrics without reducing disparate goals to a single dimension.

---

## 5. The Visual Hints Map

130+ observable keywords, each mapped to a domain. This is the bridge from raw perception to semantic category.

A selection:

```
Vision / audio signal        →  Domain matched
─────────────────────────────────────────────────
"dumbbell" / "squat" / "rep" →  Body & Fitness
"meal" / "cooking" / "diet"  →  Health & Longevity
"book" / "reading" / "notes" →  Spirit & Purpose
"laptop" / "code" / "commit" →  Career & Craft
"receipt" / "wallet" / "bill"→  Financial Security
"guitar" / "canvas" / "paint"→  Impact & Legacy
"breathing" / "therapy"      →  Mental Balance
"trading" / "portfolio"      →  Wealth & Assets
"bed" / "sleeping" / "rest"  →  Rest & Recovery
"controller" / "console"     →  Gaming & Esports
```

These are not ML features. They are handcrafted, reviewed, versioned. The ontology is a *design decision about what counts as evidence*, made explicit.

---

## 6. The Scoring Pipeline

```
WikiPage ingested
       │
       ▼
bestMatch(page, activeGoals):

  text = title + body + tags + contextTokens()
  isRich = source ∈ {vision, audio, screen}

  for each active goal:
    def = resolveDomain(goal.domain)

    hintHits   = def.contextHints.count { it in text }   × 0.20
    nameHits   = goal.name.tokens.count { it in text }   × 0.25
    visualHits = if isRich: VISUAL_HINTS[domain].count { it in text } × 0.35

    score = sum(above)
    if source == sensor:  score × 1.30   (GPS/IMU = stronger signal)
    if isRich:            score × 1.10   (vision/audio > text-only)

    signalTypes = (hintHits > 0) + (nameHits > 0) + (visualHits > 0)

  → best goal by score

  Accept iff:
    score ≥ 0.50  AND
    signalTypes ≥ 2  AND       (require multi-signal convergence)
    DebounceLedger.allow(30min, goalId)  (one prompt per 30 min per goal)

  → PromptOverlay: "Did this count toward [Goal]?"
  → One tap confirms check-in
```

The multi-signal requirement (≥2 independent signal types) prevents false positives. A single keyword in a long document should not trigger a goal check-in. Convergent evidence — name + hint + visual, or GPS + name + audio — is required.

---

## 7. Context Tokens

At every scoring call, all live sensor state is flattened to a string token list:

```kotlin
contextTokens():
  locationName.words()           // ["gold's", "gym"]
  sensorMetrics.activity.label   // "running"
  audio.removePrefix("Audio: ")  // ["protein", "shake", "post-workout"]
  scene.words()                  // ["barbell", "rack", "plates"]
  foregroundApp.words()          // ["strava"]
```

These tokens augment the wiki page text before scoring. The full life context — not just the page content — participates in every domain match.

This is why a photo of a meal, by itself, might score 0.30 for "Health & Longevity." The same photo, taken while the user says "post-workout protein," GPS at a gym, motion=walking, scores 0.87. The context is the signal.

---

## 8. The ACT Feedback Loop

Rachmaninov includes a second layer: **ACT** (Adaptive Consistency Tracker). This reads historical check-in grades from WikiStore and generates insights that modify day proposals.

```
RachmaAct.computeStats(wikiStore):
  For each goal, last 90 days of graded check-ins:
    avgGrade             (0.0–1.0)
    consistency          (fraction of days with a check-in)
    trend                (last-7d vs prior-7d slope)
    stagnatingGoalIds    (consistency < 0.20 AND trend < 0)
    globalHealthScore    (weighted average)
```

Stagnating goals receive modified proposals:
- **Mode** downgraded to REST
- **Duration** shortened to 15 min
- **Action text** changes to: *"Recover: light session for [goal] — reset and rebuild consistency"*

The system learns that forcing continuation on a stalled goal is counterproductive. It proposes a recovery session instead.

---

## 9. The Day Proposal Engine (Rachmaninov.proposeDay)

```
proposeDay(goals, contextTokens):

  1. ACT stats → inject stagnation insights into LLM prompt
  2. ontology enrichment → each goal gets domain/mode/scoreAxis pre-resolved
  3. contextTokens → rank goals by current context relevance
  4. if LLM available:
       LLM fills ONLY: actionText (specific sentence), durationMin, scope
       ontology fields (domain/mode/scoreAxis) are NOT overrideable by LLM
     else:
       ontologyFallback() → instant, deterministic, no API call needed
  5. merge: LLM actionText + ontology domain = ProposedAction
```

The architectural choice: **ontology owns semantics; LLM owns language**.

The LLM is not asked "what domain is fitness?" — it is told. The LLM is asked "given that this is a HEAL/LIFT goal with unit=reps, write one crisp action sentence for today." This separation:
- Guarantees consistent domain classification across users and models
- Prevents hallucinated domains
- Makes the proposal deterministic when no LLM is available

---

## 10. Live Sync Architecture

The ontology is not static. It evolves — new domains, new keywords, adjusted weights. These changes must propagate to all Aura clients without a new app release.

```
Praxis server
  GET /api/rachmaninov/ontology
  → { version, updatedAt, ontology{}, visualHints{} }
  → Cache-Control: max-age=3600
  → ETag: "1.0.0"
         │
         │ Aura app startup (async, non-blocking)
         ▼
RachmaninoffSync.sync():
  conditional GET with If-None-Match
  304 → load from filesDir/rachmaninov_ontology.json (cached)
  200 → parse, write cache, update liveOntology / liveVisualHints
  error → fall back to bundled PraxisOntology.kt constants

PraxisEventBridge reads liveOntology / liveVisualHints
→ domain changes ship without app release
```

---

## 11. The Unbuilt Rachmaninov

The current implementation covers scoring and proposal. The full vision extends further:

```
RACHMANINOV ENGINE (planned)

  Task sensitivity classifier
    LOW  → cloud model (Gemini/GPT-4o)
    MED  → anonymize PII, then cloud
    HIGH → local model (Gemma, on-device)

  Anonymization pipeline
    personal wiki → strip names/places/IDs
    → goal wiki (for Praxis)
    → strip biometrics/exact location
    → community wiki (anonymized flow only)

  Community training loop
    Axiom reads anonymized community flows
    Synthesizes: "what works for domain X, archetype Y"
    Writes back compressed insight
    Never exposes individual records

  Dynamic model routing
    which LLM × which task × which data sensitivity
```

This is the privacy orchestrator: the entity that decides what a user's data is allowed to become, and routes each inference accordingly.

The philosophical claim is strong: **privacy is not a constraint on the system's capability. It is the system's constitutive architecture.** A user who cannot trust that their most sensitive moments stay local will not use the system with honesty. Honest use is the only use that produces valuable data. Therefore privacy maximization is also capability maximization.
