# Chat26 — Node 5 Subnode 5.S1 — Delivery Evidence Schema Migration Implementation

## Objective

Implement only the approved Node 5 Subnode 5.S1 database schema migration for the expanded Whole Delivery Tracking lifecycle.

This task is limited to the database migration/schema layer. Do not implement application APIs, UI, receiver workflow, timeline changes, or final completion logic yet.

## Authoritative Decisions

Use these Records as the governing baseline:

- `00_PROJECT_CONTROL/ROADMAP.md`
- `00_PROJECT_CONTROL/CHECKPOINTS/Chat25_Node5_Execution_Checkpoint.md`
- `02_ARCHITECTURE/locked_decisions/Chat24_Node5_Architecture_Decisions.md`
- `03_IMPLEMENTATION/implementation_reports/Chat25_Node5_S1_Delivery_Evidence_Schema_Migration_Design_Report.md`
- `05_DEBUGGING/investigations/Chat26_Node5_Current_Lifecycle_Investigation_Report.md`

The Chat25 S1 design is approved for implementation unless current source evidence reveals a direct incompatibility. If a material contradiction is found, stop and report it rather than inventing a new architecture.

## Required Schema Contract

### Events

Expand the existing `events.event_type` CHECK constraint so it accepts both the historical values and the canonical Node 5 values:

```text
arrival
checkin
departure
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

Preserve:

```text
UNIQUE (trip_id, event_type)
```

Do not weaken, remove, recreate unnecessarily, or otherwise alter this duplicate-protection strategy.

### Trips

Keep the existing major status vocabulary unchanged:

```text
active
draft
published
claimed
in_progress
completed
```

Do **not** add `in_transit` or `delivered` to `trips.status`.

Add nullable timestamp fields for final completion acknowledgements according to the approved design. Use the exact compatible column naming established by the current source/design evidence; if the source contains a material naming conflict with the approved architecture, stop and report it instead of silently choosing a new contract.

The architectural responsibility remains:

```text
trips.status
→ overall trip lifecycle

events
→ detailed physical delivery milestones/evidence

trips completion timestamps
→ final driver/receiver acknowledgements
```

## Migration Safety Requirements

Implement the migration as a safe transactional schema change where supported by the project's migration system.

Required behavior:

1. Preserve all existing rows.
2. Preserve all legacy `arrival`, `checkin`, and `departure` values.
3. Expand the event CHECK constraint to include the canonical Node 5 values.
4. Preserve `UNIQUE (trip_id, event_type)`.
5. Add only the approved nullable completion timestamp fields.
6. Do not rewrite historical event rows.
7. Do not delete or recreate tables unnecessarily.
8. Do not change existing RLS behavior unless the migration absolutely requires it; receiver authorization is a later application/security task.
9. Do not add application behavior in this task.

## Source/Application Boundary

This migration intentionally creates the schema foundation before application-layer lifecycle work.

Do not modify:

```text
API routes
Server actions
Frontend pages/components
Timeline UI
Dashboard UI
AI summary behavior
Receiver workflow
Authorization logic
Completion endpoint
```

Those will be handled in later Node 5 implementation steps.

## Required Pre-Implementation Checks

Before editing anything:

- Confirm actual source repository and branch.
- Inspect the current migration files and migration ordering.
- Confirm the current `events` constraint and uniqueness definition.
- Confirm the current `trips` definition.
- Confirm the current completion-field state.
- Check whether an equivalent migration already exists to avoid duplication.

If an equivalent migration already exists, do not create a duplicate migration. Report the finding and reconcile only if it is clearly safe and within this S1 scope.

## Verification Requirements

After implementation:

1. Run the project's appropriate migration/schema validation mechanism.
2. Verify the migration applies successfully.
3. Verify the resulting `events.event_type` constraint accepts all canonical values and legacy values.
4. Verify `UNIQUE (trip_id, event_type)` remains present.
5. Verify `trips.status` still contains only the existing major statuses.
6. Verify the completion timestamp columns exist and are nullable.
7. Verify no historical rows were rewritten or deleted.
8. Run relevant non-destructive build/type/test checks available for this schema change.
9. Report exact commands and results; never claim a test was run if it was not.

If database access is unavailable, do not fabricate runtime verification. Clearly mark the relevant result UNKNOWN and provide static migration evidence instead.

## Hard Boundaries

```text
Application source changes = NO
Frontend changes = NO
API changes = NO
Authorization changes = NO
Receiver workflow = NO
Timeline/dashboard changes = NO
AI changes = NO
Production/shared data mutation = NO
Destructive data operations = NO
Commit source repository = NO
Push source repository = NO
```

The migration itself is the only authorized implementation change.

## Required Implementation Report

Create exactly one report in the Records repository:

`03_IMPLEMENTATION/implementation_reports/Chat26_Node5_S1_Delivery_Evidence_Schema_Migration_Implementation_Report.md`

The report must include:

1. Implementation status.
2. Source repository, branch, commit, and working-tree evidence.
3. Exact migration file created/modified.
4. Before/after schema evidence.
5. Event vocabulary evidence.
6. Uniqueness evidence.
7. Trips status evidence.
8. Completion timestamp column evidence.
9. Migration execution/validation evidence.
10. Test/build/type-check evidence actually performed.
11. Historical-data preservation assessment.
12. Any blockers, deviations, or UNKNOWN items.
13. VERIFIED / INFERRED / UNKNOWN summary.
14. Explicit confirmation of what was not changed.

Do not create additional implementation prompts or unrelated Records files.

## Stop Condition

Stop and report to ChatGPT before proceeding if any of the following occurs:

- The approved schema contract conflicts materially with the current source.
- The migration system requires an architectural change outside S1.
- Existing data would need rewriting/deletion.
- `UNIQUE (trip_id, event_type)` cannot be preserved safely.
- Completion-field naming cannot be resolved from existing evidence.
- The change would require application/API/UI modifications to succeed.

The next Node 5 lifecycle implementation will be authorized separately after ChatGPT reviews this S1 implementation report.
