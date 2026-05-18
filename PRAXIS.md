# PRAXIS — The Arena

> *"Praxis, in Aristotle, is action undertaken for its own sake — not as means to an end,*  
> *but as the expression of a disposition toward the good."*

---

## 1. The Name

Aristotle distinguishes three forms of purposive activity:

- **Theoria** — contemplation; knowing for the sake of knowing
- **Poiesis** — making; action that produces an object external to itself
- **Praxis** — doing; action that is its own end, whose excellence lies in its exercise

This system is named for the third. Not because it produces outcomes (though it does), but because the commitment to *show up* — to act, to log, to be held accountable — is itself the practice being cultivated. The outcome is evidence of the practice, not its purpose.

---

## 2. The Core Problem: Accountability Without Surveillance

Existing accountability tools fail in one of two ways:

1. **Too weak** — rely entirely on self-report. Progress bars move because the user wills them to. No mechanism distinguishes honest effort from optimistic logging.

2. **Too strong** — impose external monitoring (location tracking, keystroke logging, screen recording). Treat the user as subject under observation, not agent with sovereignty over their own data.

Praxis proposes a third path: **peer validation**. Effort is graded by humans who have their own goals at stake — collaborators, partners, coaches — not by algorithm and not by panopticon.

The community grades effort. The individual retains sovereignty over what is shared.

---

## 3. Architecture

```
CLIENT (React 19 + MUI v7 + Vite + TypeScript)
  Supabase JWT authentication (anon key in browser)
  Deployed: Vercel
         │ HTTPS
         ▼
BACKEND (Express 5 + TypeScript, Node ≥ 20)
  Supabase service-role key (server-only, bypasses RLS)
  ~50 controllers · rate limiting · Sentry observability
  Deployed: Railway (auto-deploy on push to main)
         │
  ┌──────┴──────────────────────────────┐
  ▼              ▼                      ▼
Supabase        Stripe               Gemini API
Postgres        subscriptions        Axiom agent
+ RLS           bets/payments        coaching
```

---

## 4. The Goal System

### GoalTree

One per user. A directed acyclic graph of GoalNodes with prerequisite and parent-child relationships.

### GoalNode

```
id                UUID
domain            one of 14 domains (Rachmaninov ontology)
name              user-defined string
weight  W_j       0.1–2.0, auto-adjusts via peer feedback
progress          0.0–1.0
parentId          optional (goal decomposition)
prerequisiteIds   optional (sequenced goals)
customDetails     freeform (units, target, notes)
```

### Weight Adjustment

Goal weight `W_j` self-calibrates via peer outcome grades:

```
SUCCEEDED        → W × 0.8    (getting easier; reduce focus)
TRIED_BUT_FAILED → W × 0.95   (marginal reduction)
MEDIOCRE         → W × 1.1    (needs more attention)
DISTRACTED       → W × 1.2    (needs more focus)
```

A consistently high-outcome goal shrinks in weight — the user is succeeding; less push needed. A chronically stalled goal grows — it needs more attention and heavier investment. Weight modulates how much a check-in moves overall progress.

### Goal Hierarchy Example

```
Career & Craft (domain)
├── "Ship Praxis v2" (root, progress=45%)
│   ├── "Implement matching engine" (completed ✓)
│   ├── "Build betting system" (in-progress)
│   └── "Axiom coaching v2" (blocked — prereq above)
└── "Land consulting client" (root, progress=20%)
```

### Validity Layers

```
PERSONAL    self-assessed only; no external validation required
VALIDATED   ≥1 collaborator has graded ≥3 check-ins
COMMUNITY   anonymized flow contributed to community wiki
```

A goal cannot reach VALIDATED by self-report alone. This operationalizes Popper without invoking him: **a goal that has never produced a falsifiable check-in is not a goal — it is an intention.**

---

## 5. Check-ins and Grading

A **check-in** is the atomic unit of Praxis:

```
goalId · progress_delta · grade · notes · actor_type · agent_name · timestamp
```

`actor_type` ∈ `user | agent | axiom | aura`

This field records *who made the check-in*. The ledger is transparent. The user always knows what touched their goals.

### Grade Rationale Schema

```
COMPLETENESS   Did they do what they said they would do?
EFFORT         Given circumstances, how hard was the attempt?
OUTCOME        Did the action produce the intended result?
CONSISTENCY    Is this part of a sustained pattern?
```

Grades are assigned by collaborators with their own goals at stake — not neutral observers, but stakeholders. The grader's credibility is implicated in their assessment.

---

## 6. Social Mechanics

### Sparring (Accountability Partners)

```
User A ──request──► User B
                    (7-day expiry)
User B ──accept──► SparringPartner created
                   node_id_a ↔ node_id_b
                  → mutual progress notifications
                  → partner check-in prompts
```

Two people bind a shared goal node. Each becomes witness to the other's effort. Not competition — mutual commitment.

### Duels

```
User A challenges User B (or random match)
  title · category · stakePP · deadlineDays · goalNodeId
         │
         ▼
DuelResolutionCron (nightly)
  both users check-in progress
  higher relative progress wins
  winner receives loser's stakePP
  Stripe handles real-money variant
```

Duels convert private goals into public contests. Skin in the game — PP or real currency — makes commitment costly and therefore credible.

### Team Challenges

```
Team (up to N members)
  title · domain · stakePP/member · deadline
  ├── member 1 check-ins
  ├── member 2 check-ins
  └── member N check-ins
  → aggregate progress vs deadline
  → completion: all members gain XP + badge
  → failure: stakePP redistributed to achievers
```

Collective commitment device. A team that fails together redistributes stakes to those who delivered.

### Accountability Bets (Stripe)

Real stakes change behavior. Ariely et al. on commitment devices: financial stakes increase follow-through ~40% vs. no stakes.

```
User commits goal + deadline + real €
  opponentType: 'self' | 'duel'
  stakeAmount: € (Stripe PaymentIntent held in escrow)
       │
  on deadline:
  ├── success → stake returned + PP bonus
  └── failure → stake → partner / charitable pool
               (5–10% Praxis fee)
```

This is not gamification. It is **skin in the game** — the mechanism by which a preference becomes a commitment.

### Matching Engine

```
MatchingEngineService.calculateCompatibilityScore(treeA, treeB):
  domainScore    = Jaccard(domainsA, domainsB)          × 0.60
  progressScore  = 1 - |avgProgressA - avgProgressB|    × 0.25
  structureScore = avg(depthSimilarity, countSimilarity) × 0.15
  → compatibility ∈ [0, 1]
```

Suggests accountability partners with overlapping domains and similar momentum. SAB: Similarity (domains), Affinity (progress parity), Behavior (check-in consistency). The goal is not to match by personality — it is to match by where you are in the same kind of struggle.

---

## 7. Axiom — The AI Agent

Axiom is Praxis's agentic AI. Runs nightly on all active users; available on-demand in conversation.

### Nightly Scan (AxiomUnifiedScanService)

```
User context (goals, check-ins, notebook, wiki, trackers)
       │
       ▼
AxiomProgressEstimationService  → progress % from trackers
AxiomDailySummaryService        → distill day's check-ins
AxiomPersonaService             → build/update psychological profile
AxiomLearningService            → detect: which methods work, what causes failure
```

### On-Demand (Conversational)

```
AxiomAgentController
├── AICoachingService (Gemini + persona context)
├── AxiomWikiSearchService (semantic community wiki)
├── AxiomMultimodalService (image/audio input)
└── Tool calls (agentic):
      create_bet · create_duel · create_team_challenge
      log_tracker · create_goal · update_goal_progress
      create_notebook_entry · push_notification · suggest_match
      dispatch_job  ← Lattice (physical device dispatch)
```

All tool calls pass through `AxiomActionRouter` (Zod schema validation) → `AxiomActionExecutor` → audit log.

### The dispatch_job Tool

Axiom can submit physical jobs to any device in the Lattice network:

```
User: "Axiom, I finished the CAD model. Print the bracket."
Axiom: → dispatch_job(deviceSlug="workshop-3d-printer",
                       jobType="print_file",
                       payload={file: "bracket_v2.stl"})
      → job written to device_jobs table (submitted_by='axiom')
      → device agent polls, downloads, executes
      → status callback → Axiom confirms to user
```

A goal about building something can, via Axiom, directly trigger the physical production step.

---

## 8. The Lattice — Physical World Network

Any machine — CNC mill, 3D printer, computer, smart home hub, camera, server — can be registered as a Lattice node.

### Device Registration

```
POST /api/lattice/devices/register
  Authorization: Bearer <user_jwt>
  { name, slug, type, capabilities[] }

→ { id, apiKey: "dk_..." }
```

The device receives a `dk_*` key (not JWT). Used for all device-to-server communication.

### Device Types

```
3dprinter · cnc_mill · computer · smart_home · wearable · camera · server · custom
```

### Job Lifecycle

```
submitted_by: user | axiom | aura | mcp

PENDING → RUNNING → DONE
                 ↘ FAILED
         ← CANCELLED (if still pending)

fields: type · payload JSON · progress_pct (0–100)
        result JSON · error_msg · started_at · completed_at
```

`submitted_by` records provenance. When Axiom dispatches, `submitted_by='axiom'`. When Aura voice-dispatches, `submitted_by='aura'`. The physical job ledger is as transparent as the goal ledger.

### Device Agent Protocol (demo_device.py)

```python
register()        → receive dk_* key
heartbeat_loop()  → POST /lattice/devices/heartbeat (30s)
poll_loop()       → POST /lattice/jobs/poll (10s)
execute_job(job)  → simulate/execute, PATCH status per step
SIGINT            → mark device offline
```

---

## 9. Agent System (External Integrations)

```
Agent connects:
  GET /api/agent/:slug/connect → one-time connect code → user authorizes in-app

Agent calls API:
  x-api-key: pk_live_...
  → authenticateToken resolves to userId + agentLabel
  → actor_type='agent', agent_name='aura-field' written on every checkin

Personal Agent Token (PAT):
  POST /api/agents/keys/personal { agentLabel }
  → pk_live_...

MCP Server (praxis-mcp-server/):
  Tools: get_goals · post_checkin · update_progress
         search_wiki · get_daily_summary
  → Claude Desktop, VS Code Copilot, any MCP client
```

**Aura is an agent.** It authenticates with a PAT, posts check-ins, patches goal progress, reads active goals. Every touch attributed to `actor_type='aura'` in the ledger.

---

## 10. Community Wiki

```
Personal wiki (Aura, local, private)
       │ user opt-in per goal
       │ anonymization (names/places/IDs stripped)
       ▼
Goal wiki (Praxis DB, per-user, private)
       │ AxiomWikiWriterService (nightly)
       │ distill to will→action→effect flow
       ▼
Community wiki (global, anonymized, AI-edited)
       │
       ├── AxiomWikiSearchService: reads for coaching context
       ├── Axiom: writes back compressed insights
       └── Future: training corpus for Rachmaninov model
```

Not a user-facing feature. An inference substrate. Axiom reads it silently to answer: "what works for people in domain X trying to do Y?" No individual record is ever accessible. The community wiki contains only anonymized `will → action → effect` flows.

---

## 11. Psychological Framework

Axiom uses psychological models as **context, not diagnosis**:

- **Big Five** — openness, conscientiousness, extraversion, agreeableness, neuroticism → coaching tone adjustment
- **Jungian archetypes** — Hero, Sage, Explorer, Creator, Caregiver... → goal framing language
- **DSM-5 (contextual)** — identify self-sabotage loops; not diagnose

Applications:
- Frame advice in the user's own symbolic language
- Identify recurrent stagnation patterns
- Suggest alternative methods when current approach fails
- Adjust Axiom "harshness" setting (gentle ↔ direct)

Profile builds passively from check-in language, failure patterns, and response to coaching styles. **Never exposed. Never shared.** Used only per-user, on-device and per-user server context.

---

## 12. Notebook (Personal Log)

Structured journal tightly coupled to goals:

```
Entry types: note | journal | reflection | goal_progress
Tags: domain + goal
Export: PDF via NarrativePdfRenderer / NotebookPdfRenderer
```

Axiom reads notebook context before every coaching session. The journal is not decoration — it is the qualitative layer that explains the quantitative check-in record.

---

## 13. Gamification

```
PP (Praxis Points)  currency for bets and duels
XP                  experience points, drives level rank
Streak              consecutive daily check-in days
Quests              daily/weekly challenges per domain
Badges              unlocked by milestones
Leaderboard         opt-in, domain-filtered
```

Gamification is not the product. It is the social coordination layer that makes check-ins feel worth doing on days when abstract goals feel remote.

---

## 14. Technical Stack

| Component | Technology |
|---|---|
| Frontend | React 19, MUI v7, Vite, TypeScript |
| Backend | Express 5, TypeScript, Node ≥ 20 |
| Database | Supabase (PostgreSQL + RLS) |
| Auth | Supabase JWT + custom PAT system |
| AI agent | Axiom (Gemini 2.5 Flash, tool-calling) |
| Payments | Stripe (subscriptions, bets escrow) |
| Deploy | Railway (backend), Vercel (frontend) |
| Observability | Sentry |
| Physical network | Lattice (device registry, job queue, `dk_*` auth) |
| MCP server | Claude Desktop / VS Code integration |

Source: `https://github.com/ilPez00/praxis-backend`
