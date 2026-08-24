# CURRENT_STATUS.md

**Last updated:** Aug 24, 2026

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

The Core MVP is **not being discarded**. It is the verified foundation being extended into the broader product model defined by the Chat8 roadmap rework.

## Current Product Direction

The active product model is:

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

Important active direction:

- Company creates/publishes the trip opportunity; the driver does not create/own the trip.
- Company creator/sender and receiving-company relationships are contextual per trip.
- Eligible drivers evaluate available trips and choose whether to accept.
- Exactly one valid driver must win simultaneous acceptance through backend/database atomicity.
- After assignment, driver-side transport events must be authorized against the assigned driver/trip relationship.
- AI remains an evidence-grounded interpretation layer and must not invent or replace deterministic evidence.

## Security / Authentication State

```text
RLS investigation                    → CLOSED / VERIFIED
Rate-limiting architecture            → DECIDED
IDOR / API authorization              → OPEN
Authentication implementation          → PAUSED
```

RLS should not be reopened unless new contradictory evidence appears.

Authentication implementation is paused until the current product, role, identity, trip lifecycle, eligibility, atomic acceptance, authorization matrix, and IDOR rules are locked and verified through the active Node 1 gate.

## Active Roadmap

The active execution roadmap is now **7 Nodes**:

| Node | Objective | Baseline | Status |
|---|---|---:|---|
| Node 1 | Product + Authorization Rework | 2 days | 🔵 **CURRENT / NEXT** |
| Node 2 | Authentication + Identity | 3 days | 🔵 BLOCKED |
| Node 3 | Company Trip Creation + Publishing | 3 days | 🔵 FUTURE |
| Node 4 | Driver Marketplace + Atomic Claim | 3 days | 🔵 FUTURE |
| Node 5 | Whole Delivery Tracking | 5 days | 🔵 FUTURE |
| Node 6 | Security + Evidence | 3 days | 🔵 FUTURE |
| Node 7 | AI + Final Integration + Demo | 3 days | 🔵 FUTURE |

Baseline total: **22 planned days**. These are estimates, not hard deadlines.

### Current Node Gate

**Node 1 — Product + Authorization Rework** is the immediate active milestone.

Node 1 must lock:

- Company / Driver roles
- Auth user → Company/Driver identity mapping
- Contextual creator/sending and receiving-company relationships
- Trip relationships and required data
- Trip state machine
- Driver eligibility
- Atomic first-valid acceptance rule
- Complete authorization matrix
- IDOR/API protection rules
- Authentication requirements derived from the final model

**Responsibility split:**

```text
Node 1 → design + lock authorization / IDOR rules
Node 6 → implement + verify authorization / IDOR rules
```

Node 2 authentication implementation must not begin until the Node 1 gate is explicitly verified.

## Subnode Rule

A Subnode is used only for **significant unexpected work inside a Node**.

```text
Small bug
→ fix inside current Node

Significant unexpected issue
→ create Subnode

Major blocker / architecture change
→ stop and reassess roadmap
```

Atomic first-winner acceptance is already a known Core requirement of Node 4 and is **not** treated as an unexpected Subnode.

If one Node accumulates **3 or more Subnodes**, a roadmap reassessment is required.

## Stretch Feature Strategy

The previous standalone stretch list has been merged into the relevant Nodes rather than remaining as a separate execution track.

- Node 3: company dashboard/role capabilities required by the locked trip-creation model.
- Node 5: dwell-time, mandatory Check-in photo, repeatable Add Evidence, and geofence badge — conditional and lower priority than the core delivery lifecycle.
- Node 7: public shareable evidence link and AI inconsistency detection — higher-value optional enhancements after baseline integration; video capture is lowest priority.

Core Node acceptance criteria always take priority over stretch work.

## Do Not Re-discuss / Preserve

The following remain established foundations unless new evidence or an explicit recorded decision requires change:

- Existing Core MVP evidence architecture
- Event immutability architecture
- Verified RLS findings
- Existing deterministic evidence principle
- Core navigation/evidence foundations already implemented

The broader product/security model is now governed by the active `ROADMAP.md` and the Chat8 handoff records.

## Next Action

Do **not** resume authentication implementation yet.

The next project action is to complete **Node 1 — Product + Authorization Rework** using the investigation-first workflow, then derive/lock the authentication requirements for Node 2.

For implementation handoffs:

```text
03_IMPLEMENTATION/prompts/
```

For Antigravity implementation reports:

```text
03_IMPLEMENTATION/implementation_reports/
```

For investigations:

```text
05_DEBUGGING/investigations/
```
