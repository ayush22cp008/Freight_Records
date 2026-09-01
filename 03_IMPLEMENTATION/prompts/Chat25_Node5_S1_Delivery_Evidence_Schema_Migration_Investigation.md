# Chat25 — Node 5 Subnode 5.S1 — Delivery Evidence Schema Migration Investigation / Design

## Role
You are Antigravity, the implementation/execution agent. This is an INVESTIGATION + DESIGN ONLY task. Do not modify application source code, database schema, configuration, tests, or shared/production data.

## Project
- Project: Freight — AI Builders Hackathon
- Source repository: `ayush22cp008/freight_hackathon`
- Records repository: `ayush22cp008/Freight_Records`
- Hackathon day: **12**
- Chat: **25**
- Parent Node: **Node 5 — Whole Delivery Tracking**
- Subnode: **5.S1 — Delivery Evidence Schema Migration**

## Why this task exists
Chat24 established that the existing `events` schema cannot directly support the expanded Node 5 lifecycle. The prior Chat24 S1 prompt exists, but the required design report was not produced in the Records repo. Chat25 must complete the investigation/design evidence before any migration implementation is considered.

This is not permission to implement the migration.

## Governing Records
Read before investigation:
1. `00_PROJECT_CONTROL/ROADMAP.md`
2. `00_PROJECT_CONTROL/CURRENT_STATUS.md`
3. `00_PROJECT_CONTROL/PROJECT_STATE.md`
4. `00_PROJECT_CONTROL/MASTER_ARCHITECTURE.md`
5. `01_BRAIN_HANDOFFS/ChatGPT/Chat10_Node1_FINAL_LOCK.md`
6. `01_BRAIN_HANDOFFS/Claude/Node 5 Architecture Consistency Review — Chat24_Node5_Architecture_Decisions.md`
7. `02_ARCHITECTURE/locked_decisions/Chat24_Node5_Architecture_Decisions.md`
8. `02_ARCHITECTURE/locked_decisions/Chat4_Node3_Decision_EventsTableSchema.md`
9. `03_IMPLEMENTATION/implementation_reports/Chat24_Node5_Current_Source_Investigation_Report.md`
10. Relevant Node 3/4 implementation reports affecting `trips` or `events`.

If a referenced path is absent, record `UNKNOWN`; do not invent a replacement. Report contradictions as `CONFLICT` with exact paths/evidence.

## Chat25 locked decisions to apply
### Q3 — Event vocabulary
Ayush selected **Option A**. New Node 5 persisted lifecycle event names must exactly follow the Node 1 FINAL LOCK:

```text
ARRIVED_AT_PICKUP
PICKUP_CHECKED_IN
GOODS_LOADED
PICKUP_DEPARTED
IN_TRANSIT
ARRIVED_AT_DELIVERY
RECEIVER_CHECKED_IN
GOODS_UNLOADED
DELIVERY_DEPARTED
```

Historical Core MVP values remain unchanged:

```text
arrival
checkin
departure
```

Do not rename historical records. Do not implement the earlier Chat24 lowercase proposal.

### Q4 — Uniqueness
Ayush approved **Option 1**:

- retain database-level duplicate protection equivalent to `UNIQUE (trip_id, event_type)` for canonical single-delivery lifecycle events;
- canonical lifecycle milestones are single-occurrence within the single-delivery scope;
- if deferred/stretch Repeatable Add Evidence is implemented later, it must use a separate repeatable evidence mechanism rather than weakening or duplicating canonical lifecycle event types.

Q3/Q4 are conceptually resolved, but this S1 investigation must verify their compatibility with the actual source/schema and document any remaining issue.

## Locked lifecycle
The schema must eventually support:

```text
PICKUP
→ ARRIVED_AT_PICKUP
→ PICKUP_CHECKED_IN
→ GOODS_LOADED
→ PICKUP_DEPARTED
→ IN_TRANSIT
→ ARRIVED_AT_DELIVERY
→ RECEIVER_CHECKED_IN
→ GOODS_UNLOADED
→ DELIVERY_DEPARTED
→ DRIVER_COMPLETION_CONFIRMED
→ RECEIVER_DELIVERY_CONFIRMED
→ DELIVERED / COMPLETED
```

Final completion requires both Driver completion and Receiving Company confirmation and must remain atomic/concurrency-safe.

## Investigation / design questions
Establish evidence first. Do not guess.

1. **Current `events` schema:** inspect exact table definition, event type CHECK constraint, keys, indexes, `(trip_id,event_type)` uniqueness, timestamps, GPS/photo fields, RLS/policies, triggers/functions.
2. **Historical compatibility:** determine whether `arrival/checkin/departure` can remain unchanged and identify all source consumers of those values. Do not mutate data.
3. **Uniqueness:** verify that the approved Q4 model safely protects every canonical Node 5 milestone. Explicitly distinguish canonical one-time milestones from deferred repeatable evidence.
4. **Event vocabulary:** verify the exact Node 1 names above are compatible with the current schema and application consumers. Do not reopen Q3 unless evidence shows a genuine contradiction.
5. **Trip status:** inspect actual `trips.status` constraints and determine the smallest required change for the locked lifecycle. Do not add `in_transit` merely because it is a physical milestone; preserve the Node 1 major state model unless evidence requires otherwise.
6. **State vs events:** determine what belongs in `trips.status`, immutable `events`, and final confirmation fields/relationships without creating conflicting duplicate sources of truth.
7. **RLS/security:** inspect compatibility only; do not weaken or casually reopen the CLOSED/VERIFIED RLS decision. Identify application-level authorization work separately for later Node 5/6 implementation.
8. **Migration safety:** design a conceptual migration sequence covering constraints, indexes, data preservation, deployment ordering, rollback, and validation. Do not execute SQL or modify database state.
9. **Application compatibility:** identify consumers of event types, statuses, timeline queries, insertion helpers, and uniqueness assumptions; classify impact.
10. **Future testing:** define validation required after implementation, including historical preservation, canonical duplicate protection, lifecycle ordering, RLS preservation, and compatibility.

## Important evidence/path issue to resolve
The existing Chat24 architecture record cited:

`05_DEBUGGING/investigations/Chat24_Node5_S1_Delivery_Evidence_Schema_Migration_Design_Report.md`

Claude's review reported that this design report was missing. The current Chat24 source investigation report is actually recorded at:

`03_IMPLEMENTATION/implementation_reports/Chat24_Node5_Current_Source_Investigation_Report.md`

Do not silently rename or overwrite historical records. Determine and document the exact current state of the expected S1 design report path.

## Source baseline discipline
The Chat24 current-source report contains an uncertain/assumed commit SHA. Do not reuse that SHA. Inspect the actual current source repository revision and record the exact branch, commit SHA, and working-tree status available to you.

## Scope boundary
This task is investigation/design only.

DO NOT:
- modify application source;
- modify database schema or migrations;
- create/apply migration SQL;
- change production/shared database state;
- add tests;
- implement Node 5 lifecycle;
- implement Receiver UI/API;
- implement completion logic;
- commit or push source changes.

## Required Records report
Create exactly one report:

`05_DEBUGGING/investigations/Chat25_Node5_S1_Delivery_Evidence_Schema_Migration_Design_Report.md`

Use these sections:

1. Subnode Status
2. Why 5.S1 Was Created
3. Records Baseline
4. Source Repository Baseline
5. Current `events` Schema
6. Current `trips` Status Schema
7. Existing Data Compatibility
8. Event Uniqueness Analysis
9. Event Vocabulary Analysis
10. Proposed Target Schema Contract
11. State vs Event Responsibility
12. RLS / Authorization Compatibility
13. Migration Sequence Design
14. Backward Compatibility / Historical Data Preservation
15. Future Validation / Test Requirements
16. Risks and Rollback Considerations
17. Q3/Q4 Resolution Compatibility
18. Evidence / Path Issue Resolution
19. Exact Decisions Required Before Implementation
20. Subnode Exit Criteria
21. Evidence Index
22. VERIFIED / INFERRED / UNKNOWN Summary
23. Explicit Non-Changes

### Target schema contract
Describe the target contract precisely enough for ChatGPT to review, but do not write implementation SQL. Include required event types, uniqueness/duplicate strategy, status values, constraints/indexes, historical-data treatment, and compatibility requirements.

### Decision discipline
Use exactly:
- `VERIFIED` — directly supported by current source, migration, command output, or existing evidence.
- `INFERRED` — reasonable conclusion not directly confirmed.
- `UNKNOWN` — insufficient evidence.

Never promote INFERRED/UNKNOWN to VERIFIED.

## Exit criteria
5.S1 is ready for ChatGPT review only when:
- current schema is directly evidenced;
- incompatibilities are directly evidenced;
- Q3 exact naming is respected;
- Q4 uniqueness strategy is explicitly analyzed and supported;
- target schema contract is documented;
- historical preservation is explicit;
- migration safety/order is explicit;
- compatibility impact is explicit;
- the report/path issue is resolved or explicitly marked UNKNOWN with evidence;
- no unresolved major architecture conflict remains.

## Final response to ChatGPT
Return only:

```text
5.S1 INVESTIGATION/DESIGN COMPLETE

Report:
05_DEBUGGING/investigations/Chat25_Node5_S1_Delivery_Evidence_Schema_Migration_Design_Report.md

Source commit inspected:
<exact SHA>

Branch:
<exact branch>

Design status:
READY FOR CHATGPT REVIEW / BLOCKED

Major conflict:
YES / NO

Implementation performed:
NO

Source changes:
NONE

Database changes:
NONE
```

Do not paste the report contents or source code into chat.
