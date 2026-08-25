# PROJECT_STATE.md — Project State

## Historical Project Nodes

- ✅ Node 0 — Problem research & selection (LOCKED)
- ✅ Historical Node 1 — Solution design + stack (LOCKED)
- ✅ Historical Node 2 — Build plan — REVISED: 4-day plan superseded by 25-day hackathon scope (Aug 21–Sep 15)
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

The historical Node map above is preserved. The active remaining hackathon execution follows the 7-Node roadmap established by the Chat8 product/security rework and finalized by the later roadmap review.

| Active Node | Objective | Baseline | Status |
|---|---|---:|---|
| **Node 1** | Product + Authorization Rework | 2 days | 🔒 **COMPLETE / LOCKED** |
| **Node 2** | Authentication + Identity | 3 days | 🔵 **ACTIVE DESIGN / NOT LOCKED** |
| **Node 3** | Company Trip Creation + Publishing | 3 days | 🔵 FUTURE |
| **Node 4** | Driver Marketplace + Atomic Claim | 3 days | 🔵 FUTURE |
| **Node 5** | Whole Delivery Tracking | 5 days | 🔵 FUTURE |
| **Node 6** | Security + Evidence | 3 days | 🔵 FUTURE |
| **Node 7** | AI + Final Integration + Demo | 3 days | 🔵 FUTURE |

**Baseline:** 22 planned days. Durations are estimates; actual duration must be recorded after each Node.

## Node 1 — Product + Authorization Rework

Node 1 is formally locked by:

`01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md`

That record states:

```text
Node 1 Product Model                 → LOCKED
Trip Lifecycle                       → LOCKED
Delivery State Machine               → LOCKED
Issues / Emergency Model             → LOCKED
Evidence Model                       → LOCKED
Authorization Matrix                 → LOCKED
IDOR / API Authorization             → LOCKED
Concurrency Rules                    → LOCKED
Claude Independent Review            → APPROVED
NODE 1                               → COMPLETE
```

The locked identity invariant is:

```text
1 Auth User ↔ exactly 1 application identity
1 Auth User ↔ exactly 1 application role
Role = Company OR Driver
```

The final lock also states that authentication implementation remains paused until the Node 2 authentication contract is designed and independently reviewed.

## Node 2 — Authentication + Identity

**Current state:**

```text
Node 2 broad investigation          → COMPLETE
Remaining auth evidence             → COMPLETE
Signup/onboarding investigation     → COMPLETE
Claude contract review              → COMPLETE
Node 2 contract                     → DRAFT / NOT LOCKED
Authentication implementation      → PAUSED
```

Current contract draft:

`02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`

Claude's independent review concluded that the contract was **NOT READY FOR LOCK** because several load-bearing decisions remain unresolved.

### Remaining Node 2 decisions

1. Signup / onboarding consistency
2. Email-confirmation policy
3. Session lifecycle / refresh
4. One-user → one-identity enforcement mechanism
5. Authentication rate-limiting policy
6. RLS / service-role boundary for Node 2
7. Final acceptance-test matrix

### Signup / onboarding evidence checkpoint

The targeted investigation established that current signup creates the Supabase Auth User and application identity through separate operations rather than one database transaction.

A verified failure state is:

```text
Auth User EXISTS
Application identity MISSING
```

The investigation also identifies a reverse orphan risk through the current `ON DELETE SET NULL` relationship.

This is evidence for a design decision; it is not an implementation authorization.

## Security State

```text
RLS investigation                    → CLOSED / VERIFIED
Rate-limiting architecture            → DECIDED
IDOR / API authorization              → LOCKED BY NODE 1
Authentication implementation          → PAUSED
```

Do not reopen RLS without new contradictory evidence.

## Active State Transition

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
NODE 1 FINAL LOCK
        ↓
NODE 2 — Authentication + Identity
        ↓
Contract design / investigation / decision
```

## Current Project State

```text
Historical Core MVP       → COMPLETE / VERIFIED
Active roadmap             → 7 Nodes
Node 1                     → COMPLETE / LOCKED
Node 2                     → ACTIVE DESIGN / NOT LOCKED
Authentication             → PAUSED
RLS                        → CLOSED / VERIFIED
Rate limiting architecture → DECIDED
IDOR/API authorization     → LOCKED BY NODE 1
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

## Checkpoint

Chat11 checkpoint:

`00_PROJECT_CONTROL/CHECKPOINTS/Chat11_Day4_Node2_Checkpoint.md`

## Record Routing

ChatGPT ↔ Antigravity bridge:

```text
GitHub Records repository
```

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

Architecture records:

```text
02_ARCHITECTURE/
```

Project-control records:

```text
00_PROJECT_CONTROL/
```
