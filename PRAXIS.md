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

1. **Too weak** — they rely entirely on self-report, with no mechanism to distinguish honest effort from optimistic logging. Progress bars move because the user wills them to.

2. **Too strong** — they impose external monitoring (location tracking, keystroke logging, screen recording) in exchange for accountability. They treat the user as a subject under observation, not an agent with sovereignty over their own data.

Praxis proposes a third path: **peer validation**. Effort is graded by humans who have their own goals at stake — collaborators, partners, coaches — not by an algorithm, and not by a panopticon.

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

Every user has one GoalTree. It is a directed acyclic graph of GoalNodes.

### GoalNode

```
id                UUID
domain            one of 14 domains (from Rachmaninov ontology)
name              user-defined string
weight  W_j       0.1–2.0, auto-adjusts via peer feedback
progress          0.0–1.0
parentId          optional (for goal decomposition)
prerequisiteIds   optional (for sequenced goals)
customDetails     freeform object (units, target, notes)
```

### Weight Adjustment

Goal weight `W_j` self-calibrates via peer feedback grades:

```
W_j(t+1) = W_j(t) × (1 + α × (grade - 0.5))

grade ∈ [0, 1]   (peer-assigned)
α = 0.1          (learning rate)
```

A consistently high-graded goal (grade > 0.5) grows in weight. A low-graded goal shrinks. Weight affects how much a check-in moves progress. The community's assessment of effort quality flows back into the goal's influence on the user's aggregate progress.

---

## 5. Check-ins and Grading

A **check-in** is the atomic unit of Praxis. It records:
```
goalId · progress_delta · grade · notes · actor_type · agent_name · timestamp
```

`actor_type` is one of: `user | agent | axiom | aura`

This field records *who made the check-in* — the user themselves, Axiom (the AI coach), an Aura sensor event, or an external agent acting on their behalf. The ledger is transparent. The user always knows what touched their goals.

### Grading Schema

```
COMPLETENESS   Did they do what they said they would do?
EFFORT         Given circumstances, how hard was the attempt?
OUTCOME        Did the action produce the intended result?
CONSISTENCY    Is this part of a sustained pattern?
```

Grades are assigned by collaborators who have their own goals — not neutral observers, but stakeholders. This creates epistemic accountability: the grader's own credibility is at stake in their assessment.

---

## 6. Validity Layers

Praxis distinguishes between goals at different maturity levels:

```
PERSONAL    → self-assessed only; no external validation required
VALIDATED   → at least one collaborator has graded ≥3 check-ins
COMMUNITY   → anonymized flow contributed to community wiki
```

A goal cannot be declared VALIDATED by self-report alone. This is not a technical restriction — it is a philosophical one. Self-belief that has never been tested by external reality is not knowledge. It is hypothesis.

The system operationalizes Karl Popper without invoking him: **a goal that has never produced a falsifiable check-in is not a goal. It is an intention.**

---

## 7. Axiom — The AI Coach

Axiom is the resident AI agent in Praxis. It does not replace the human in the accountability loop — it enhances it.

### What Axiom Does

```
Daily brief:
  reads user's active goals, recent check-ins, grades
  reads Rachmaninov ontology (which domain, which score axis)
  reads ACT stats (consistency, trend, stagnation)
  reads community wiki (anonymized: what worked for archetype X in domain Y)
  → proposes today's actions via Rachmaninov.proposeDay()
  → one specific, measurable action per goal
  → delivered as voice/text brief through Aura

Agent tools available to Axiom:
  create_checkin(goalId, progressDelta, grade, notes)
  update_goal(goalId, fields)
  dispatch_job(deviceSlug, jobType, payload)  ← Lattice integration
  read_activity()
  list_goals()
```

### The dispatch_job Tool

Axiom can submit physical jobs to any device in the Lattice network:

```
User: "Axiom, I finished the CAD model. Print the bracket."
Axiom: → dispatch_job(deviceSlug="workshop-3d-printer", jobType="print_file",
                       payload={file: "bracket_v2.stl"})
      → creates job in device_jobs table
      → device agent polls, downloads, executes
      → status callback → Axiom confirms to user
```

This closes the loop: a goal about building something can, via Axiom, directly trigger the physical production step.

---

## 8. The Lattice — Physical World Network

Lattice is Praxis's physical device layer. Any machine — CNC mill, 3D printer, computer, smart home hub, camera, server — can be registered as a Lattice node.

### Device Registration

```
POST /api/lattice/devices/register
  Authorization: Bearer <user_jwt>
  { name, slug, type, capabilities[] }

Response: { id, apiKey: "dk_..." }
```

The device receives a `dk_*` key (not a JWT). This key is used for all device-to-server communication — heartbeats, job polls, status updates.

### Device Types

```
3dprinter · cnc_mill · computer · smart_home
wearable  · camera   · server   · custom
```

### Job Lifecycle

```
PENDING → RUNNING → DONE
                 ↘ FAILED
         ← CANCELLED (if still pending)
```

```
device_jobs:
  id · device_id · goal_id? · type · payload JSON
  status · submitted_by (user|axiom|aura|mcp)
  progress_pct (0–100) · result JSON · error_msg
  created_at · started_at · completed_at
```

`submitted_by` records provenance. When Axiom dispatches a job, `submitted_by='axiom'` is written. When Aura voice-dispatches, `submitted_by='aura'`. The physical job ledger is as transparent as the goal ledger.

### Device Agent Protocol (demo_device.py)

Any device runs a polling agent:
```python
register() → receive dk_* key
heartbeat_loop() → POST /lattice/devices/heartbeat (30s interval)
poll_loop() → POST /lattice/jobs/poll (10s interval)
execute_job(job) → simulate/execute work, PATCH status every step
SIGINT → mark device offline
```

---

## 9. Personal Agent Tokens (PAT)

Praxis supports named agents acting on behalf of a user — not just Axiom, but any external system (Aura, a custom script, an MCP client):

```
POST /api/agents/keys/personal
  Authorization: Bearer <user_jwt>
  { agentLabel: "aura-field" }

Response: { key: "pk_live_..." }
```

The PAT is used like a JWT but carries `actor_type='agent'` and `agent_name='aura-field'`. Every check-in or job dispatch using this token is attributed to the named agent in the activity ledger.

This enables **multi-agent accountability**: a user's goals can be touched by Axiom, by Aura, by external services — and every touch is recorded, attributed, and auditable by the user.

---

## 10. The Community Dimension

The full Praxis vision includes a community layer that Rachmaninov enables:

```
Individual goal wiki
       │ anonymization (names, locations, biometrics stripped)
       ▼
Community wiki (domain X, archetype Y: what actions produced grade ≥0.8?)
       │ Axiom reads
       ▼
Compressed insight: "Users matching archetype Y in domain X
  who do 5×5 strength work 3x/week outperform daily cardio by 0.3σ"
       │ injected into Axiom coaching prompt
       ▼
Better daily proposals for all users in that domain
```

No individual record is ever accessible in the community layer. The community wiki contains only anonymized `will → action → effect` flows — causal patterns with all identifying information removed.

The epistemological claim: **the community's aggregate experience is a commons. Individual experience is private property.** Rachmaninov is the property boundary.

---

## 11. Bets and Commitments

Praxis allows users to place financial stakes on goals:

- **Bet with self**: "If I don't complete X by date Y, I donate €Z to cause C"
- **Bet with partner**: "We each commit €Z. The one with higher graded progress wins"
- **Public commitment**: goal + stake publicly declared; community grades outcome

Stakes are held via Stripe, disbursed automatically on goal resolution.

This is not gamification. It is **skin in the game** — the mechanism by which commitment becomes costly and therefore credible. A costless commitment is not a commitment. It is a preference.

---

## 12. Technical Stack

| Component | Technology |
|---|---|
| Frontend | React 19, MUI v7, Vite, TypeScript |
| Backend | Express 5, TypeScript, Node ≥ 20 |
| Database | Supabase (PostgreSQL + RLS) |
| Auth | Supabase JWT + custom PAT system |
| AI agent | Axiom (Gemini 2.5 Flash, tool-calling) |
| Payments | Stripe (subscriptions, bets) |
| Deploy | Railway (backend), Vercel (frontend) |
| Observability | Sentry |
| Physical network | Lattice (device registry, job queue, device-key auth) |

Source: `https://github.com/ilPez00/praxis-backend`
