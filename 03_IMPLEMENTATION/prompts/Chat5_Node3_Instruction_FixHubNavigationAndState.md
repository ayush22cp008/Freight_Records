# Chat5 Node3 — Implementation Instruction: Fix Hub Navigation + State Foundation

**Type:** IMPLEMENTATION
**Status:** READY
**Prerequisite:** `00_PROJECT_CONTROL/Chat5_Node3_Flow_And_Navigation_Decision.md`

## Objective

Implement the locked Trip Hub/navigation foundation before continuing Check-in.

The Hub at `/` must become the single source of truth for the current workflow state and next required action.

Do not redesign the database into a multi-stop model. Do not add future journey features. Implement only the locked current single-facility, fixed 3-event MVP flow.

## Required flow

```text
/login
  ↓
/
  ↓
/events/arrival
  ↓
/
  ↓
/events/checkin
  ↓
/
  ↓
/events/departure
  ↓
/timeline
  ↓
AI Evidence Summary
```

## Required implementation behavior

### 1. Trip Hub `/`

Update the dashboard so it:

- requires authentication as currently intended
- displays current driver/trip information
- displays Arrival / Check-in / Departure progress
- queries authoritative database state
- determines the next required event from stored event state
- displays one clear primary CTA for that next event
- does not rely on client-only state to determine the next event

Expected states:

```text
No Arrival → Start Arrival
Arrival complete → Start Check-in
Check-in complete → Start Departure
Departure complete → View Timeline
```

Do not implement AI summary as part of this instruction unless an existing completed-summary route already exists; only provide the correct final navigation target where appropriate.

### 2. Arrival navigation

Connect Hub → `/events/arrival` when Arrival is the next required event.

After successful Arrival submission:

- return to `/`
- Hub must now show Arrival completed and Check-in as the next action

### 3. Duplicate Arrival protection

Before allowing Arrival submission, verify authoritative event state.

If Arrival already exists for the active trip:

- do not create another immutable Arrival event
- do not overwrite the existing event
- return/block to the correct Hub state

Do not weaken RLS or immutability to solve this.

### 4. Check-in route foundation

Do not implement the complete Check-in feature in this task.

If needed to support navigation, establish only the minimal route/guard structure required so that `/events/checkin` is treated as the next workflow destination without pretending Check-in is complete.

The actual Check-in implementation remains the next feature task.

### 5. Out-of-order route protection

For the existing Arrival route and any route/guard foundation introduced for Check-in/Departure:

- later event must not be accessible when its prerequisite event is incomplete
- direct URL access must not bypass event order
- route decisions must be based on authoritative state

At minimum, ensure the existing Arrival route cannot create duplicates.

### 6. Back behavior

Before event submission, an event page's explicit Back/Return action should go to `/`.

Do not use browser history as the workflow controller.

### 7. Refresh behavior

After refresh, `/` must recompute the current state from the database.

The UI must correctly show the next required event based on stored records.

### 8. Authentication

Preserve the existing authentication mechanism.

Do not introduce a new auth architecture.

Ensure protected routes remain protected.

## Constraints

- Do NOT implement multi-stop.
- Do NOT add Pickup/Delivery-specific event types.
- Do NOT add an In Transit event.
- Do NOT redesign the existing database schema unless a minimal change is strictly required by the current locked MVP and is documented before implementation.
- Do NOT remove existing verified functionality.
- Do NOT modify records in `Freight_Records` from the source project unless specifically required by the project workflow.
- Preserve immutable event storage and existing security/RLS design.

## Verification requirements

After implementation, run appropriate automated/build checks.

Then write an implementation report to:

`03_IMPLEMENTATION/implementation_reports/Chat5_Node3_Report_FixHubNavigationAndState.md`

The report must include:

1. Files changed
2. What was changed
3. Current route graph after the fix
4. Current state/next-event logic
5. Duplicate Arrival protection
6. Authentication/route guard behavior
7. Build/test results
8. Known limitations
9. Manual verification steps for Ayush

Do not claim manual browser verification was performed by Antigravity unless it actually was.

## Completion condition

This task is complete only when:

- `/` is a usable Trip Hub
- the Hub links to the correct next event
- Arrival success returns to Hub
- Hub reflects Arrival completion
- duplicate Arrival submission is prevented
- refresh reconstructs state from the database
- navigation does not depend on browser history
- build/tests pass or failures are explicitly documented
- the implementation report is written

After completion, stop. Do not continue automatically into Check-in implementation.
