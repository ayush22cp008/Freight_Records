# Chat9 — Roadmap Review: Node + Subnode + Stretch Merge

**Project:** Freight — AI Builders Hackathon  
**Purpose:** Review the existing 25-day roadmap against the Chat8 product/security rework, preserve useful stretch features, and define the active Node/Subnode execution model for Claude review.

## 1. Review Outcome

The existing `00_PROJECT_CONTROL/ROADMAP.md` remains useful as historical planning context because it contains the original Core MVP schedule, stretch priorities, testing buffer, UI polish, and submission preparation.

The Chat8 roadmap rework establishes the new product model and a 7-Node execution structure. These should be merged rather than replacing the historical roadmap blindly.

The new active planning model is:

```text
Historical roadmap
        +
Chat8 product/security direction
        +
Existing stretch priorities
        ↓
Active 7-Node roadmap
```

The existing historical roadmap must remain preserved until the project control roadmap is deliberately updated.

## 2. Active 7-Node Execution Model

The active remaining hackathon work is organized into seven Nodes:

| Node | Objective | Baseline Target | Priority |
|---|---|---:|---|
| Node 1 | Product + Authorization Rework | 2 days | 🔴 Critical |
| Node 2 | Authentication + Identity | 3 days | 🔴 Critical |
| Node 3 | Company Trip Creation + Publishing | 3 days | 🔴 Critical |
| Node 4 | Driver Marketplace + Atomic Claim | 3 days | 🔴 Critical |
| Node 5 | Whole Delivery Tracking | 5 days | 🔴 Critical |
| Node 6 | Security + Evidence | 3 days | 🔴 Critical |
| Node 7 | AI + Final Integration + Demo | 3 days | 🔴 Critical |
| **Total** | | **22 days** | |

These durations are planning estimates, not hard deadlines. Actual duration must be recorded after each Node.

## 3. Short-Term Goal Rule

A Node is one short-term milestone.

Preferred operating target:

```text
1 Node ≈ 1–2 days where practical
```

However, the Node duration is not a hard constraint. Complexity varies by Node. The current Chat8 baseline intentionally allocates 2–5 days by scope.

A Node is complete only when its acceptance criteria are satisfied and manually verified. Time passing does not make a Node complete.

## 4. Node Structure

Every Node should track:

- Node ID
- Objective
- Dependencies
- Tasks
- Merged stretch work that naturally belongs to the Node
- Acceptance criteria
- Planned duration
- Actual duration
- Status
- Investigation/report requirements
- Manual verification requirements
- Roadmap-change requirement, if applicable

## 5. Subnode Model

A Subnode is a smaller, explicitly tracked unit of significant work inside a parent Node.

Subnodes are created only when unexpected work is substantial enough to require its own investigation, implementation, or verification.

### Rules

```text
Small bug
→ fix inside the current Node

Significant unexpected issue
→ create a Subnode under the current Node

Major blocker / architecture change
→ stop and reassess the roadmap before continuing
```

Example:

```text
Node 4 — Driver Marketplace + Atomic Claim

4.1 Available trip list
4.2 Trip details / offer
4.3 Accept trip
4.4 Atomic claim

Unexpected major race-condition problem
        ↓
4.S1 — Race-condition investigation
4.S2 — Atomic-claim fix
4.S3 — Concurrency verification
        ↓
Return to Node 4
```

A Subnode does not become a replacement for the parent Node. It exists to contain significant unexpected work while keeping the parent milestone intact.

## 6. Stretch Feature Merge Strategy

The original roadmap contains eight stretch features. They should not remain as an unrelated second roadmap.

They should be attached to the Node where they naturally belong while preserving their original priority logic.

### Node 3 — Company Trip Creation + Publishing

Relevant merged stretch scope:

- **Company role/dashboard** when needed for company trip creation/publishing.

This must not be implemented as an old company-directly-assigns-driver model. The current product model is company publishes a trip opportunity and eligible drivers choose whether to accept.

### Node 5 — Whole Delivery Tracking

Relevant merged stretch scope:

- **Derived dwell-time display** — derived from deterministic timestamps.
- **Mandatory photo at Check-in** — only if it remains appropriate after the core delivery flow is reliable.
- **Repeatable “Add Evidence” mid-trip event** — higher-risk schema work; only if the core single-delivery lifecycle is stable.
- **Geofence proximity badge** — only if it can be added without compromising core delivery reliability.

### Node 6 — Security + Evidence

Relevant scope:

- IDOR/API authorization
- Role/relationship authorization
- Evidence integrity
- Rate-limit implementation/verification

The existing rate-limiting architecture decision remains unchanged; only implementation/verification is tracked here.

### Node 7 — AI + Final Integration + Demo

Relevant merged stretch scope:

- **Public shareable read-only evidence link** — high demo value and low relative complexity.
- **AI inconsistency detection** — deepens the AI evidence layer.
- UI polish / demo readiness.
- **Video capture** — highest complexity and lowest incremental evidence value; only if all higher-priority work is complete and time remains.

The original AI-depth enhancement ideas remain optional upgrades to the AI layer:

- confidence/completeness scoring;
- multi-signal evidence cross-checking;
- natural-language Q&A over deterministic evidence.

## 7. Priority and Scope Protection

The existing stretch priorities should still influence what is built first:

1. High-value, lower-risk demo improvements first.
2. AI depth where it strengthens the core evidence story.
3. Moderate-risk structural features only after the main lifecycle is reliable.
4. Highest-risk / lowest-value features are cut first if time becomes constrained.

The core delivery story always takes priority over stretch work.

```text
Core Node work
   ↓
High-value attached stretch
   ↓
Moderate-risk stretch
   ↓
Optional stretch
```

Never sacrifice a Core Node acceptance criterion to fit a stretch feature.

## 8. Active Node Definitions

### NODE 1 — Product + Authorization Rework

Objective: lock the product, role, lifecycle, eligibility, claim, authorization, and IDOR model before authentication implementation.

Required decisions include:

- Company and Driver roles.
- Auth user → Company/Driver identity mapping.
- Contextual creator/sending and receiving company relationships.
- Minimum trip relationships.
- Trip state machine.
- Driver eligibility.
- Atomic first-valid acceptance.
- Full authorization matrix.
- API/IDOR protection rules.
- Authentication requirements derived from the locked model.

Gate: Node 2 cannot begin until Node 1 acceptance criteria are explicitly verified.

### NODE 2 — Authentication + Identity

Objective: implement authentication against the final Node 1 identity/role model.

Includes company/driver authentication, role identification, identity mapping, protected routes, sessions, and wrong-role/unauthorized-access testing.

### NODE 3 — Company Trip Creation + Publishing

Objective: allow a company to create and publish a complete trip opportunity.

Includes pickup, destination, receiving company, distance, duration, payment/offer, shipment details, publish flow, and company authorization.

Manual offer increases remain allowed for the hackathon; automated pricing is deferred.

### NODE 4 — Driver Marketplace + Atomic Claim

Objective: allow eligible drivers to evaluate available trips and atomically claim one.

Includes available trip list, trip details, offer visibility, acceptance, first-winner claim, assignment persistence, losing-driver response, and race-condition testing.

### NODE 5 — Whole Delivery Tracking

Objective: extend the existing three-event evidence demo into a reliable single-delivery lifecycle from pickup through destination and delivery confirmation.

Target flow:

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

Existing evidence integrity remains foundational.

### NODE 6 — Security + Evidence

Objective: close the authorization/IDOR gap and verify security boundaries of the final product model.

Includes privileged-route authorization, trip/driver/company relationship checks, immutable evidence, timestamp/GPS/photo integrity, rate-limit implementation/verification, and direct API/security tests.

### NODE 7 — AI + Final Integration + Demo

Objective: integrate the complete delivery scenario, evidence timeline, AI summary, high-value stretch features, final regression, and demo preparation.

AI remains evidence-grounded and must not invent or replace deterministic evidence.

## 9. Subnode Tracking Template

When a significant unexpected issue occurs, track it as:

```text
SUBNODE:
PARENT NODE:
TRIGGER:

OBJECTIVE:

EVIDENCE / INVESTIGATION:

ROOT CAUSE:

FIX:

VERIFICATION:

STATUS:

IMPACT ON PARENT NODE:

ROADMAP CHANGE REQUIRED:
YES / NO
```

The normal investigation-first workflow still applies. Do not jump straight from symptom to fix.

## 10. Node Completion Rule

A Node becomes COMPLETE only when:

```text
[ ] Required tasks complete
[ ] Acceptance criteria satisfied
[ ] Required investigations resolved or explicitly deferred
[ ] Security checks complete for the Node's scope
[ ] Build/test evidence recorded
[ ] Ayush manual verification completed
[ ] Implementation report recorded
```

Actual duration must be recorded alongside planned duration.

## 11. Roadmap Change Rule

The roadmap should be explicitly revised when:

- a product/architecture decision changes;
- a significant blocker appears;
- a Subnode materially changes the schedule;
- a requirement is added/removed;
- implementation is substantially faster/slower than planned;
- hackathon time requires reprioritization.

Do not silently deviate from the active roadmap.

## 12. Historical Roadmap Preservation

The existing `00_PROJECT_CONTROL/ROADMAP.md` is historical planning context and should not be erased or silently rewritten as part of this review.

The purpose of this document is to propose the merged active execution model. The project-control roadmap can be updated deliberately after review/approval.

## 13. Review Request for Claude

Please review this roadmap model against the existing Chat8 product/security handoffs and the historical `ROADMAP.md`.

Focus the review on:

1. Whether the 7 Nodes cover the required remaining product work.
2. Whether the merged stretch features are placed under sensible Nodes.
3. Whether the Subnode model is clear and should remain limited to significant unexpected work.
4. Whether any Node has a missing dependency or acceptance criterion.
5. Whether the sequencing creates a hidden architectural/security problem.
6. Whether any stretch feature should be moved, deferred, or removed.
7. Whether the 22-day baseline is internally coherent as a planning model.
8. Whether the roadmap cleanly preserves the already-completed Core MVP while moving to the broader product model.

### Expected review output

Return:

```text
REVIEW STATUS: APPROVE / APPROVE WITH CHANGES / REJECT

CRITICAL CHANGES:
- ...

RECOMMENDED CHANGES:
- ...

OPTIONAL CHANGES:
- ...

NODE-LEVEL FINDINGS:
- Node 1: ...
- Node 2: ...
- Node 3: ...
- Node 4: ...
- Node 5: ...
- Node 6: ...
- Node 7: ...

SUBNODE MODEL:
APPROVE / CHANGE

STRETCH MERGE:
APPROVE / CHANGE

FINAL RECOMMENDATION:
...
```

## 14. Source Records

This review is based on:

- `01_BRAIN_HANDOFFS/ChatGPT/Chat8_New_Update_Hackathon_Node_Roadmap_Rework.md`
- `01_BRAIN_HANDOFFS/ChatGPT/Chat8_New_Update_Product_Model_and_Security_Direction.md`
- `01_BRAIN_HANDOFFS/ChatGPT/Chat8_New_Update_Authentication_Implementation_Pause_Checkpoint.md`
- `00_PROJECT_CONTROL/ROADMAP.md`

The Chat8 records define the current product/security direction; the existing roadmap supplies the historical Core MVP and stretch planning context.
