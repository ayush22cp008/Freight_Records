# CURRENT_STATUS.md

**Last updated:** Aug 25, 2026

## Where we are

**Historical Core MVP — IMPLEMENTED / VERIFIED.**

The original fixed single-facility, 3-event Core MVP remains completed and preserved:

- Login → Trip Hub → Arrival → Check-in → Departure → Timeline → AI Evidence Summary.
- Trip Hub remains the workflow state source of truth for the original Core MVP.
- Arrival, Check-in, and Departure are immutable evidence events with GPS + server timestamp.
- Arrival and Departure require photo evidence; Check-in remains optional-photo under the original Core MVP scope.
- Timeline displays recorded events chronologically with evidence.
- AI Evidence Summary interprets deterministic Arrival + Check-in + Departure evidence.
- AI summary truncation fix was implemented and browser-verified.
- `npm run build` passes.

The Core MVP is **not being discarded**. It is the verified foundation being extended into the broader product model defined by the active 7-Node roadmap.

## Current Product Direction

The active product model remains:

```text
Company creates / publishes trip
        ↓
Eligible drivers see opportunity
        ↓
Driver evaluates trip economics/details
        ↓
Driver accepts
        ↓
Atomic first-valid acceptance wins
        ↓
Trip locks to winning driver
        ↓
Pickup
        ↓
Arrival / Check-in / Load / Depart
        ↓
In transit
        ↓
Destination / receiving company
        ↓
Arrival / Check-in / Unload / Delivery confirmation
        ↓
Delivery completed
        ↓
Immutable evidence timeline
        ↓
AI evidence-grounded summary
```

The product rules and authorization model are governed by the Node 1 final lock:

`01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md`

## Node 1 — Product + Authorization Rework

```text
Status → 🔒 FINAL LOCKED / COMPLETE
```

The Node 1 final-lock record explicitly states that Node 1 is formally locked and complete, with Claude independent final review:

```text
APPROVE — NO BLOCKING FINDINGS
```

The locked model includes:

- exactly 1 application identity per Auth User;
- exactly 1 application role per Auth User;
- role = Company OR Driver;
- contextual trip participant relationships;
- trip lifecycle and delivery sequence;
- server-side authorization / IDOR rules;
- concurrency rules;
- authentication requirements derived from the locked model.

This status is based on the existing Node 1 FINAL LOCK record; it does not claim that every later implementation acceptance criterion has been independently re-verified in this checkpoint.

## Node 2 — Authentication + Identity

```text
Status → 🔵 ACTIVE DESIGN / NOT LOCKED
```

Node 2 broad authentication/identity investigations are complete. The current contract is:

`02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`

Status:

```text
DRAFT / NOT LOCKED
```

Claude independently reviewed the draft and found it **NOT READY FOR LOCK** because several load-bearing decisions remain unresolved.

### Resolved evidence stage

```text
Broad Node 2 investigation              ✅ COMPLETE
Remaining auth evidence investigation  ✅ COMPLETE
Signup/onboarding investigation         ✅ COMPLETE
Claude independent contract review      ✅ COMPLETE
```

### Current Node 2 decisions still requiring resolution

1. Signup / onboarding consistency
2. Email-confirmation policy
3. Session lifecycle / refresh
4. One-user → one-identity enforcement mechanism
5. Authentication rate-limiting policy
6. RLS / service-role boundary for the Node 2 contract
7. Final acceptance-test matrix

### Signup / onboarding evidence

The targeted investigation established that the current signup flow performs Auth User creation and application identity creation as separate operations rather than one database transaction.

A verified current failure state is:

```text
Auth User EXISTS
Application identity MISSING
```

The investigation also identified the reverse orphan risk associated with the current `ON DELETE SET NULL` relationship.

No implementation fix has been authorized from this evidence.

## Security / Authentication State

```text
RLS investigation                    → CLOSED / VERIFIED
Rate-limiting architecture            → DECIDED
IDOR / API authorization              → LOCKED AS NODE 1 CONTRACT
Authentication implementation          → PAUSED
```

RLS should not be reopened unless new contradictory evidence appears.

Authentication implementation remains paused until the Node 2 contract is designed, independently reviewed, and locked.

## Active Roadmap Position

The active execution roadmap remains the existing 7-Node roadmap in `ROADMAP.md`.

```text
Historical Core MVP                  → IMPLEMENTED / VERIFIED
Node 1 Product + Authorization       → 🔒 COMPLETE / LOCKED
Node 2 Authentication + Identity    → 🔵 ACTIVE DESIGN / NOT LOCKED
Node 3 Company Trip Creation        → FUTURE
Node 4 Driver Marketplace            → FUTURE
Node 5 Whole Delivery Tracking       → FUTURE
Node 6 Security + Evidence           → FUTURE
Node 7 AI + Final Integration + Demo → FUTURE
```

**No roadmap rewrite is made by this checkpoint.**

## Checkpoint

Chat11 checkpoint:

`00_PROJECT_CONTROL/CHECKPOINTS/Chat11_Day4_Node2_Checkpoint.md`

The checkpoint preserves the transition from the Node 1 lock into active Node 2 contract resolution without treating the Node 2 draft as locked.

## Subnode Rule

A Subnode is used only for significant unexpected work inside a Node.

```text
Small bug
→ fix inside current Node

Significant unexpected issue
→ create Subnode

Major blocker / architecture change
→ stop and reassess roadmap
```

If one Node accumulates 3 or more Subnodes, perform an explicit roadmap reassessment.

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

## Next Action

Do **not** resume authentication implementation.

The next project action is to resolve the Node 2 signup/onboarding consistency decision from the verified investigation evidence, then resolve the remaining Node 2 contract decisions before the contract is locked.
