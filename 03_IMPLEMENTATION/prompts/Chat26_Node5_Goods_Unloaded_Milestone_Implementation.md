# Chat26 — Node 5 — GOODS_UNLOADED Milestone Implementation

## Objective
Implement the next Node 5 delivery milestone: `GOODS_UNLOADED`.

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

Implement `GOODS_UNLOADED` only. Do not implement `DELIVERY_DEPARTED`, final completion/dual confirmation, or Driver Dashboard redesign.

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
- `05_DEBUGGING/investigations/Chat26_Node5_Receiver_Checked_In_Driver_Profile_Not_Found_Investigation.md`

The canonical event vocabulary and `UNIQUE (trip_id,event_type)` protection are already established. Do not create a schema migration for this milestone.

## Actor / Authorization

`GOODS_UNLOADED` is an **Assigned Driver** action. The authorization matrix defines unload/delivery as belonging to the assigned driver.

Enforce server-side:

1. Authenticated user exists.
2. Authenticated user resolves to the existing driver identity.
3. The selected/active trip belongs to that authenticated driver (`trips.driver_id`).
4. The trip is in an established active lifecycle state (`active`, `claimed`, or `in_progress`, as supported by current source).
5. The receiving-company relationship is not used as driver authorization; receiver check-in is the preceding company action.
6. Client-supplied driver IDs must never establish authorization.

Follow the existing secure driver authorization pattern used by verified Node 5 milestones.

## State / Sequence Rules

`GOODS_UNLOADED` is legal only after the destination and receiver prerequisites have been recorded:

```text
ARRIVED_AT_DELIVERY
        ↓
RECEIVER_CHECKED_IN
        ↓
GOODS_UNLOADED
```

At minimum, the backend must reject:

- unauthenticated requests;
- requests from a non-driver or unrelated driver;
- trips not belonging to the authenticated assigned driver;
- trips outside the established active lifecycle states;
- unload before `ARRIVED_AT_DELIVERY`;
- unload before `RECEIVER_CHECKED_IN`;
- duplicate `GOODS_UNLOADED`.

Use the existing `events` table for the detailed milestone. Do not add a new `trips.status` value such as `unloaded`.

## Event Contract

Record exactly:

```text
GOODS_UNLOADED
```

Do not write a legacy `arrival`, `checkin`, or `departure` event for this milestone. Preserve historical legacy rows unchanged.

Retain the existing database uniqueness protection as the final duplicate safeguard.

## Evidence / Photo / GPS

Inspect the verified Node 5 evidence pattern and reuse the existing secure mechanism.

If the current unload UX supports optional photo evidence, reuse the existing photo-upload infrastructure, including the recently fixed dual-role authorization for company uploads; do not create a second storage system.

Capture GPS/evidence according to the established Node 5 pattern where applicable.

Do not invent new buckets, public storage URLs, or evidence architecture.

## UI

Add the smallest necessary driver-side UI for `GOODS_UNLOADED`.

The UI should:

- expose the action only after `RECEIVER_CHECKED_IN`;
- clearly identify the active delivery/trip;
- allow the established evidence inputs where applicable;
- prevent duplicate submission while pending;
- submit to the server-side endpoint;
- show success/failure clearly;
- return/refresh the existing lifecycle/timeline state after success.

Extend the existing dashboard/timeline minimally. Do not redesign the Driver Dashboard.

The next `DELIVERY_DEPARTED` action must not be implemented in this task.

## API

Create the smallest necessary server endpoint following the established Node 5 naming/pattern, for example:

```text
POST /api/events/goods-unloaded
```

Use the actual source convention if it differs.

The endpoint must perform authentication, assigned-driver authorization, active-trip validation, prerequisite event checks, duplicate protection, and canonical event insertion server-side.

Do not rely on frontend checks for authorization or sequencing.

Do not trust a client-supplied authoritative timestamp or actor identity. Reuse the established server-side timestamp/evidence pattern.

## Timeline

The existing unified timeline must display:

```text
STEP 6: ARRIVED_AT_DELIVERY
STEP 7: RECEIVER_CHECKED_IN
STEP 8: GOODS_UNLOADED
```

Use the canonical event type and existing chronological rendering. Do not create a second timeline.

## Implementation Boundaries

Allowed:

- New server/API route for `GOODS_UNLOADED`.
- Minimal driver page/client required for unload.
- Minimal dashboard/lifecycle change required to expose unload.
- Minimal timeline/event-label mapping if required.
- Directly relevant tests/checks.

Not allowed:

```text
DELIVERY_DEPARTED                    = NO
DRIVER_COMPLETION_CONFIRMED          = NO
RECEIVER_DELIVERY_CONFIRMED          = NO
Final completion                     = NO
Driver Dashboard redesign            = NO
New schema migration                  = NO
trips.status architecture change     = NO
Legacy historical row rewriting      = NO
New relationship architecture        = NO
Repeatable evidence redesign         = NO
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
        ↓
GOODS_UNLOADED recorded successfully
```

Confirm the event appears in the unified timeline with the established evidence fields where applicable.

### Negative / security paths

Verify server-side rejection for:

- unauthenticated user;
- another driver;
- unassigned/non-authorized driver;
- nonexistent or unauthorized trip;
- unload before `ARRIVED_AT_DELIVERY`;
- unload before `RECEIVER_CHECKED_IN`;
- duplicate unload;
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

After implementation, stop. ChatGPT will review the report and separately authorize `DELIVERY_DEPARTED`.

## Required Implementation Report

Create exactly one Records report:

`03_IMPLEMENTATION/implementation_reports/Chat26_Node5_Goods_Unloaded_Milestone_Implementation_Report.md`

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
14. VERIFIED / INFERRED / UNKNOWN summary.
15. Explicit confirmation that delivery departure, final completion, and Driver Dashboard redesign were not implemented.
16. Any issues, constraints, or stop conditions.

Do not create another implementation prompt or unrelated Records file.

## Stop Conditions

Stop and report instead of continuing if:

- the current source cannot safely identify the assigned driver;
- `RECEIVER_CHECKED_IN` cannot be enforced server-side as the prerequisite;
- the existing schema is materially inconsistent with the locked Node 5 architecture;
- secure evidence storage cannot be reused safely;
- implementation requires a new relationship model or `trips.status` architecture change;
- existing verified Node 5 behavior would have to be broken or rewritten;
- implementation would require `DELIVERY_DEPARTED`, final completion, or dashboard redesign.

## Completion Boundary

This task is complete only when `GOODS_UNLOADED` is implemented and the implementation report is written.

Do not automatically proceed to `DELIVERY_DEPARTED`. ChatGPT will review the implementation report and separately authorize the next milestone.

Do not push to GitHub unless explicitly instructed by Ayush.