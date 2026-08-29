# Chat16 — Day 9 — Node 3 Company Trip Creation + Publishing Implementation Plan

## Status

**PLAN — REVISED AFTER CLAUDE REVIEW**

This is the corrected Chat16 plan. The earlier `Chat17_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Plan.md` filename was a numbering error in the Records workflow; this Chat16 file is the authoritative corrected plan for this conversation/checkpoint.

## 1. Purpose

Define the implementation plan for **Node 3 — Company Trip Creation + Publishing** based on the completed current-source investigation and independent Claude architecture review.

This is a **plan only**. It is not an Antigravity implementation instruction.

## 2. Authoritative Inputs

Records reviewed:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md
03_IMPLEMENTATION/implementation_reports/Chat16_Day9_Node3_Current_Source_Investigation_Report.md
01_BRAIN_HANDOFFS/Claude/Chat17_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Plan_claude_review.md
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
Trip becomes available to eligible drivers
```

Node 3 ends at publication/availability. Driver acceptance, atomic claim, and delivery execution remain later Nodes.

## 4. Current State Confirmed

Existing and reusable:

```text
- Supabase authentication/session foundation
- freight_identities role/verification foundation
- Authenticated App Router structure
- Supabase server client infrastructure
- Existing companies table
- Existing trips table
- Existing driver/event/timeline consumers
```

Missing:

```text
- Company-owned trip relationship on trips
- Receiving-company relationship on trips
- Node 3 trip detail fields
- Pre-driver-assignment trip state
- Company Create Trip backend operation
- Company Publish backend operation
- Company trip creation UI
- Company publishing UI
- Explicit server-side authorization for these Company actions
- Safe receiving-company lookup/selection mechanism
```

The current `trips.driver_id` is `NOT NULL` and reflects the historical driver-first MVP model. Node 3 requires a trip to exist before a driver is assigned.

## 5. Decisions Locked Before Implementation

### 5.1 Identity

Reuse the completed Node 2 identity/authentication path.

Do not create a second authentication or identity mechanism.

The acting Company must be derived from the authenticated server-side identity, not trusted from a client-supplied company ID.

### 5.2 Trip creator/sending Company relationship

The existing `companies` table is the Company entity for Node 3.

The Node 3 trip creator/sending-company relationship must reference the Company entity:

```text
trips.company_id → companies(id)
```

Do not reference `freight_identities(id)` for the trip Company foreign key.

The exact existing column/type definitions must be verified immediately before migration and implementation.

### 5.3 Receiving Company relationship

The receiving Company is also represented by the existing `companies` table:

```text
trips.receiving_company_id → companies(id)
```

Node 3 must preserve a real Company-to-Company relationship rather than silently replacing the locked relationship model with arbitrary free text.

### 5.4 Receiving-company lookup/selection

The current Company RLS model does not permit a Company to browse every other Company record. Therefore implementation must explicitly establish a **minimal, authorization-safe receiving-company lookup mechanism**.

The lookup must:

```text
- expose only the minimum fields needed for trip creation;
- not expose sensitive Company data;
- not weaken broad Company profile RLS unnecessarily;
- be server-authorized;
- allow selection of another valid Company;
- support sending Company = receiving Company if the locked product model permits it.
```

A dedicated server-side lookup endpoint/action or narrowly scoped database access policy is acceptable if it preserves the security boundary.

Do not improvise an unrestricted `SELECT * FROM companies` policy.

### 5.5 Driver assignment

The driver remains unassigned at creation/publication time.

The existing trip `driver_id` relationship must therefore support a null/unassigned state without breaking existing driver/event consumers.

Do not implement marketplace or claim logic in Node 3.

### 5.6 Trip status

Node 1 requires the pre-claim lifecycle to support:

```text
DRAFT → PUBLISHED / AVAILABLE
```

The existing historical `active` status must be preserved as a legal value for existing MVP trips unless current data/source inspection proves a safe explicit migration of those rows.

Before migration, enumerate the exact post-migration legal status values and their meanings.

The implementation should add database-level state validation where safely compatible with existing rows. The constraint must preserve existing valid `active` rows while preventing arbitrary invalid status values.

Do not silently rename existing `active` rows.

Do not implement CLAIMED/IN_PROGRESS/DELIVERED behavior as Node 3 work.

### 5.7 Distance and duration

For Node 3, **distance and duration are manually entered Company trip fields**.

Do not introduce maps, geocoding, routing, external distance APIs, or automated calculation unless an existing project dependency already provides this and the implementation plan is explicitly amended.

### 5.8 Offer/payment

Store the Company's initial offer/payment value.

Manual Company offer adjustment is allowed by the roadmap.

Automated pricing is explicitly deferred.

## 6. Implementation Workstreams

### Workstream A — Database/schema migration

Create a safe migration that evolves the historical `trips` model for Node 3.

Required outcomes:

```text
- Allow trip creation without an assigned driver.
- Add creator/sending Company relationship referencing companies(id).
- Add receiving Company relationship referencing companies(id).
- Add required Node 3 trip detail fields.
- Add initial offer/payment representation.
- Establish DRAFT/PUBLISHED lifecycle while preserving valid historical active rows.
- Preserve existing trip/event data.
- Preserve compatibility with existing driver/event consumers.
```

Before applying constraints, inspect:

```text
- existing trip rows;
- existing status values;
- existing driver references;
- existing event references;
- existing indexes/constraints;
- existing RLS policies;
- migration order.
```

Do not delete existing trips merely to satisfy the new model.

If a migration requires a temporary/backfill/default strategy, document exactly what it does and why.

### Workstream B — Receiving Company lookup

Implement the smallest safe mechanism allowing a verified Company to select a valid receiving Company.

Requirements:

```text
- server-side authorization;
- minimal returned Company fields;
- no unrestricted Company-directory exposure;
- valid Company foreign-key relationship;
- safe handling of same-company sender/receiver where permitted.
```

The mechanism must not weaken the existing general Company profile RLS policy simply for convenience.

### Workstream C — Server-side authorization and API

Implement Company-only trip creation and publishing operations.

Creation must:

```text
- Require an authenticated session.
- Resolve the acting identity server-side.
- Require verified Company role/state according to the locked Node 2 model.
- Associate the new trip with that Company server-side.
- Validate required trip fields server-side.
- Validate receiving Company relationship server-side.
- Prevent client control of privileged ownership fields.
- Create the trip in DRAFT/pre-publication state.
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

**Security precedent:** do not copy the existing events API pattern where a client-supplied `trip_id` is trusted without verifying the relevant trip relationship. Node 3 must explicitly derive and verify Company ownership server-side.

### Workstream D — Company UI

Add a real Company trip-management path to the authenticated Company experience.

Minimum UI capabilities:

```text
- Start Create Trip
- Enter pickup details
- Enter destination details
- Select receiving Company through the safe lookup mechanism
- Enter distance manually
- Enter duration manually
- Enter shipment/trip details
- Enter offer/payment
- Validate input
- Save/create trip
- Show created trip details
- Publish trip
- Show resulting published state
```

The UI must not expose driver acceptance/claim controls as Node 3 functionality.

### Workstream E — Compatibility

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
- receiving Company lookup authorization
- valid receiver relationship
- publishing own draft trip
- publishing another Company's trip rejected
- publishing invalid/non-publishable state rejected
- Company A cannot read Company B's trip by direct trip ID
- driver remains unassigned after creation/publication
- existing driver/event behavior remains compatible
```

### Manual — Ayush gate

After implementation evidence is complete, Ayush must manually verify:

```text
1. Log in as verified Company A.
2. Open Company portal.
3. Create a valid trip.
4. Select a valid receiving Company.
5. Confirm trip details are displayed correctly.
6. Confirm sender/creator Company is correct.
7. Confirm receiver relationship is correct.
8. Confirm initial offer is correct.
9. Confirm distance/duration values are correct.
10. Confirm trip starts in DRAFT/pre-publication state.
11. Publish the trip.
12. Confirm PUBLISHED/AVAILABLE state.
13. Confirm an eligible driver can see the published opportunity as required.
14. Confirm Company B cannot publish Company A's trip.
15. Confirm Company B cannot directly read Company A's protected draft trip.
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
- Receiving-company lookup
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
- Automated routing/geocoding
- Broad security hardening outside Node 3
- Reopening Node 1 authorization decisions
- Reopening Node 2 identity decisions
```

## 10. Migration Safety Gate

Before modifying the schema, implementation must verify:

```text
- Current trips data shape
- Existing rows
- Existing status values
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
[ ] Trip is associated with the creating/sending Company via companies(id).
[ ] Receiver relationship is stored correctly via companies(id).
[ ] Safe receiving-company lookup exists.
[ ] Pickup/destination details are stored.
[ ] Distance is stored as manually entered data.
[ ] Duration is stored as manually entered data.
[ ] Initial offer/payment is stored.
[ ] Trip can exist without an assigned driver.
[ ] DRAFT/pre-publication state works.
[ ] Existing valid active trips remain compatible.
[ ] Company can publish its own trip.
[ ] Published trip is AVAILABLE to eligible drivers according to the Node 1 rule.
[ ] Unauthorized Company cannot publish another Company's trip.
[ ] Company cannot directly read another Company's protected trip.
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
2. Confirm companies(id) and exact current schema.
3. Implement safe trips schema migration.
4. Verify migration/build state.
5. Implement receiving-company lookup with minimal exposure.
6. Implement server-side Company create operation.
7. Implement publish operation with ownership/state checks.
8. Add Company Create Trip UI.
9. Add Company publish UI.
10. Add/update targeted tests.
11. Run build/typecheck/lint/tests.
12. Produce implementation evidence report.
13. Ayush performs manual verification.
14. Resolve any manual findings.
15. Create/update Node 3 completion checkpoint only after acceptance evidence.
```

## 13. Risk Classification

The current investigation identified no unexpected or major architecture blocker.

The historical `driver_id NOT NULL` constraint is **known Node 3 migration work**, not a Subnode by itself.

The existing Company lookup/RLS limitation is a known Node 3 design requirement identified during review; it must be solved explicitly within Node 3 and is not a reason to create a Subnode unless implementation reveals a genuinely unexpected architecture problem.

If implementation discovers a genuinely unexpected major architecture issue, stop before bypassing the locked model and report it for reassessment.

## 14. Review Outcome

Claude independently reviewed the previous Node 3 plan and returned **APPROVE WITH CHANGES**. The accepted changes are incorporated into this corrected Chat16 plan:

```text
1. companies(id) is the explicit FK target for creator/sending and receiving Company.
2. Receiving-company lookup must be explicitly designed and security-scoped.
3. Existing active status must be preserved for compatibility and the legal status set must be explicit.
4. Distance and duration are manual inputs for Node 3.
5. Cross-company trip read protection is an explicit acceptance/test requirement.
6. New APIs must not copy the existing client-supplied-trip-ID authorization weakness.
```

## 15. Next Step

This corrected plan is ready for final review/approval.

It must NOT be treated as Antigravity execution instructions until the plan is approved and a separate implementation prompt is created in:

```text
03_IMPLEMENTATION/prompts/
```

The implementation prompt should reference this corrected Chat16 plan and the final investigation report rather than duplicating unsupported assumptions.
