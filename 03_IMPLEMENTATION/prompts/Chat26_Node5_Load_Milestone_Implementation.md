# Chat26 — Node 5 — GOODS_LOADED Milestone Implementation

## Objective

Implement the **Load** milestone for Node 5 — Whole Delivery Tracking.

This is the first application-layer lifecycle step after the verified S1 schema migration.

The target pickup sequence is:

```text
ARRIVED_AT_PICKUP
        ↓
PICKUP_CHECKED_IN
        ↓
GOODS_LOADED
        ↓
PICKUP_DEPARTED
```

Implement **GOODS_LOADED only** in this task. Do not implement transit, destination, receiver workflow, final completion, or dashboard redesign.

## Authoritative Records

Read before changing source:

- `00_PROJECT_CONTROL/ROADMAP.md`
- `00_PROJECT_CONTROL/CURRENT_STATUS.md`
- `00_PROJECT_CONTROL/PROJECT_STATE.md`
- `00_PROJECT_CONTROL/CHECKPOINTS/Chat25_Node5_Execution_Checkpoint.md`
- `02_ARCHITECTURE/locked_decisions/Chat24_Node5_Architecture_Decisions.md`
- `03_IMPLEMENTATION/implementation_reports/Chat25_Node5_S1_Delivery_Evidence_Schema_Migration_Design_Report.md`
- `05_DEBUGGING/investigations/Chat26_Node5_Current_Lifecycle_Investigation_Report.md`
- `03_IMPLEMENTATION/implementation_reports/Chat26_Node5_S1_Delivery_Evidence_Schema_Migration_Implementation_Report.md`

The current Supabase schema has already been manually verified by Ayush to accept the canonical Node 5 event vocabulary, preserve uniqueness, and contain the two completion timestamp fields. Do not repeat or alter the S1 migration.

## Required Behavior

Add a driver-authorized Load action that records exactly one canonical event:

```text
GOODS_LOADED
```

The implementation must use the existing server-side event architecture and secure authenticated context. Do not trust a client-supplied actor identity.

## State / Sequence Rules

`GOODS_LOADED` must be allowed only when the trip is in the correct Node 5 pickup progression.

At minimum, enforce server-side that:

1. The trip exists.
2. The authenticated user is the authorized driver for the claimed/in-progress trip.
3. The trip is in an appropriate active lifecycle state.
4. The required preceding pickup milestone `PICKUP_CHECKED_IN` exists for the trip.
5. `GOODS_LOADED` has not already been recorded.
6. The event is recorded using the canonical uppercase event type.
7. The existing database uniqueness protection remains the final duplicate safeguard.

Do not bypass or weaken existing RLS/authorization.

If the current source has a different but compatible server-side authorization pattern, follow that established pattern rather than inventing a parallel authorization architecture.

## Existing Legacy Flow

The current verified implementation uses legacy event names:

```text
arrival
checkin
departure
```

Do not rewrite historical rows.

For the new Load milestone, use only:

```text
GOODS_LOADED
```

Do not create a new legacy `departure`, `checkin`, or other ambiguous event for Load.

## UI Requirements

Add the smallest necessary UI change to expose Load to the authorized driver at the correct point in the pickup flow.

The UI must:

- show Load only when appropriate;
- prevent obvious duplicate submission while the request is pending;
- call the server-side implementation;
- show success/failure clearly;
- refresh/update the existing unified timeline or lifecycle display so `GOODS_LOADED` becomes visible.

Do not redesign the dashboard or timeline in this task. Extend the existing UI minimally.

## Evidence / Photo Handling

Reuse the existing secure evidence architecture if the current Load event contract supports an evidence/photo attachment.

Do not invent a new storage system.

If the current Node 5 requirements do not require a Load photo, do not add one merely for this task. Document the decision in the implementation report.

## Implementation Boundaries

Allowed:

- API/server action required to create `GOODS_LOADED`.
- Minimal driver UI required to trigger Load.
- Minimal existing timeline/lifecycle mapping required to display the new event.
- Tests directly relevant to the Load milestone.

Not allowed:

```text
IN_TRANSIT implementation          = NO
ARRIVED_AT_DELIVERY                 = NO
RECEIVER_CHECKED_IN                 = NO
GOODS_UNLOADED                      = NO
DELIVERY_DEPARTED                   = NO
Final completion                    = NO
Driver Dashboard redesign           = NO
Schema migration changes             = NO
Legacy historical row rewriting     = NO
Unrelated refactors                 = NO
```

## Verification Requirements

Test the Load milestone at minimum for:

### Positive path

```text
Authorized driver
+ correct trip
+ PICKUP_CHECKED_IN exists
        ↓
GOODS_LOADED recorded successfully
```

### Negative / security paths

Verify server-side rejection for:

- unauthenticated request;
- user who is not the trip's authorized driver;
- trip not in an appropriate lifecycle state;
- Load attempted before pickup check-in;
- duplicate Load attempt.

Do not create destructive or shared-production test data merely for testing. Use the project's established safe testing approach.

Run appropriate build/type/test checks available in the source repository and report exact commands/results.

## Important Source-Consistency Rule

Before implementation, inspect the current source because the verified S1 migration changed the database capability but did not update the legacy application code.

If changing the application requires a material architecture decision outside this Load milestone, stop and report the conflict instead of silently changing the architecture.

## Required Implementation Report

Create exactly one Records report:

`03_IMPLEMENTATION/implementation_reports/Chat26_Node5_Load_Milestone_Implementation_Report.md`

Include:

1. Implementation status.
2. Source repository/branch/commit before and after.
3. Exact files changed.
4. Server/API behavior.
5. Authorization evidence.
6. Sequence/state validation evidence.
7. UI behavior.
8. Timeline/event rendering behavior.
9. Duplicate protection evidence.
10. Test/build/type-check commands and actual results.
11. Manual verification status, clearly separated from automated evidence.
12. Any UNKNOWN/INFERRED items.
13. VERIFIED / INFERRED / UNKNOWN summary.
14. Explicit confirmation that no out-of-scope Node 5 features were implemented.

Do not create another implementation prompt or unrelated Records file.

## Stop Conditions

Stop and report instead of implementing further if:

- the current source does not provide a safe way to identify the authorized driver;
- the required sequence cannot be enforced server-side without a new architectural decision;
- the S1 schema is inconsistent with the source in a material way;
- existing verified Arrival/Check-in behavior would have to be broken or rewritten;
- implementation would require transit, receiver, completion, or dashboard work.

## Completion Boundary

This task is complete only when the Load milestone is implemented and evidenced.

Do not proceed automatically to `PICKUP_DEPARTED` after completing Load. ChatGPT will review the Load implementation report and separately authorize the next Node 5 milestone.
