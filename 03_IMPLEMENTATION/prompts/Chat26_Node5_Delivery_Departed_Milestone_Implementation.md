# Chat26 — Node 5 — DELIVERY_DEPARTED Milestone Implementation

## Objective
Implement the next Node 5 delivery milestone: `DELIVERY_DEPARTED`.

The verified delivery sequence now reaches:

```text
ARRIVED_AT_DELIVERY
        ↓
RECEIVER_CHECKED_IN
        ↓
GOODS_UNLOADED
        ↓
DELIVERY_DEPARTED
```

Implement `DELIVERY_DEPARTED` only. Do not implement final dual confirmation/completion or Driver Dashboard redesign.

## Authoritative Records

Read before changing source:

- `00_PROJECT_CONTROL/ROADMAP.md`
- `00_PROJECT_CONTROL/CURRENT_STATUS.md`
- `00_PROJECT_CONTROL/PROJECT_STATE.md`
- `00_PROJECT_CONTROL/CHECKPOINTS/Chat25_Node5_Execution_Checkpoint.md`
- `02_ARCHITECTURE/locked_decisions/Chat24_Node5_Architecture_Decisions.md`
- `03_IMPLEMENTATION/implementation_reports/Chat26_Node5_Current_Lifecycle_Investigation_Report.md`
- `03_IMPLEMENTATION/implementation_reports/Chat26_Node5_S1_Delivery_Evidence_Schema_Migration_Implementation_Report.md`
- `03_IMPLEMENTATION/implementation_reports/Chat26_Node5_Arrived_At_Delivery_Milestone_Implementation_Report.md`
- `03_IMPLEMENTATION/implementation_reports/Chat26_Node5_Receiver_Checked_In_Milestone_Implementation_Report.md`
- `03_IMPLEMENTATION/implementation_reports/Chat26_Node5_Goods_Unloaded_Milestone_Implementation_Report.md`
- `05_DEBUGGING/investigations/Chat26_Node5_Receiver_Checked_In_Driver_Profile_Not_Found_Investigation.md`

The canonical event vocabulary and database uniqueness protection are already established. Do not create or modify the Node 5 schema migration.

## Actor / Authorization

`DELIVERY_DEPARTED` is an **Assigned Driver** action after unloading.

Enforce server-side:

1. Authenticated user exists.
2. Authenticated user resolves to the existing driver identity.
3. The selected trip belongs to that authenticated driver (`trips.driver_id`).
4. The trip is in an established active lifecycle state (`active`, `claimed`, or `in_progress`, as supported by current source).
5. Authorization must come from the authenticated server-side identity and trip relationship, never from a client-supplied actor ID.

Follow the secure driver authorization pattern used by the verified Node 5 milestones.

## State / Sequence Rules

`DELIVERY_DEPARTED` is legal only after the receiving-side and unloading prerequisites:

```text
ARRIVED_AT_DELIVERY
        ↓
RECEIVER_CHECKED_IN
        ↓
GOODS_UNLOADED
        ↓
DELIVERY_DEPARTED
```

At minimum, the backend must reject:

- unauthenticated requests;
- requests from an unrelated/non-assigned driver;
- nonexistent or unauthorized trips;
- trips outside the established active lifecycle states;
- departure before `ARRIVED_AT_DELIVERY`;
- departure before `RECEIVER_CHECKED_IN`;
- departure before `GOODS_UNLOADED`;
- duplicate `DELIVERY_DEPARTED`.

Do not use or create a new `trips.status` value such as `delivery_departed`. The detailed milestone belongs in the `events` table.

## Event Contract

Record exactly:

```text
DELIVERY_DEPARTED
```

Do not write a legacy `departure` event for this Node 5 milestone. Historical legacy rows must remain unchanged.

Retain the existing `UNIQUE (trip_id,event_type)` database protection as the final duplicate safeguard.

## Evidence / Photo / GPS

Reuse the established Node 5 secure evidence architecture.

If the current departure UX supports optional photo evidence, use the existing upload mechanism. Do not create a second storage system, public bucket, or alternate evidence architecture.

Capture GPS and server-side timestamp using the established Node 5 pattern where applicable. Do not trust a client-supplied authoritative timestamp.

## UI

Add the smallest necessary driver-side UI for `DELIVERY_DEPARTED`.

The UI should:

- expose the action only after `GOODS_UNLOADED`;
- clearly identify the active delivery/trip;
- allow established evidence inputs where applicable;
- prevent duplicate submission while pending;
- call the server-side endpoint;
- show success/failure clearly;
- refresh/update the existing lifecycle/timeline state after success.

Do not redesign the Driver Dashboard or timeline in this task.

Do not implement final completion/dual confirmation in this task.

## API

Create the smallest necessary server endpoint following the established Node 5 route convention, for example:

```text
POST /api/events/delivery-departed
```

Use the actual source convention if it differs.

The endpoint must perform authentication, assigned-driver authorization, active-trip validation, prerequisite event checks, duplicate protection, and canonical event insertion server-side.

Do not rely on frontend checks for authorization or sequencing.

## Timeline

The existing unified timeline must display the canonical event after `GOODS_UNLOADED`:

```text
STEP 8: GOODS_UNLOADED
STEP 9: DELIVERY_DEPARTED
```

Use the canonical event type and existing chronological rendering. Do not create a second timeline.

## Completion Boundary

`DELIVERY_DEPARTED` is the final physical-movement milestone before the separate final completion/dual-confirmation stage.

Do **not** automatically mark the trip fully completed merely because `DELIVERY_DEPARTED` was recorded.

Do not set or invent:

```text
DRIVER_COMPLETION_CONFIRMED
RECEIVER_DELIVERY_CONFIRMED
completed status
```

unless existing source behavior requires an unrelated read-only display update; no final completion mutation belongs in this task.

## Implementation Boundaries

Allowed:

- New server/API route required for `DELIVERY_DEPARTED`.
- Minimal driver page/client required for delivery departure.
- Minimal dashboard/lifecycle change required to expose the action.
- Minimal timeline/event-label mapping if required.
- Directly relevant tests/checks.

Not allowed:

```text
DRIVER_COMPLETION_CONFIRMED          = NO
RECEIVER_DELIVERY_CONFIRMED          = NO
Final completion                     = NO
Driver Dashboard redesign            = NO
New schema migration                  = NO
trips.status architecture change     = NO
Legacy historical row rewriting      = NO
New relationship architecture        = NO
Evidence-storage redesign            = NO
Unrelated refactors                  = NO
```

## Verification Requirements

### Positive path

Verify:

```text
Authenticated Assigned Driver
+ correct active trip
+ ARRIVED_AT_DELIVERY exists
+ RECEIVER_CHECKED_IN exists
+ GOODS_UNLOADED exists
        ↓
DELIVERY_DEPARTED recorded successfully
```

Confirm the event appears in the unified timeline with GPS/timestamp/photo evidence where applicable.

Confirm the trip is **not** treated as fully completed solely from this event.

### Negative / security paths

Verify server-side rejection for:

- unauthenticated user;
- another driver;
- unassigned/non-authorized driver;
- nonexistent or unauthorized trip;
- departure before `ARRIVED_AT_DELIVERY`;
- departure before `RECEIVER_CHECKED_IN`;
- departure before `GOODS_UNLOADED`;
- duplicate departure;
- invalid/out-of-scope trip lifecycle state.

Do not perform destructive tests against shared production data. Use the established safe testing approach.

Run available relevant checks and record exact commands/results. At minimum:

```text
npx tsc --noEmit
```

Do not claim checks passed unless actually executed.

## Manual Verification Boundary

Clearly distinguish:

- automated/type/build/test evidence;
- Antigravity implementation observations;
- Ayush manual browser verification.

Use `VERIFIED`, `INFERRED`, and `UNKNOWN` accurately. Do not claim Ayush verification without actual browser evidence from Ayush.

After implementation, stop. ChatGPT will review the report and separately authorize the final completion/dual-confirmation stage.

## Required Implementation Report

Create exactly one Records report:

`03_IMPLEMENTATION/implementation_reports/Chat26_Node5_Delivery_Departed_Milestone_Implementation_Report.md`

Include:

1. Implementation status.
2. Source repository/branch/commit before and after, if available.
3. Exact files changed.
4. API/server behavior.
5. Authentication and assigned-driver authorization evidence.
6. Sequence/state validation evidence.
7. UI behavior.
8. Timeline/event rendering behavior.
9. GPS/photo/evidence handling.
10. Duplicate protection.
11. Negative/security checks actually performed.
12. Exact build/type/test/lint commands and actual results.
13. Manual browser verification status.
14. Explicit evidence that final completion was not triggered.
15. VERIFIED / INFERRED / UNKNOWN summary.
16. Explicit confirmation that final dual confirmation and Driver Dashboard redesign were not implemented.
17. Any issues, constraints, or stop conditions.

Do not create another implementation prompt or unrelated Records file.

## Stop Conditions

Stop and report instead of continuing if:

- the current source cannot safely identify the assigned driver;
- `GOODS_UNLOADED` cannot be enforced server-side as the immediate prerequisite;
- the existing schema is materially inconsistent with the locked Node 5 architecture;
- secure evidence storage cannot be reused safely;
- implementation requires a new relationship model or `trips.status` architecture change;
- existing verified Node 5 behavior would have to be broken or rewritten;
- implementation would require final completion/dual confirmation or dashboard redesign.

## Completion Boundary

This task is complete only when `DELIVERY_DEPARTED` is implemented and the implementation report is written.

Do not automatically proceed to final completion/dual confirmation. ChatGPT will review the implementation report and separately authorize that stage.

Do not push to GitHub unless explicitly instructed by Ayush.