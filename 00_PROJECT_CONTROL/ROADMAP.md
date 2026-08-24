# ROADMAP.md

**Project:** Freight — AI Builders Hackathon  
**Hackathon window:** Aug 21 – Sep 15, 2026  
**Roadmap status:** ACTIVE EXECUTION ROADMAP — Node/Subnode model approved after Chat9 review.  
**Baseline planning horizon:** approximately 22 remaining hackathon days after the Chat8 product/security rework.

> Historical day-by-day planning and the original Core MVP remain preserved as project history. The active remaining execution model is now organized into 7 Nodes with merged stretch work and a controlled Subnode mechanism.

---

# 1. Current Product Direction — LOCKED FOR ACTIVE PLANNING

The target product story is a focused end-to-end Freight delivery lifecycle:

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

Important product rules:

- The company creates and publishes the trip opportunity; the driver does not create/own the trip.
- Company sender/creator and receiver roles are contextual to each trip, not permanently fixed global roles.
- Eligible drivers evaluate the trip and choose whether to accept it.
- Acceptance must be atomic so exactly one valid driver wins.
- After assignment, only the assigned driver may perform driver-side transport events for that trip.
- AI remains an evidence-grounded interpretation layer and must not invent or replace deterministic evidence.

---

# 2. Security / Architecture Starting State

```text
RLS investigation                    → CLOSED / VERIFIED
Rate-limiting architecture            → DECIDED
IDOR / API authorization              → OPEN
Authentication implementation          → PAUSED
```

Do not reopen the RLS investigation unless new contradictory evidence appears.

Authentication implementation remains blocked until the product/role/trip/authorization model in Node 1 is explicitly locked and verified.

---

# 3. Active Execution Model — 7 Nodes

| Node | Work | Baseline Days | Priority | Dependency | Status |
|---|---|---:|---|---|---|
| **Node 1** | Product + Authorization Rework | 2 | 🔴 Critical | None | 🔵 NEXT |
| **Node 2** | Authentication + Identity | 3 | 🔴 Critical | Node 1 | 🔵 BLOCKED |
| **Node 3** | Company Trip Creation + Publishing | 3 | 🔴 Critical | Nodes 1–2 | 🔵 FUTURE |
| **Node 4** | Driver Marketplace + Atomic Claim | 3 | 🔴 Critical | Node 3 | 🔵 FUTURE |
| **Node 5** | Whole Delivery Tracking | 5 | 🔴 Critical | Node 4 | 🔵 FUTURE |
| **Node 6** | Security + Evidence | 3 | 🔴 Critical | Nodes 2–5 | 🔵 FUTURE |
| **Node 7** | AI + Final Integration + Demo | 3 | 🔴 Critical | Nodes 3–6 | 🔵 FUTURE |
| **Total** | | **22** | | | |

These are planning estimates, not hard deadlines. Actual duration must be recorded after each Node.

### Short-term goal rule

A Node is one short-term milestone. Prefer roughly 1–2 days for simple Nodes/tasks where practical, but retain the baseline duration when the Node genuinely contains larger scope. A Node is complete only when its acceptance criteria are satisfied and Ayush manually verifies the result.

---

# 4. NODE 1 — Product + Authorization Rework

**Baseline:** 2 days  
**Status:** 🔵 NEXT  
**Blocks:** Node 2 and all later authentication/authorization-sensitive work

## Objective

Lock the product, role, identity, trip, lifecycle, eligibility, atomic-claim, authorization, and IDOR model before resuming authentication implementation.

## Tasks

1. Lock Company and Driver identities.
2. Lock Auth user → Company/Driver identity mapping.
3. Lock contextual creator/sending-company and receiving-company relationships.
4. Lock minimum trip relationships and required commercial/trip data.
5. Lock trip state machine and legal transitions.
6. Lock driver eligibility rules.
7. Lock atomic first-valid acceptance rule.
8. Lock the full authorization matrix.
9. Define server-side IDOR/API relationship checks.
10. Derive final authentication requirements from the locked model.

## Authorization / IDOR responsibility

**Node 1 = design + lock.**  
**Node 6 = implement + verify.**

Node 1 must not leave unresolved `TBD` permissions in the authorization matrix.

## Acceptance Criteria

```text
[ ] Roles locked
[ ] Identity mapping locked
[ ] Company/receiver relationship locked
[ ] Trip relationships locked
[ ] Trip state machine locked
[ ] Driver eligibility locked
[ ] Atomic acceptance rule locked
[ ] Authorization matrix fully resolved
[ ] IDOR protection rules defined
[ ] Authentication requirements derived
[ ] Ayush verification complete
```

**Gate:** Node 2 cannot begin until these criteria are explicitly verified.

---

# 5. NODE 2 — Authentication + Identity

**Baseline:** 3 days  
**Status:** 🔵 BLOCKED BY NODE 1  
**Dependency:** Node 1

## Objective

Implement authentication against the final Node 1 identity and authorization model.

## Tasks

- Company authentication
- Driver authentication
- Role identification
- Auth user → Company/Driver mapping
- Protected routes
- Session handling
- Authenticated request context
- Wrong-role / unauthorized-access testing

## Acceptance Criteria

```text
[ ] Company can authenticate
[ ] Driver can authenticate
[ ] Authenticated identity is reliable
[ ] Application can determine Company vs Driver
[ ] Identity maps correctly
[ ] Protected routes reject unauthenticated users
[ ] Wrong-role access is rejected
[ ] Authentication tests pass
[ ] Ayush verification complete
```

---

# 6. NODE 3 — Company Trip Creation + Publishing

**Baseline:** 3 days  
**Status:** 🔵 FUTURE  
**Dependency:** Nodes 1–2

## Objective

Allow a company to create and publish a complete trip opportunity for eligible drivers.

## Tasks

- Company trip creation UI/API
- Pickup
- Destination / receiving company
- Distance
- Expected duration
- Payment / offer
- Shipment details
- Draft state if required
- Publish action
- Company relationship/authorization checks

## Merged stretch / supporting feature

- Company dashboard/role functionality needed for trip creation and publishing belongs here.
- The exact required company capabilities are **locked in Node 1**, not deferred to Node 3.
- Manual offer increases are allowed for the hackathon; automated pricing is deferred.

## Acceptance Criteria

```text
[ ] Company can create valid trip
[ ] Receiver relationship stored
[ ] Trip details visible
[ ] Initial offer stored
[ ] Company can publish
[ ] Published trip becomes available according to eligibility rules
[ ] Unauthorized company/user actions are rejected
[ ] Ayush verification complete
```

---

# 7. NODE 4 — Driver Marketplace + Atomic Claim

**Baseline:** 3 days  
**Status:** 🔵 FUTURE  
**Dependency:** Node 3

## Objective

Allow eligible drivers to discover, evaluate, and atomically claim one available trip.

## Tasks

- Available trip list
- Trip detail view
- Pickup/destination display
- Distance display
- Duration display
- Payment/offer display
- Driver acceptance
- Atomic first-winner claim
- Assigned-driver persistence
- Losing-driver response
- Concurrency/race-condition tests

## Core requirement — not stretch

Atomic first-valid acceptance is a **baseline Node 4 requirement**, not a hypothetical Subnode.

The backend/database must guarantee:

```text
Trip AVAILABLE
       ↓
Driver A accepts
Driver B accepts
       ↓
Exactly ONE wins
       ↓
Trip becomes CLAIMED
       ↓
assigned_driver_id = winner
```

Frontend controls must not be trusted for exclusivity.

## Acceptance Criteria

```text
[ ] Eligible drivers can see available trips
[ ] Driver can evaluate trip
[ ] Driver can accept
[ ] Exactly one simultaneous acceptance succeeds
[ ] Winner becomes assigned driver
[ ] Trip cannot be claimed again
[ ] Losing driver receives clear response
[ ] Assignment cannot be manipulated through client input
[ ] Concurrency tests pass
[ ] Ayush verification complete
```

---

# 8. NODE 5 — Whole Delivery Tracking

**Baseline:** 5 days  
**Status:** 🔵 FUTURE  
**Dependency:** Node 4

## Objective

Extend the existing three-event evidence workflow into a reliable single-delivery lifecycle.

## Target flow

```text
Pickup
 ↓
Arrival
 ↓
Check-in
 ↓
Load
 ↓
Depart
 ↓
In transit
 ↓
Destination
 ↓
Receiver Arrival
 ↓
Receiver Check-in
 ↓
Unload / Delivery
 ↓
Receiver confirmation
 ↓
Completed
```

## Tasks

### Pickup side

- Arrival
- GPS capture
- Server timestamp
- Photo/evidence as required
- Check-in
- Load
- Depart

### Transit

- In-transit state
- Trip status progression

### Destination

- Destination arrival
- Receiving-company relationship
- Receiver check-in
- Unload/delivery event
- Delivery confirmation
- Completion state

### Evidence

Preserve immutable event history and the established server-side evidence architecture.

## Merged stretch features — conditional

These features are attached to Node 5 because they naturally extend the delivery/evidence lifecycle:

1. **Derived dwell-time display** — low complexity, derived from deterministic timestamps.
2. **Mandatory Check-in photo** — optional enhancement after core flow stability.
3. **Repeatable “Add Evidence” mid-trip event** — higher-risk schema work; only if core single-delivery lifecycle is reliable.
4. **Geofence proximity badge** — only if it can be added without compromising core delivery reliability.

### Node 5 scope warning

Node 5 is the tightest Node by scope-to-time ratio. The four stretch items above are not mandatory to declare Node 5 complete.

If time becomes constrained, cut in this order first:

1. Geofence badge
2. Mandatory Check-in photo
3. Repeatable Add Evidence

The core delivery lifecycle always takes priority.

## Acceptance Criteria

```text
[ ] Published trip can progress to claimed/in-progress/delivered/completed
[ ] Pickup events work
[ ] Transit state works
[ ] Destination events work
[ ] Receiving company performs permitted action
[ ] Unauthorized actors are blocked
[ ] Evidence timeline remains coherent
[ ] End-to-end single-delivery scenario works
[ ] Ayush verification complete
```

Multiple-stop support remains deferred unless the core single-delivery lifecycle is already reliable and explicit approval is given.

---

# 9. NODE 6 — Security + Evidence

**Baseline:** 3 days  
**Status:** 🔵 FUTURE  
**Dependency:** Nodes 2–5

## Objective

Implement and verify the authorization/security boundaries defined in Node 1 and protect the final evidence workflow.

## Tasks

### Authorization / IDOR — implementation + verification

Node 1 designs/locks the matrix and relationship rules. Node 6 implements and verifies them across privileged APIs.

Test examples:

```text
Driver A → Trip assigned to Driver B → DENY
Driver B → Trip assigned to Driver B → ALLOW

Unrelated company → another company's private trip → DENY
Receiving company → permitted receiving action → ALLOW
Creating company → permitted creator action → ALLOW
```

### Evidence integrity

- Immutable event behavior
- Server timestamps
- GPS capture
- Photo/evidence constraints
- Event ordering/state rules

### Rate limiting

Implement/verify the already-decided rate-limiting architecture. Do not reopen the architecture question without new contradictory evidence.

### Security testing

- Direct API manipulation
- Forged trip IDs
- Forged driver IDs
- Forged company IDs
- Wrong-role requests
- Unassigned-driver event submission
- Receiver/creator boundary violations
- Duplicate acceptance
- Replay/duplicate event attempts as applicable

## Acceptance Criteria

```text
[ ] IDOR attack paths blocked
[ ] Every privileged API route has explicit authorization
[ ] Driver assignment boundary enforced
[ ] Company relationship boundary enforced
[ ] Atomic claim remains secure
[ ] Evidence remains immutable
[ ] Rate limiting verified
[ ] Security test results recorded
[ ] Ayush verification complete
```

---

# 10. NODE 7 — AI + Final Integration + Demo

**Baseline:** 3 days  
**Status:** 🔵 FUTURE  
**Dependency:** Nodes 3–6

## Objective

Integrate the complete delivery scenario, evidence timeline, AI layer, selected high-value stretch features, final regression, and demo preparation.

## Baseline tasks

- AI evidence-grounded summary
- Timeline integration
- Final API/UI integration
- End-to-end test
- Realistic demo data/scenario
- Security regression
- Critical bug fixing
- UX/demo polish
- Hackathon presentation/demo flow

## Merged stretch features — priority based

### High priority

- **Public shareable read-only evidence link** — high demo value and relatively low complexity.
- **AI inconsistency detection** — only after the baseline AI/timeline flow is stable.

### Optional / time permitting

- Additional AI-depth enhancements:
  - confidence/completeness scoring;
  - multi-signal evidence cross-checking;
  - natural-language Q&A over deterministic evidence.

- **Video capture** is the lowest-priority stretch item and should be attempted only if all higher-priority work is already complete and reliable.

### Node 7 scope protection

Node 7 is heavily loaded for its 3-day baseline. If time becomes constrained:

1. Video capture is cut first.
2. AI inconsistency detection is cut next.
3. Lower-value polish/features are cut before core integration or final verification.

## AI boundary

AI may summarize/organize/cross-check deterministic evidence but must not invent GPS, timestamps, event types, or unsupported blame/causality.

## Acceptance Criteria

```text
[ ] Complete delivery scenario works from company creation to completion
[ ] Evidence timeline visible
[ ] AI summary generated from recorded evidence
[ ] Security regression passes
[ ] Critical bugs resolved
[ ] Demo can be repeated reliably
[ ] Presentation story is coherent
[ ] Ayush verification complete
```

---

# 11. Subnode System

A **Subnode** is a smaller tracked unit of significant unexpected work inside a parent Node.

### Simple rule

```text
Small bug
→ fix inside current Node

Significant unexpected issue
→ create Subnode under current Node

Major blocker / architecture change
→ stop and reassess roadmap
```

### Example

```text
Node 4 — Driver Marketplace + Atomic Claim

4.1 Available trip list
4.2 Trip details / offer
4.3 Accept trip
4.4 Atomic claim

Unexpected significant issue
        ↓
4.S1 — Investigation
4.S2 — Fix
4.S3 — Verification
        ↓
Return to Node 4
```

Atomic acceptance itself is **not** a Subnode because it is already a known Core Node 4 requirement.

### Subnode escalation rule

A soft escalation trigger applies:

```text
3 or more Subnodes under one Node
        ↓
Roadmap reassessment required
```

This does not automatically create a new Node. It forces an explicit review of scope, duration, dependencies, and whether the parent Node or roadmap needs adjustment.

---

# 12. Node Completion Rule

A Node becomes `COMPLETE` only when:

```text
[ ] Required tasks complete
[ ] Acceptance criteria satisfied
[ ] Required investigations resolved or explicitly deferred
[ ] Security checks complete for the Node scope
[ ] Build/test evidence recorded
[ ] Ayush manual verification completed
[ ] Implementation report recorded
```

Record both:

```text
Planned duration
Actual duration
```

Time passing does not make a Node complete.

---

# 13. Roadmap Change Rule

Explicitly revise this roadmap when:

- Product/architecture decision changes.
- A significant blocker appears.
- A Subnode materially changes the schedule.
- A Node accumulates 3+ Subnodes.
- A requirement is added/removed.
- Actual implementation is materially faster/slower than planned.
- Hackathon time requires reprioritization.

Do not silently deviate from the active roadmap.

---

# 14. Historical Core MVP — PRESERVED

The original Core MVP remains an important completed project milestone:

1. Driver-only login / pre-seeded trip foundation
2. Arrival → Check-in → Departure event flow
3. GPS + authoritative server timestamp
4. Mandatory Arrival/Departure photos, optional Check-in photo
5. Immutable insert-only event storage
6. Chronological timeline
7. AI evidence summary

These capabilities were implemented and verified before the Chat8 product/security rework. They are not being discarded; they are being extended into the broader company → marketplace → delivery → evidence product story.

---

# 15. Historical Stretch Features — PRESERVED THROUGH MERGE

The original roadmap stretch concepts are retained through the active Node structure rather than as an independent day-by-day stretch track:

| Original stretch | Active Node placement | Priority |
|---|---|---|
| Public shareable evidence link | Node 7 | High |
| AI inconsistency detection | Node 7 | High / time permitting |
| Derived dwell-time | Node 5 | Optional |
| Mandatory Check-in photo | Node 5 | Optional |
| Repeatable Add Evidence | Node 5 | Riskier / optional |
| Geofence proximity badge | Node 5 | Low / cut early |
| Company role/dashboard | Node 3 | Defined by Node 1, implemented as needed for trip creation/publishing |
| Video capture | Node 7 | Lowest / cut first |

The old standalone stretch-day calendar is therefore superseded as the active execution mechanism, while its useful feature priorities are preserved.

---

# 16. Current Active Position

```text
Historical Core MVP                → IMPLEMENTED / VERIFIED
Chat8 product model                → CLARIFIED
RLS                                 → CLOSED / VERIFIED
Rate-limiting architecture         → DECIDED
IDOR/API authorization              → OPEN
Authentication implementation       → PAUSED
Active roadmap                      → 7 Nodes
Current Node                        → NODE 1
Node 1 objective                   → Product + Authorization Rework
```

Do not resume authentication implementation until Node 1 acceptance criteria are explicitly verified.

---

# 17. Working Method

Every Node continues to use the investigation-first project workflow:

```text
Observe
→ Investigate
→ Collect evidence
→ Determine root cause
→ Decide
→ Implement
→ Build/Test
→ Ayush manual verification
→ Record implementation report
→ Mark Node complete
```

Do not jump directly from an observed symptom to an implementation fix when investigation is required.

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

The historical records and the active roadmap must remain auditable.
