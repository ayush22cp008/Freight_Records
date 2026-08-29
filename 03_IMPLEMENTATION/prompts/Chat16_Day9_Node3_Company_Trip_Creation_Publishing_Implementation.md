# Chat16 — Day 9 — Node 3 Company Trip Creation + Publishing — Antigravity Implementation Instruction

## Execution Status

**APPROVED FOR IMPLEMENTATION BY AYUSH**

Execute the corrected Node 3 implementation plan referenced below.

This is an implementation instruction. Unlike the plan, this file authorizes source/database changes within the exact scope below.

## 1. Authoritative Records

Read these Records before changing anything:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md
03_IMPLEMENTATION/implementation_reports/Chat16_Day9_Node3_Current_Source_Investigation_Report.md
03_IMPLEMENTATION/plans/Chat16_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Plan.md
01_BRAIN_HANDOFFS/Claude/Chat17_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Plan_claude_review.md
```

Application source baseline from the investigation:

```text
Repository: ayush22cp008/freight_hackathon
Branch: main
Investigated commit: c1df4a99ae84dd04fdf1254628d23c1c0d1a0b11
```

Before implementation, inspect the actual current source state. If it materially differs from the investigated baseline, stop and report the difference before proceeding.

## 2. Objective

Implement **Node 3 — Company Trip Creation + Publishing**.

Target flow:

```text
Verified Company
      ↓
Company Portal
      ↓
Create Trip
      ↓
Validate trip data + authorization
      ↓
Create DRAFT trip with no driver assigned
      ↓
Review trip
      ↓
Publish
      ↓
PUBLISHED / AVAILABLE to eligible drivers
```

Node 3 ends at publication/availability.

## 3. Mandatory Architecture Decisions

### Company relationships

Use the existing `companies` entity.

Creator/sending Company:

```text
trips.company_id → companies(id)
```

Receiving Company:

```text
trips.receiving_company_id → companies(id)
```

Do not use `freight_identities(id)` as the trip Company FK.

### Identity

Reuse the completed Node 2 identity/authentication path.

The acting Company must be derived server-side from the authenticated identity.

Never trust a client-provided Company ID as proof of ownership.

### Driver

Trips created/published in Node 3 have no assigned driver.

Make the existing trip driver relationship nullable safely.

Do not implement driver marketplace, acceptance, atomic claim, or claim locking.

### Status

Node 3 lifecycle:

```text
DRAFT → PUBLISHED / AVAILABLE
```

Preserve existing valid historical `active` trips. Do not silently rename or delete them.

Before adding a status constraint, inspect all existing status values and establish a legal post-migration set that preserves valid historical data and supports the Node 3 lifecycle.

### Distance/duration

Distance and duration are **manual Company-entered values** for Node 3.

Do not add maps, geocoding, routing, or external distance APIs.

### Offer

Store the Company's initial offer/payment value.

Manual adjustment is allowed.

Automated pricing is deferred.

## 4. Workstream A — Database Migration

Create a safe migration that:

```text
- makes trips.driver_id nullable;
- adds creator/sending Company relationship to companies(id);
- adds receiving Company relationship to companies(id);
- adds required Node 3 trip fields;
- stores initial offer/payment;
- supports DRAFT and PUBLISHED states;
- preserves historical active rows;
- preserves existing trip/event data;
- preserves compatibility with current driver/event consumers.
```

Immediately before migration, inspect:

```text
- existing trip rows
- existing status values
- existing driver references
- existing event references
- constraints
- indexes
- RLS policies
- migration order
```

Do not delete/reset production data.

If safe migration requires a material data-loss or architecture decision, stop and report it.

## 5. Workstream B — Receiving Company Lookup

Implement the smallest safe mechanism that allows a verified Company to select a receiving Company.

Requirements:

```text
- server-authorized;
- minimal Company fields only;
- no unrestricted Company directory exposure;
- valid companies(id) FK relationship;
- supports the allowed same-company sender/receiver case.
```

Do not weaken the general Company profile RLS policy with an unrestricted SELECT policy merely to implement the picker.

## 6. Workstream C — Company Create Trip

Implement a server-side Company-only create operation.

It must:

```text
- require authenticated session;
- resolve authenticated identity server-side;
- require verified Company role/state according to Node 2;
- derive creator Company server-side;
- validate all required trip fields server-side;
- validate receiving Company server-side;
- prevent client control of ownership fields;
- create the trip in DRAFT/pre-publication state;
- leave driver unassigned.
```

Do not trust client-provided `company_id`, `driver_id`, or equivalent ownership identifiers.

## 7. Workstream D — Company Publish Trip

Implement a server-side publish operation.

It must:

```text
- require authenticated verified Company;
- load the trip server-side;
- verify the acting Company owns/is authorized for the trip;
- verify current status is publishable;
- perform only the intended DRAFT → PUBLISHED transition;
- reject attempts to publish another Company's trip;
- preserve driver_id as unassigned;
- not trust client-supplied Company ownership identifiers.
```

**Security warning:** do not copy the existing events API pattern where a client-supplied `trip_id` is used without verifying the relevant relationship. Node 3 create/publish operations must verify ownership server-side.

If service-role/privileged database access is used, perform explicit application-level authorization; RLS alone is not sufficient proof of authorization.

## 8. Workstream E — Company UI

Add the Company trip-management path to the existing authenticated Company experience.

Minimum UI:

```text
- Create Trip entry point
- pickup details
- destination details
- receiving Company selection
- manually entered distance
- manually entered duration
- shipment/trip details
- offer/payment
- validation
- save/create
- trip details display
- publish action
- resulting PUBLISHED state display
```

Do not add driver claim/acceptance UI.

## 9. Workstream F — Compatibility

Verify existing driver/event/timeline behavior remains functional when a trip has no driver.

Do not redesign the event system.

Do not alter Node 1 or Node 2 architecture.

## 10. Required Tests

Add/update targeted automated tests for:

```text
- valid Company trip creation;
- invalid/missing required data;
- unauthenticated create rejection;
- non-Company create rejection;
- unverified Company rejection where applicable;
- creator ownership derived from authenticated identity;
- receiving Company lookup authorization;
- valid receiver relationship;
- publishing own DRAFT trip;
- publishing another Company's trip rejected;
- invalid/non-publishable state rejected;
- Company A cannot read Company B's protected trip by direct trip ID;
- driver remains unassigned after creation/publication;
- existing driver/event behavior remains compatible.
```

Run the project's appropriate:

```text
- typecheck/build
- lint where configured
- relevant tests
```

Record exact commands and results.

## 11. Manual Verification — Do Not Perform on Behalf of Ayush

After automated verification, provide the exact manual verification steps for Ayush.

Ayush must verify:

```text
1. Log in as verified Company A.
2. Open Company portal.
3. Create a valid trip.
4. Select a valid receiving Company.
5. Confirm sender/creator Company.
6. Confirm receiver relationship.
7. Confirm pickup/destination.
8. Confirm distance/duration.
9. Confirm initial offer.
10. Confirm DRAFT state.
11. Publish.
12. Confirm PUBLISHED/AVAILABLE state.
13. Confirm eligible driver can see the published opportunity as required.
14. Confirm Company B cannot publish Company A's trip.
15. Confirm Company B cannot directly read Company A's protected trip.
```

Do not mark manual verification as complete unless Ayush performs it.

## 12. Explicit Non-Goals

Do NOT implement:

```text
- Driver marketplace
- Driver acceptance
- Atomic first-valid claim
- Claim locking
- Full delivery lifecycle
- Pickup/delivery event redesign
- AI changes
- Automated pricing
- Automated routing/geocoding
- Broad security hardening outside Node 3
- Reopening Node 1 decisions
- Reopening Node 2 decisions
```

## 13. Stop Conditions

Stop and report before continuing if:

```text
- current source differs materially from the investigated baseline;
- companies(id) cannot safely support the required relationships;
- existing trip data cannot be migrated safely;
- a Node 1/Node 2 locked decision must be reopened;
- a major unexpected architecture blocker appears;
- implementing the receiving-company lookup requires unsafe broad data exposure.
```

Do not invent a workaround.

## 14. Implementation Evidence Report

After implementation and automated verification, create/update the implementation report in Records:

```text
03_IMPLEMENTATION/implementation_reports/Chat16_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Report.md
```

The report must include:

```text
- source commit before implementation;
- source commit after implementation if available;
- exact files changed;
- exact migrations created;
- schema changes;
- API/server changes;
- UI changes;
- authorization behavior;
- receiving-company lookup design;
- status migration/backward compatibility behavior;
- tests added/updated;
- exact verification commands and results;
- any deviations from the approved plan;
- any unresolved issues;
- VERIFIED / INFERRED / UNKNOWN classification;
- explicit statement that Ayush manual verification has NOT been performed by Antigravity.
```

Do not create a Node 3 completion checkpoint yourself unless the existing project workflow explicitly requires it; report completion evidence first for ChatGPT/Ayush review.

## 15. Final Antigravity Response

Return a concise summary:

```text
NODE 3 IMPLEMENTATION COMPLETE / BLOCKED

Plan executed:
03_IMPLEMENTATION/plans/Chat16_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Plan.md

Implementation report:
03_IMPLEMENTATION/implementation_reports/Chat16_Day9_Node3_Company_Trip_Creation_Publishing_Implementation_Report.md

Source commit before:
<sha>

Source commit after:
<sha>

Files changed:
<short list/count>

Migration:
<path/name>

Build/typecheck/lint/tests:
<PASS/FAIL + exact summary>

Major blocker:
YES / NO

Ayush manual verification:
NOT PERFORMED
```
