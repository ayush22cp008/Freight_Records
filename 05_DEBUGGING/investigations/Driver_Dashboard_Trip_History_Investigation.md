# Driver Dashboard / Trip-History UX Investigation

**Project:** Freight — AI Builders Hackathon  
**Investigation:** Driver Dashboard enhancement after Node 5 closure  
**Date:** September 2, 2026  
**Status:** INVESTIGATION COMPLETE — implementation not authorized by this file alone  
**Scope:** Driver Dashboard / trip-history UX only

## 1. Purpose

Investigate the dashboard requirement recorded for work after Node 5: the Driver Dashboard should clearly separate available trips, the driver's current/active trip, and past/completed trips.

This investigation must not reopen Node 4 atomic claiming or change the locked trip-claim authorization model.

## 2. Records Basis

The authoritative Node 5 execution checkpoint records the future dashboard requirement as:

```text
Driver Dashboard
├── Available Trips
├── My / Active Trip
└── Past / Completed Trips
```

The same checkpoint explicitly defers this work until Node 5 is formally completed and accepted so it does not mix with or reopen the already-closed Node 4 scope.

Reference record:
`00_PROJECT_CONTROL/CHECKPOINTS/Chat25_Node5_Execution_Checkpoint.md`

## 3. Current Source Observation

Current source inspected:
`src/app/(authenticated)/page.tsx`

Source revision inspected:
`f10df2b8aab1df681e368074c50f76e251c354b4`

### 3.1 Authentication / role routing

The authenticated root route currently:

- redirects unauthenticated users to `/login`;
- gives reviewer authorization priority and redirects reviewers to `/reviewer/queue`;
- resolves the Freight identity and verification state;
- renders a company dashboard for verified company identities;
- otherwise renders the driver dashboard.

### 3.2 Available Trips

When the driver has no active trip, the dashboard queries published trips where `driver_id` is null and renders an **Available Trips** list.

The current cards expose:

- pickup/facility name;
- destination name;
- distance;
- duration;
- payout;
- the existing `ClaimTripButton`.

This confirms that available-trip discovery and the existing claim entry point already exist in the current dashboard.

### 3.3 My / Active Trip

When the authenticated driver has an active/claimed/in-progress trip, the dashboard queries the driver's trip and derives the next workflow action from stored event evidence.

The current state machine covers the expanded Node 5 lifecycle, including:

```text
Arrival
→ Check-in
→ Goods Loaded
→ Pickup Departure
→ In Transit
→ Arrival at Delivery
→ Receiver Check-in
→ Goods Unloaded
→ Delivery Departed
→ Driver / Receiver completion
```

The dashboard therefore already functions as the driver's workflow controller for the current trip.

### 3.4 Past / Completed Trips

The current driver-dashboard implementation does **not** provide a dedicated past/completed-trip section.

The active-trip lookup intentionally filters to:

```text
active
claimed
in_progress
```

When no such trip exists, the current branch switches to Available Trips. There is no corresponding completed-trip query or historical-trip section in the inspected dashboard source.

## 4. Evidence

### Evidence A — Records requirement

`Chat25_Node5_Execution_Checkpoint.md` explicitly records the future dashboard structure as Available Trips, My / Active Trip, and Past / Completed Trips, and states that dashboard work is deferred until after Node 5 acceptance.

### Evidence B — Current source

`src/app/(authenticated)/page.tsx` contains:

- a published-trip query for the no-active-trip branch;
- an active/claimed/in-progress trip query for the active workflow branch;
- event-derived next-action logic for the active trip;
- no dedicated query/render path for completed historical trips.

### Evidence C — Node boundary

Node 4 remains closed. The dashboard enhancement is therefore a UX/history presentation change and must preserve the existing server-side atomic claim behavior and authenticated driver identity boundaries.

## 5. Root Cause / Gap

**Root cause:** The current dashboard was designed primarily as a workflow controller and marketplace entry point. It does not yet expose a historical view for trips that have left the active lifecycle.

This is a **feature/UX gap**, not evidence of a Node 4 claiming defect.

## 6. Decision

Proceed with a separate Driver Dashboard / trip-history UX implementation after the Node 5 closure already recorded in the current project state.

The minimum target structure is:

```text
Driver Dashboard
├── Available Trips
│   └── Published, unclaimed trips
│
├── My / Active Trip
│   └── Current claimed/in-progress trip + next required action
│
└── Past / Completed Trips
    └── Trips historically assigned to this authenticated driver
```

The implementation should reuse existing authorization and identity rules rather than introducing a second ownership mechanism.

## 7. Non-Goals / Locked Boundaries

Do **not**:

- reopen or redesign Node 4 atomic claiming;
- change who is eligible to claim a trip;
- trust a client-supplied driver ID for historical-trip access;
- introduce a second source of truth for trip ownership;
- alter the canonical Node 5 event lifecycle;
- add unrelated dashboard features without a separate investigation;
- bypass the Dashboard's role/state architecture;
- make the dashboard responsible for changing trip state directly.

## 8. Implementation Questions for the Next Plan

Before implementation, the implementation plan should resolve:

1. Which completed/historical trip fields should be displayed in the compact history card.
2. Whether completed trips should link to the existing Timeline and/or another read-only trip detail view.
3. How many historical trips should be loaded initially and whether ordering should be newest-first.
4. Whether non-completed historical states need a separate section or remain covered by My / Active Trip.
5. How the query preserves authenticated-driver ownership without accepting a client-selected driver identity.
6. Whether the current root route should remain one server component or be split into dashboard subcomponents for maintainability.
7. Whether company/reviewer dashboard behavior in the same root route should remain unchanged.

## 9. Required Verification After Implementation

The implementation must be manually verified by Ayush with evidence for at least:

```text
1. Driver with published trips and no active trip
   → Available Trips shown

2. Driver with an active/claimed/in-progress trip
   → My / Active Trip shown
   → existing next-event workflow remains correct

3. Driver with completed historical trip(s)
   → Past / Completed Trips shown
   → only that authenticated driver's history is exposed

4. Driver with no completed history
   → empty-state shown cleanly

5. Existing Node 5 active workflow
   → unchanged

6. Existing claim flow
   → unchanged
```

Build/type-check evidence must also be recorded after implementation.

## 10. Current Investigation Status

```text
Records requirement identified       → VERIFIED
Current dashboard source inspected   → VERIFIED
Available Trips capability            → VERIFIED / EXISTS
My / Active Trip capability           → VERIFIED / EXISTS
Past / Completed Trips                → VERIFIED / MISSING
Root cause                            → VERIFIED AS UX/FEATURE GAP
Node 4 regression                     → NOT INDICATED
Implementation plan                   → REMAINING
Implementation instruction             → REMAINING
Implementation                        → NOT STARTED
Build/test                            → REMAINING
Ayush manual verification             → REMAINING
```

## 11. Recommended Next Step

Create a small implementation plan from this investigation, limited to the Driver Dashboard sections and trip-history presentation. After plan review and Ayush approval, create the Antigravity implementation instruction under `03_IMPLEMENTATION/prompts/`.
