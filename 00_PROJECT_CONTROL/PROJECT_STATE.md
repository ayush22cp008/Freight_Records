# PROJECT_STATE.md — Project State

## Historical Project Nodes

- ✅ Node 0 — Problem research & selection (LOCKED)
- ✅ Node 1 — Solution design + stack (LOCKED)
- ✅ Node 2 — Build plan — REVISED: 4-day plan superseded by 25-day hackathon scope (Aug 21–Sep 15)
- ✅ Node 2.5 — Core logic test (LOCKED)
  - ✅ Test 1 — GPS
  - ✅ Test 2 — Camera → Storage upload
  - ✅ Test 3 — Immutable insert-only RLS (via service-role route)
- ✅ Historical Node 3 — Original Core MVP build execution completed
  - ✅ Driver-only login + pre-seeded trip
  - ✅ Trip Hub / workflow state foundation
  - ✅ Arrival event
  - ✅ Check-in event
  - ✅ Departure event
  - ✅ Chronological Timeline
  - ✅ AI Evidence Summary via Groq
  - ✅ AI Summary truncation fix + browser verification

## Active Execution Roadmap — 7 Nodes

The historical Node map above is preserved. The **active remaining hackathon execution** now follows the 7-Node roadmap established by the Chat8 product/security rework and finalized in `00_PROJECT_CONTROL/ROADMAP.md`.

| Active Node | Objective | Baseline | Status |
|---|---|---:|---|
| **Node 1** | Product + Authorization Rework | 2 days | 🔵 **CURRENT / NEXT** |
| **Node 2** | Authentication + Identity | 3 days | 🔵 BLOCKED BY NODE 1 |
| **Node 3** | Company Trip Creation + Publishing | 3 days | 🔵 FUTURE |
| **Node 4** | Driver Marketplace + Atomic Claim | 3 days | 🔵 FUTURE |
| **Node 5** | Whole Delivery Tracking | 5 days | 🔵 FUTURE |
| **Node 6** | Security + Evidence | 3 days | 🔵 FUTURE |
| **Node 7** | AI + Final Integration + Demo | 3 days | 🔵 FUTURE |

**Baseline:** 22 planned days. Durations are estimates; actual duration must be recorded after each Node.

## Current Gate — Node 1

### Node 1 — Product + Authorization Rework

**Purpose:** lock the product and authorization model before authentication implementation resumes.

Node 1 must explicitly resolve:

- Company / Driver roles
- Auth user → Company/Driver identity mapping
- Contextual creator/sending-company and receiving-company relationships
- Minimum trip relationships and required trip/commercial data
- Trip state machine
- Driver eligibility
- Atomic first-valid acceptance rule
- Complete authorization matrix
- Server-side IDOR/API protection rules
- Authentication requirements derived from the final model

### Authorization responsibility split

```text
Node 1 → design + lock authorization matrix / IDOR rules
Node 6 → implement + verify authorization / IDOR rules
```

Node 2 cannot begin until the Node 1 acceptance criteria are explicitly verified.

## Security State

```text
RLS investigation                    → CLOSED / VERIFIED
Rate-limiting architecture            → DECIDED
IDOR / API authorization              → OPEN
Authentication implementation          → PAUSED
```

Do not reopen the RLS investigation without new contradictory evidence.

## Subnode Model

A Subnode is a controlled unit of **significant unexpected work inside a parent Node**.

```text
Small bug
→ fix inside current Node

Significant unexpected issue
→ create Subnode under current Node

Major blocker / architecture change
→ stop and reassess roadmap
```

Atomic first-valid acceptance is already a known Core requirement of Active Node 4 and is not treated as an unexpected Subnode.

**Escalation rule:** if a Node accumulates **3+ Subnodes**, perform an explicit roadmap reassessment.

## Merged Stretch Strategy

The historical stretch features remain preserved but are now attached to the active Node where they naturally belong:

- Node 3 — company dashboard/role capabilities required by the locked trip-creation model.
- Node 5 — derived dwell-time, mandatory Check-in photo, repeatable Add Evidence, geofence proximity badge; all conditional and subordinate to the core delivery lifecycle.
- Node 7 — public shareable evidence link, AI inconsistency detection, and optional AI-depth enhancements; video capture is lowest priority and may be cut first.

Core Node acceptance criteria always take priority over stretch work.

## State Transition

```text
Historical Core MVP
        ↓
IMPLEMENTED / VERIFIED
        ↓
Chat8 product + security rework
        ↓
ACTIVE 7-NODE ROADMAP
        ↓
NODE 1 — Product + Authorization Rework
        ↓
NODE 2 — Authentication + Identity
        ↓
NODE 3 — Company Trip Creation + Publishing
        ↓
NODE 4 — Driver Marketplace + Atomic Claim
        ↓
NODE 5 — Whole Delivery Tracking
        ↓
NODE 6 — Security + Evidence
        ↓
NODE 7 — AI + Final Integration + Demo
```

## Current Project State

```text
Historical Core MVP       → COMPLETE / VERIFIED
Active roadmap             → 7 Nodes
Current active Node        → Node 1
Node 1 state               → NEXT / GATE
Authentication             → PAUSED
RLS                        → CLOSED / VERIFIED
Rate limiting architecture → DECIDED
IDOR/API authorization     → OPEN
```

## Completion Rule

An active Node is `COMPLETE` only after:

- Required tasks are complete.
- Acceptance criteria are satisfied.
- Required investigations are resolved or explicitly deferred.
- Required security checks are complete.
- Build/test evidence is recorded.
- Ayush manual verification is complete.
- Implementation report is recorded.

Time passing alone does not complete a Node.

## Record Routing

Implementation handoffs:

```text
03_IMPLEMENTATION/prompts/
```

Antigravity implementation reports:

```text
03_IMPLEMENTATION/implementation_reports/
```

Investigations:

```text
05_DEBUGGING/investigations/
```
