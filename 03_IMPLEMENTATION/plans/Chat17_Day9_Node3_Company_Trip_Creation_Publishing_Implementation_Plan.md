# Chat17 — Day 9 — Node 3 Company Trip Creation + Publishing Implementation Plan

## 1. Purpose

Define the implementation plan for **Node 3 — Company Trip Creation + Publishing** based on the completed current-source investigation.

This is a **plan only**. It is not an Antigravity implementation instruction.

## 2. Authoritative Inputs

Records reviewed:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md
03_IMPLEMENTATION/implementation_reports/Chat16_Day9_Node3_Current_Source_Investigation_Report.md
```

Current source baseline inspected by Antigravity:

```text
Repository: ayush22cp008/freight_hackathon
Branch: main
Commit: c1df4a99ae84dd04fdf1254628d23c1c0d1a0b11
```

## 3. Node 3 Objective

Implement the company-owned trip creation and publishing foundation required for the later Driver Marketplace Node.

Target flow:

```text
Verified Company
      ↓
Company Portal
      ↓
Create Trip
      ↓
Validate trip data + company authorization
      ↓
Persist trip in pre-driver-assignment state
      ↓
Company reviews trip
      ↓
Publish
      ↓
Trip becomes available according to eligibility rules
```

Node 3 ends at publication/availability. Driver acceptance, atomic claim, and delivery execution remain later Nodes.

## 4. Current State Confirmed

Existing and reusable:

```text
- Supabase authentication/session foundation
- freight_identities role/verification foundation
- Authenticated App Router structure
- Supabase server client infrastructure
- Existing trips table
- Existing driver/event/timeline consumers
```

Missing:

```text
- Company-owned trip relationship
- Receiving-company relationship
- Node 3 trip detail fields
- Pre-driver-assignment trip state
- Company Create Trip backend operation
- Company Publish backend operation
- Company trip creation UI
- Company publishing UI
- Explicit server-side authorization for these Company actions
```

The current `trips.driver_id` is NOT NULL and reflects the historical driver-first MVP model. Node 3 requires a trip to exist before a driver is assigned.

## 5. Architecture Decisions / Guardrails

### 5.1 Identity

Reuse the completed Node 2 identity/authentication path.

Do not create a second authentication or identity mechanism.

The acting Company must be derived from the authenticated server-side identity, not trusted from a client-supplied company ID.

### 5.2 Trip ownership

A trip must be associated with its creating/sending Company.

The exact final column name must follow existing naming conventions and migration evidence. `company_id` is an implementation candidate from the investigation, not a previously locked column name.

### 5.3 Receiving Company

A trip must preserve the receiving-company relationship required by Node 3.

Do not use only free-form receiver text if an existing Company relationship can be represented safely. If the existing product/data model does not establish the exact representation, document the choice in implementation evidence rather than silently inventing a separate company model.

### 5.4 Driver assignment

The driver remains unassigned at creation/publication time.

The existing driver relationship must therefore support a null/unassigned state without breaking existing driver/event consumers.

Do not implement marketplace or claim logic in Node 3.

### 5.5 Lifecycle

Node 3 must establish the pre-claim lifecycle needed by later Nodes.

At minimum:

```text
DRAFT → PUBLISHED / AVAILABLE
```

The exact status representation must use the safest schema compatible with the current source and future Node 4/5 lifecycle.

Do not implement CLAIMED/IN_PROGRESS/DELIVERED behavior as part of Node 3 except where existing code requires preserving compatibility.

### 5.6 Offer

Store the Company's initial trip offer/payment value.

Manual Company adjustment is allowed by the roadmap.

Automated pricing is explicitly deferred.

## 6. Implementation Workstreams

### Workstream A — Database/schema migration

Create a safe migration that evolves the historical `trips` model for Node 3.

Required outcomes:

```text
- Allow trip creation without an assigned driver.
- Add a creator/sending Company relationship.
- Add receiving Company relationship.
- Add required Node 3 trip detail fields.
- Add initial offer/payment representation.
- Establish a safe draft/published state representation.
- Preserve existing trip/event data.
- Preserve compatibility with existing driver/event consumers.
```

Before applying constraints, inspect existing data and migration ordering.

Do not delete existing trips merely to satisfy the new model.

If a migration requires a temporary/backfill/default strategy, document exactly what it does and why.

### Workstream B — Server-side authorization and API

Implement Company-only trip creation and publishing operations.

Creation must:

```text
- Require an authenticated session.
- Resolve the acting identity server-side.
- Require verified Company role/state according to the locked Node 2 model.
- Associate the new trip with that Company server-side.
- Validate required trip fields server-side.
- Prevent client control of privileged ownership fields.
- Create the trip in the correct pre-publication state.
```

Publishing must:

```text
- Require authenticated verified Company.
- Load the target trip server-side.
- Verify the acting Company owns/is authorized for the trip.
- Verify the current state permits publication.
- Transition only through the intended state transition.
- Prevent publishing another Company's trip.
- Do not accept arbitrary owner/company IDs from the client as authority.
```

Do not rely on RLS alone if a service-role/privileged client is used.

### Workstream C — Company UI

Add a real Company trip-management path to the authenticated Company experience.

Minimum UI capabilities:

```text
- Start Create Trip
- Enter required trip details
- Select/enter receiving Company according to the established model
- Enter offer/payment
- Validate input
- Save/create trip
- Show created trip details
- Publish trip
- Show resulting published state
```

The UI must not expose driver acceptance/claim controls as Node 3 functionality.

### Workstream D — Compatibility

Verify existing driver/event/timeline code remains functional when published/draft trips have no driver assigned.

Existing driver-specific queries should continue to scope to assigned drivers rather than assuming every trip has a driver.

Do not redesign the event system in Node 3.

## 7. Required Validation

### Automated

Run the project's appropriate:

```text
- typecheck/build
- lint where configured
- relevant tests
```

Also add/update targeted tests for:

```text
- valid Company trip creation
- missing/invalid required data
- unauthenticated create rejection
- non-Company create rejection
- unverified Company rejection where applicable
- ownership derived from authenticated Company
- publishing own draft trip
- publishing another Company's trip rejected
- publishing invalid/non-publishable state rejected
- driver remains unassigned after creation/publication
```

### Manual — Ayush gate

After implementation evidence is complete, Ayush must manually verify:

```text
1. Log in as verified Company.
2. Open Company portal.
3. Create a valid trip.
4. Confirm trip details are displayed correctly.
5. Confirm receiver relationship is correct.
6. Confirm initial offer is correct.
7. Confirm trip starts in the intended pre-publication state.
8. Publish the trip.
9. Confirm published state.
10. Confirm unauthorized Company behavior is rejected.
```

Do not mark Node 3 complete until this manual gate is performed and recorded.

## 8. Security Requirements

Node 3 implementation must explicitly protect:

```text
- Company identity
- Company ownership of created trips
- Receiver relationship
- Trip read/update authorization as applicable
- Publish authorization
- Client-supplied trip/company identifiers
```

The server must derive authorization from authenticated identity and server-side relationships.

Do not introduce an IDOR through a new trip API.

Do not weaken existing RLS/authentication.

## 9. Explicit Non-Goals

Do NOT implement:

```text
- Driver marketplace UI
- Driver acceptance
- Atomic first-valid claim
- Claim locking
- Full delivery lifecycle
- Pickup/delivery event redesign
- AI summary changes
- Automated pricing engine
- Broad security hardening outside Node 3
- Reopening Node 1 authorization decisions
- Reopening Node 2 identity decisions
```

## 10. Migration Safety Gate

Before modifying the schema, implementation must verify:

```text
- Current trips data shape
- Existing rows
- Existing driver/event references
- Existing constraints
- Existing RLS/policies
- Migration order
```

If safe migration cannot be established without a material data-loss or architecture decision, stop and report the blocker. Do not delete or reset data.

## 11. Definition of Done

Node 3 is complete only when all are true:

```text
[ ] Company can create a valid trip.
[ ] Trip is associated with the creating/sending Company.
[ ] Receiver relationship is stored correctly.
[ ] Pickup/destination details are stored.
[ ] Distance is stored if required by the chosen model.
[ ] Duration is stored.
[ ] Initial offer/payment is stored.
[ ] Trip can exist without an assigned driver.
[ ] Draft/pre-publication state works.
[ ] Company can publish its own trip.
[ ] Published trip is available according to eligibility rules.
[ ] Unauthorized Company cannot publish another Company's trip.
[ ] Server derives ownership from authenticated identity.
[ ] Existing driver/event behavior is not broken.
[ ] Automated verification passes.
[ ] Antigravity implementation report contains evidence.
[ ] Ayush manual verification passes.
[ ] Records checkpoint is updated.
[ ] Only then is the Node 3 checkpoint eligible for closure.
```

## 12. Expected Implementation Sequence

```text
1. Inspect current migration/data constraints immediately before implementation.
2. Implement safe schema migration.
3. Verify migration/build state.
4. Implement server-side Company create operation.
5. Implement publish operation with ownership/state checks.
6. Add Company Create Trip UI.
7. Add Company publish UI.
8. Add/update targeted tests.
9. Run build/typecheck/lint/tests.
10. Produce implementation evidence report.
11. Ayush performs manual verification.
12. Resolve any manual findings.
13. Create/update Node 3 completion checkpoint only after acceptance evidence.
```

## 13. Risk Classification

The current investigation identified no unexpected or major architecture blocker.

The historical `driver_id NOT NULL` constraint is **known Node 3 migration work**, not a Subnode by itself.

If implementation discovers a genuinely unexpected major architecture issue, stop before bypassing the locked model and report it for reassessment.

## 14. Next Step

This plan is ready for review/approval.

It must NOT be treated as Antigravity execution instructions until the plan is approved and a separate implementation prompt is created in:

```text
03_IMPLEMENTATION/prompts/
```

The implementation prompt should reference this plan and the final investigation report rather than duplicating unsupported assumptions.
