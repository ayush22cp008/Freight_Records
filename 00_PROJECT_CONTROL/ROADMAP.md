# ROADMAP.md

**Project:** Freight — AI Builders Hackathon  
**Hackathon window:** Aug 21 – Sep 15, 2026  
**Roadmap status:** ACTIVE EXECUTION ROADMAP — Node/Subnode model approved after Chat9 review; Node 7 execution materially rephased and expanded by approved Chat29 reassessment.  
**Current execution day:** Day 14  
**Current chat:** Chat 30  

> Historical day-by-day planning and the original Core MVP remain preserved as project history. The active remaining execution model is organized in 7 Nodes with merged stretch work and a controlled Subnode mechanism. Chat29 is the approved execution clarification for Node 7.

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
IDOR / API authorization              → VERIFIED / CLOSED IN NODE 6
Authentication implementation          → COMPLETE / ACCEPTED
```

Do not reopen the RLS investigation unless new contradictory evidence appears.

Do not reopen Nodes 1–6 unless new evidence identifies a regression or a specific reviewer requirement.

---

# 3. Active Execution Model — 7 Nodes

| Node | Work | Baseline Days | Priority | Dependency | Status |
|---|---|---:|---|---|---|
| **Node 1** | Product + Authorization Rework | 2 | Critical | None | 🔒 COMPLETE / LOCKED |
| **Node 2** | Authentication + Identity | 3 | Critical | Node 1 | 🔒 COMPLETE / ACCEPTED |
| **Node 3** | Company Trip Creation + Publishing | 3 | Critical | Nodes 1–2 | 🔒 COMPLETE / ACCEPTED |
| **Node 4** | Driver Marketplace + Atomic Claim | 3 | Critical | Node 3 | 🔒 COMPLETE / ACCEPTED |
| **Node 5** | Whole Delivery Tracking | 5 | Critical | Node 4 | 🔒 COMPLETE / ACCEPTED |
| **Node 6** | Security + Evidence | 3 | Critical | Nodes 2–5 | 🔒 COMPLETE / ACCEPTED |
| **Node 7** | AI + Final Integration + Demo | 3 baseline / rephased | Critical | Nodes 3–6 | 🔵 ACTIVE / NEXT EXECUTION |
| **Total** | | **22 baseline days** | | | |

These baseline durations are planning estimates, not hard deadlines. Node 7 has been materially rephased by Chat29 because the final hackathon scope now includes a full three-portal UI/UX redesign and prioritized additional features. Actual duration must be recorded after Node 7 closure.

### Short-term goal rule

A Node is one short-term milestone. Prefer roughly 1–2 days for simple Nodes/tasks where practical, but retain the baseline duration when the Node genuinely contains larger scope. A Node is complete only when its acceptance criteria are satisfied and Ayush manually verifies the result.

---

# 4. NODE 1 — Product + Authorization Rework

**Baseline:** 2 days  
**Status:** 🔒 COMPLETE / LOCKED  

Node 1 locked the product, role, identity, trip, lifecycle, eligibility, atomic-claim, authorization, and IDOR model. Its design responsibility is complete. Node 6 implemented and verified the resulting security boundaries.

---

# 5. NODE 2 — Authentication + Identity

**Baseline:** 3 days  
**Status:** 🔒 COMPLETE / ACCEPTED  

Authentication and identity implementation was completed and accepted against the locked product model.

---

# 6. NODE 3 — Company Trip Creation + Publishing

**Baseline:** 3 days  
**Status:** 🔒 COMPLETE / ACCEPTED  

Company trip creation, publishing, receiver relationship, trip details, offer, and required authorization behavior were completed and accepted.

---

# 7. NODE 4 — Driver Marketplace + Atomic Claim

**Baseline:** 3 days  
**Status:** 🔒 COMPLETE / ACCEPTED  

Node 4 completed the eligible-trip marketplace, trip evaluation, driver acceptance, atomic first-valid claim, assigned-driver persistence, losing-driver behavior, server-side identity resolution, and client-ID manipulation protection.

Atomic first-valid acceptance was a baseline Node 4 requirement, not stretch work.

---

# 8. NODE 5 — Whole Delivery Tracking

**Baseline:** 5 days  
**Status:** 🔒 COMPLETE / ACCEPTED  

## Verified lifecycle

```text
Pickup
→ Arrival
→ Check-in
→ Load
→ Depart
→ In transit
→ Destination
→ Receiver Arrival
→ Receiver Check-in
→ Unload / Delivery
→ Receiver confirmation
→ Completed
```

The required single-delivery lifecycle, immutable evidence timeline, destination/receiver actions, final dual confirmation, database completion state, and source synchronization were completed and manually accepted.

Optional Node 5 stretch work remains deferred and was not required for closure:

- Derived dwell-time display
- Mandatory Check-in photo enhancement
- Repeatable Add Evidence mid-trip event
- Geofence proximity badge
- Multiple-stop support

---

# 9. NODE 6 — Security + Evidence

**Baseline:** 3 days  
**Status:** 🔒 COMPLETE / ACCEPTED  

Node 6 formally verified:

- IDOR attack paths blocked
- Privileged API authorization
- Driver assignment boundaries
- Company relationship boundaries
- Atomic claim security
- Evidence immutability
- Rate limiting
- Security tests
- TypeScript check with 0 errors

Ayush approved the Node 6 verification and closure.

Node 6 completion checkpoint:

`00_PROJECT_CONTROL/CHECKPOINTS/Chat28_Node6_Completion_Checkpoint.md`

Verification report:

`03_IMPLEMENTATION/implementation_reports/Chat28_Node6_Security_Evidence_Verification_Report.md`

---

# 10. NODE 7 — AI + Final Integration + Demo

**Baseline:** 3 days  
**Status:** 🔵 ACTIVE / NEXT EXECUTION  
**Dependency:** Nodes 3–6  
**Execution plan:** Approved Chat29 reassessment  
**Current position:** Day 14 / Chat30

## Objective

Complete the final hackathon-ready Freight experience by stabilizing the AI/evidence layer, adding the high-value public evidence-sharing capability, redesigning all three portals, then performing one final integrated verification/demo cycle.

**Important:** The official Node 7 acceptance criteria remain unchanged. Chat29 changes the execution phasing, priorities, and scope protection; it does not create a new Node or weaken the Node 7 acceptance gate.

## Phase 1a — Baseline AI + Shareable Evidence

Build functionality first because Phase 1b redesign touches the same screens.

### Required work

1. AI evidence-grounded summary
2. Timeline integration
3. Public shareable read-only evidence link

### Phase 1a priority

The baseline AI/timeline behavior must be stable and evidence-grounded before moving into deeper AI features or UI redesign-dependent work.

## Phase 1b — Full 3-Portal UI/UX Redesign

Redesign the final user-facing experience across all three portals:

1. Driver portal
2. Company portal
3. Reviewer portal

Use a unified visual/design system and improve navigation clarity, information hierarchy, workflow discoverability, and demo presentation.

Phase 1b absorbs the remaining Final API/UI integration and UX/demo polish work where those concerns are coupled to the redesigned screens.

## Phase 3 — New Add-On Features (Conditional)

Only begin after Phase 1a and Phase 1b are complete and stable.

Potential add-ons, in priority order:

- AI inconsistency detection — conditional; only after baseline AI/timeline stability.
- Confidence/completeness scoring — optional/time permitting.
- Multi-signal evidence cross-check — optional/time permitting.
- Natural-language Q&A over deterministic evidence — optional/time permitting.

If no meaningful time remains, skip Phase 3. Do not start speculative work.

## Explicit scope cuts

```text
Video capture                         → CUT
AI inconsistency detection            → CONDITIONAL
Optional AI-depth enhancements        → CUT unless higher-priority work is complete early
Public shareable read-only evidence   → KEPT / HIGH PRIORITY
```

## Final Step — Exactly Once

The final step is separate from the feature phases and happens once at the end:

1. Full E2E across Driver, Company, and Reviewer roles
2. Critical bug-fixing buffer
3. Realistic demo data/scenario
4. Hackathon presentation/demo flow
5. Final rehearsal

Do not repeat the complete final E2E/demo cycle after every feature.

## Explicit execution sequence

```text
Phase 1a
   ↓
Phase 1b
   ↓
Phase 3 (conditional)
   ↓
Final step — E2E + bugfix + demo + presentation
```

## Node 7 Baseline Tasks — Preserved

The official baseline remains:

- AI evidence-grounded summary
- Timeline integration
- Final API/UI integration
- End-to-end test
- Realistic demo data/scenario
- Security regression
- Critical bug fixing
- UX/demo polish
- Hackathon presentation/demo flow

These are distributed across the approved phases rather than removed from the Node.

## AI Boundary

AI may summarize, organize, or cross-check deterministic evidence but must not invent GPS, timestamps, event types, or unsupported blame/causality.

## Node 7 Acceptance Criteria — UNCHANGED

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

Node 7 is complete only when these acceptance criteria are satisfied and Ayush manually verifies the final result.

---

# 11. Node 7 Execution Governance — Chat29 Lock

The detailed approved reassessment is recorded at:

`00_PROJECT_CONTROL/Chat29_Node7_Roadmap_Reassessment_Phasing.md`

This record is the authoritative execution clarification for Node 7 from Day 14 / Chat30 onward.

### Governance rules

- Do not reopen Nodes 1–6 unless regression or reviewer evidence requires it.
- Do not bypass Phase 1a and begin speculative Phase 3 work.
- Do not begin Phase 3 until Phase 1a and Phase 1b are stable.
- Do not treat optional features as Node 7 acceptance requirements.
- Do not let UI redesign invalidate the evidence/security architecture already accepted.
- Preserve the AI evidence-grounding boundary.
- Keep the final E2E/demo/presentation cycle as a single final step.

---

# 12. Subnode System

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

### Subnode escalation rule

```text
3 or more Subnodes under one Node
        ↓
Roadmap reassessment required
```

This does not automatically create a new Node. It forces an explicit review of scope, duration, dependencies, and whether the parent Node or roadmap needs adjustment.

---

# 13. Node Completion Rule

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

# 14. Roadmap Change Rule

Explicitly revise this roadmap when:

- Product/architecture decision changes.
- A significant blocker appears.
- A Subnode materially changes the schedule.
- A Node accumulates 3+ Subnodes.
- A requirement is added/removed.
- Actual implementation is materially faster/slower than planned.
- Hackathon time requires reprioritization.

Do not silently deviate from the active roadmap.

Chat29 is itself the recorded roadmap reassessment that materially rephased Node 7 without changing its official acceptance criteria.

---

# 15. Historical Core MVP — PRESERVED

The original Core MVP remains an important completed project milestone:

1. Driver-only login / pre-seeded trip foundation
2. Arrival → Check-in → Departure event flow
3. GPS + authoritative server timestamp
4. Mandatory Arrival/Departure photos, optional Check-in photo
5. Immutable insert-only event storage
6. Chronological timeline
7. AI evidence summary

These capabilities were implemented and verified before the Chat8 product/security rework. They are not discarded; they are extended into the broader company → marketplace → delivery → evidence product story.

---

# 16. Historical Stretch Features — PRESERVED THROUGH MERGE

The useful historical stretch concepts remain represented through the active Node structure:

| Feature | Active placement | Current priority |
|---|---|---|
| Public shareable evidence link | Node 7 Phase 1a | High / kept |
| AI inconsistency detection | Node 7 Phase 3 | Conditional |
| Derived dwell-time | Node 5 | Deferred / optional |
| Mandatory Check-in photo | Node 5 | Deferred / optional |
| Repeatable Add Evidence | Node 5 | Deferred / riskier |
| Geofence proximity badge | Node 5 | Deferred / low |
| Company role/dashboard | Node 3 | Completed as required |
| Video capture | Node 7 | Cut |

The old standalone stretch-day calendar is superseded by the approved phased execution model.

---

# 17. Current Active Position

```text
Historical Core MVP                → IMPLEMENTED / VERIFIED
Node 1                              → COMPLETE / LOCKED
Node 2                              → COMPLETE / ACCEPTED
Node 3                              → COMPLETE / ACCEPTED
Node 4                              → COMPLETE / ACCEPTED
Node 5                              → COMPLETE / ACCEPTED
Dashboard follow-up                → CLOSED / VERIFIED
Historical AI follow-up            → CLOSED / VERIFIED
Node 6                              → COMPLETE / ACCEPTED
Day 13                             → CLOSED / NO PROJECT WORK
Day 14 / Chat30                    → ACTIVE
Current Node                       → NODE 7
Current phase                      → PHASE 1a
Current focus                      → AI + Timeline + Public Shareable Evidence
```

---

# 18. Working Method

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

For project-control records:

```text
00_PROJECT_CONTROL/
```

The historical records and the active roadmap must remain auditable.
