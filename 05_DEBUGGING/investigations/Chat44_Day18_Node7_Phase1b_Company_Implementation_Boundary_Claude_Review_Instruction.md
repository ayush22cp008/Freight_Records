# Chat44 — Day 18 — Node 7 — Phase 1b — Company Implementation Boundary — Claude Review Instruction

## Purpose

Perform an **independent peer review** of the proposed Company Portal Implementation Boundary before it is locked and used to create the Company implementation prompt.

This is a **review only**. Do not modify source code, modify the Company Blueprint, implement fixes, or create an implementation prompt.

## Review Context

- Project: Freight — AI Builders Hackathon
- Node: Node 7 — AI + Final Integration + Demo
- Phase: Phase 1b
- Day: Day 18
- Chat: Chat44
- Portal: Company
- Driver Portal: complete, accepted, locked
- Company Blueprint: locked
- Company existing-state inspection: completed
- Company Gap Map: completed as the reasoning input
- Company Implementation Boundary: proposed / ready for lock

## Primary Review Question

Determine whether the proposed Company Implementation Boundary is **complete, evidence-based, internally consistent, and safe to lock** when compared against:

1. The actual Company existing-state implementation inspection.
2. The locked Company Blueprint.
3. The approved implementation-preparation scope.
4. The protected Phase 1b boundaries.

The reviewer must specifically determine whether the boundary correctly limits Company implementation to frontend work using existing backend/data capabilities.

## Governing Records to Review

Review these Records repo documents:

1. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Existing_State_Implementation_Inspection_Report.md`
2. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
3. `03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md`
4. `00_PROJECT_CONTROL/DECISIONS/Chat42_Day16_Node7_Phase1b_Implementation_Boundary_Decision.md`
5. `00_PROJECT_CONTROL/DECISIONS/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary.md`

Historical Company investigation records may be consulted when useful, but current source inspection and locked decisions take precedence.

## Review Rules

### 1. Do not reopen locked product decisions

Do not redesign or reinterpret the Company Blueprint.

If the proposed boundary appears inconsistent with a locked Blueprint decision, identify the exact inconsistency and supporting evidence. Do not independently replace the locked decision.

### 2. Evidence discipline

For every material concern, classify it:

- **VERIFIED** — directly supported by the reviewed Records/source evidence.
- **INFERRED** — reasonable conclusion but not directly proven.
- **UNKNOWN** — insufficient evidence.

Do not invent missing implementation facts.

### 3. Boundary discipline

The review must distinguish:

- frontend work that is genuinely allowed;
- existing functionality that must be preserved;
- backend/API/database/security/business behavior that must remain protected;
- situations that require STOP & ESCALATE.

## Required Review Areas

### A. Existing State → Boundary Coverage

Check whether every major current-state finding from the Chat44 Company inspection is correctly represented in the proposed boundary.

Specifically review:
- Company navigation
- Dashboard
- My Created Trips / sender visibility
- Incoming Deliveries
- Receiver Check-in
- Receiver Completion
- Unified Trip Detail
- History / Timeline
- Profile / Account
- Public Share
- Responsive/mobile behavior
- shared vs Company-specific components
- frontend → API/data dependencies
- current structural defects

Identify any gap that is missing from the boundary.

### B. Blueprint → Boundary Coverage

Check whether every locked Company Blueprint requirement is either:
- explicitly inside the frontend implementation scope;
- explicitly preserved;
- explicitly protected; or
- covered by a stop/escalation condition.

Pay particular attention to:
- unified Company navigation
- Needs Attention dashboard
- My Created Trips
- Incoming Deliveries as receiver action inbox
- unified Trip Detail
- History/Timeline
- Profile/Account
- Public Share receiver-only constraint
- responsive/mobile requirements
- consistent entry paths into Trip Detail
- state-driven receiver task behavior

### C. Frontend-only Boundary

Determine whether the proposed boundary accidentally permits any work involving:
- new APIs
- API response changes
- database/schema changes
- RLS/security changes
- authentication changes
- authorization policy changes
- role assignment changes
- lifecycle/business-rule changes
- claiming/marketplace changes
- evidence integrity changes
- AI behavior
- Reviewer authority

If any protected work is accidentally allowed, flag it.

### D. Existing Backend/Data Capability

Verify that the boundary appropriately permits the frontend to **consume existing capabilities** without expanding backend behavior.

Pay particular attention to the `company_id` sender-trip data finding and whether the proposed boundary correctly distinguishes:

> existing data capability not surfaced by frontend

from:

> new backend capability required.

### E. C-05 Receiver Completion

Review the treatment of the Receiver Completion response-shape mismatch.

Confirm that:
- the API contract remains protected;
- frontend adaptation is allowed only if it can work with the established contract;
- backend response-shape modification is not permitted;
- inability to safely adapt must trigger STOP & ESCALATE.

### F. Shared Component / Driver Protection

Review whether the boundary sufficiently protects the already accepted and locked Driver Portal.

Check:
- shared Navbar
- shared authenticated layout
- other shared presentation components
- possibility of Company changes causing Driver regressions

Company work must not reopen or regress Driver decisions.

### G. Stop & Escalate Completeness

Determine whether the stop conditions are sufficient to prevent scope creep.

Look for missing conditions involving:
- API/data mismatch
- security/authorization ambiguity
- backend dependency
- shared-component risk
- locked-blueprint contradiction
- unknown behavior
- new product behavior

### H. Verification / Acceptance Gate

Check whether the boundary correctly requires:
- implementation evidence
- build/test evidence
- responsive/mobile evidence
- Company navigation/state evidence
- protected-area integrity
- Driver regression protection
- final STOP for Ayush manual browser verification
- no Company acceptance/lock without Ayush approval

## Required Review Matrix

Produce this matrix:

| Review Area | Boundary Coverage | Evidence | Finding | Severity | Required Change? |
|---|---|---|---|---|---|

Severity:
- BLOCKER
- HIGH
- MEDIUM
- LOW
- NONE

## Required Findings

Explicitly answer:

1. What is correct in the proposed boundary?
2. What is missing?
3. What is too broad?
4. What is incorrectly protected?
5. What frontend work should be added?
6. What work must be removed from the allowed scope?
7. Are the stop conditions sufficient?
8. Is C-05 handled correctly?
9. Is Driver protection sufficient?
10. Can the boundary safely become LOCKED after the review?

## Final Recommendation

Choose exactly one:

### APPROVE

The boundary is sufficiently complete and safe to lock. No material change is required.

### APPROVE WITH CHANGES

The boundary is fundamentally correct but requires specific changes before locking. List the exact changes.

### REJECT

The boundary has a material scope/evidence problem and should not be locked. Explain why and identify what must be reconsidered.

Do not make the final lock decision on behalf of Ayush.

## Important Prohibition

Do NOT:
- modify source code;
- modify the locked Company Blueprint;
- modify the proposed boundary file;
- create an implementation prompt;
- authorize Antigravity to implement;
- introduce backend/API/DB/security/business-rule work;
- reopen Driver implementation.

This review exists solely to provide an independent confidence checkpoint before the Company Implementation Boundary is locked.

## Required Output

Write the completed peer-review report to:

`05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary_Claude_Review_Report.md`

The report must be evidence-backed and must clearly separate VERIFIED, INFERRED, and UNKNOWN findings.
