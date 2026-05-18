# AYU — Business Model and Roadmap

---

## 1. Revenue Streams

```
STREAM                   MODEL              RATIONALE
───────────────────────  ─────────────────  ────────────────────────────────────
Praxis Pro subscription  €12/mo B2C         core recurring revenue
Accountability bets      5–10% fee          high-margin; behavioral lock-in
Aura Pro                 €6/mo add-on       high-LTV power users
Axiom B2B API            per-MAU licensing  medium-term; platform play
Coach marketplace        20–30% rev share   supply-side flywheel
Community wiki data      licensing          long-term; moat monetization
```

---

## 2. Praxis Tiers

```
FREE
  3 active goals · 1 accountability partner · manual check-ins
  basic Axiom daily summary · community matching

PRO (€12/mo)
  unlimited goals + partners · Aura bridge integration
  full Axiom (persona + coaching + tool calls)
  duels + team challenges · betting (points-only)
  Drive sync · export PDF notebooks · priority matching

TEAM (€30/seat/mo — B2B)
  group accountability workspace
  admin dashboard (progress, engagement, leaderboard)
  custom domains · Axiom coach per team
  Slack/Calendar integration
  target: startups, sports teams, corporate wellness
```

---

## 3. Aura Pro

```
FREE
  AR launcher · keyless AI · manual check-ins
  basic wiki (100 pages) · 3 home widgets

PRO (€6/mo or bundled with Praxis Pro)
  passive logging (audio + vision → wiki, unlimited)
  background compression (MemoryCompressor)
  Praxis bridge + GoalHudStrip + auto check-ins
  Drive sync · unlimited widgets · voice commands
  priority Tier 1 AI providers (no keyless fallback delay)
```

---

## 4. Accountability Bets (Key Differentiator)

Real stakes change behavior. Ariely, Wertenbroch et al. on commitment devices: financial stakes increase follow-through ~40% versus no stakes.

```
User commits:
  goal:   "Run 3× per week for 4 weeks"
  stake:  €50
  evidence: partner witness + Aura auto check-in (GPS + motion + vision)

Resolution:
  success → €50 returned + PP bonus
  failure → €50 → partner / charitable pool
  Praxis fee: 5–10% of resolved stake

Legal: peer-to-peer commitment device, not gambling.
       EU GDPR compliance required. Escrow via Stripe.
```

This is the product's sharpest edge. Self-commitment literature shows that *costless* declarations have minimal behavioral effect. The bet makes the intention costly — and therefore credible.

---

## 5. Axiom B2B API

```
Use case:
  Fitness / wellness / HR apps want goal-aware AI coaching
  without building ontology, matching, or behavioral models.

Endpoint:
  POST /api/axiom/b2b/coach
  input:  { userId, goals[], recentActivity[], question }
  output: { coaching_text, suggested_actions[], domain_insights[] }

Pricing: €0.02/active user/month (per-seat via Stripe)

Moat:
  Rachmaninov ontology + community wiki insights
  → coaching quality improves with scale
  → competitors cannot replicate without the behavioral corpus
```

---

## 6. Revenue Projections

```
1,000 users:
  600 Free → €0
  300 Praxis Pro → €3,600/mo
  100 Aura Pro  → €600/mo
  bet fees (avg €2/user/mo) → €400/mo
  Total: ~€4,600/mo

10,000 users:
  6,000 Free · 3,000 Pro · 1,000 Aura
  Pro: €36,000 · Aura: €6,000 · bets: €4,000
  Total: ~€46,000/mo

100,000 users + B2B API:
  Pro revenue: ~€360k/mo
  B2B API (50 partners × 1k MAU): €1k/mo
  Coach marketplace (10 coaches × €3k × 25%): €7.5k/mo
  Total: ~€400k/mo
```

---

## 7. Development Roadmap

### Tier 0 — Foundation (complete)

```
[✓] Aura: full Android launcher (CATEGORY_HOME)
[✓] Aura: AR HUD with VisionAnalyzer + ObjectDetector
[✓] Aura: AuraSession + AgentLoop + 35+ provider fallback
[✓] Aura: WikiStore + WikiIngester + WikiContext (second brain)
[✓] Aura: MemoryCompressor (WorkManager, background distillation)
[✓] Aura: PraxisOntology (14 domains, ayu schema, contextHints)
[✓] Aura: VISUAL_HINTS map (130+ vision/audio keywords → domains)
[✓] Aura: PraxisEventBridge (vision/audio → auto check-in)
[✓] Aura: GoalHudStrip (context-ranked goal in HUD)
[✓] Aura: RachmaninoffSync (live ontology fetch, ETag, bundled fallback)
[✓] Aura: DebounceLedger persistent (SharedPreferences)
[✓] Aura: Google Drive sync (6h WorkManager, drive.appdata)
[✓] Aura: AppSidebar + Nova Launcher import
[✓] Aura: WidgetStrip (prompt-generated live widgets)
[✓] Aura: Lattice voice dispatch + LatticeScreen
[✓] Aura: VoiceCommandRouter (dispatch/check-in/capture/aurora)
[✓] Praxis: /api/rachmaninov/ontology (live sync endpoint)
[✓] Praxis: Axiom agent (tool calls, persona, coaching, dispatch_job)
[✓] Praxis: duels + team challenges + accountability bets
[✓] Praxis: MatchingEngineService (Jaccard domain overlap)
[✓] Praxis: MCP server (Claude Desktop integration)
[✓] Praxis: PAT system (named agent tokens, actor attribution)
[✓] Praxis: Lattice (device registry, job queue, device-key auth)
[✓] Praxis: LatticePage (React UI), demo_device.py (Python agent)
```

### Tier 1 — Core gaps (next)

```
[ ] Rachmaninov: versioned service (version bump → cache bust)
[ ] Praxis: bidirectional sync (goal completed → Aura notification)
[ ] Praxis: mobile-first PWA (installable, push notifications)
[ ] Praxis: team challenge UI (create, join, live progress board)
[ ] Praxis: accountability bet UI (Stripe connect + real € flow)
[ ] Aura: consent layer (per-goal opt-in before any data leaves device)
[ ] Aura: onboarding (6-question interview, goal import from Praxis)
[ ] Community wiki schema + anonymization pipeline
```

### Tier 2 — Growth mechanics

```
[ ] MatchingEngine: semantic embeddings (replace Jaccard)
[ ] Axiom: weekly narrative PDF auto-generated and delivered
[ ] Axiom: habit failure detection + intervention probes
[ ] Praxis: coach marketplace (verified + commission)
[ ] Praxis: seasonal events + global domain challenges
[ ] Aura: Gemma on-device (local inference for sensitive tasks)
[ ] Aura: smart glasses bridge (Omi wearable integration)
[ ] Rachmaninov: task sensitivity classifier (local vs cloud routing)
```

### Tier 3 — Moat

```
[ ] Rachmaninov: full anonymization pipeline
[ ] Community wiki: live, Axiom-maintained, searchable
[ ] Rachmaninov: behavioral training loop (community → model)
[ ] Praxis: federated goal graph (cross-user, anonymized)
[ ] Axiom B2B API (per-MAU licensing to third-party apps)
```

---

## 8. The Competitive Moat

The system's defensibility does not come from any single feature. It comes from **the behavioral corpus**.

As users check in, grade each other, fail and recover, the community wiki accumulates anonymized `will → action → effect` flows. Axiom reads these flows when coaching. Over time, Axiom's coaching becomes calibrated to what actually works — not what productivity literature says works, but what worked for people with a similar psychological profile in a similar domain with similar stagnation patterns.

This corpus cannot be purchased. It cannot be copied. It can only be grown — one check-in at a time.

The ontology (Rachmaninov) is the schema. The corpus is the value. The bet mechanism ensures the data is honest — stakes make self-deception expensive.
