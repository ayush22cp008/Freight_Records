# Antigravity Implementation Instruction — Driver Dashboard Trip History UX

**Project:** Freight — AI Builders Hackathon  
**Source repository:** `ayush22cp008/freight_hackathon`  
**Records repository:** `ayush22cp008/Freight_Records`  
**Scope:** Driver Dashboard / trip-history UX only  
**Status:** READY FOR ANTIGRAVITY EXECUTION  

## 1. Authority / Source Records

Implement exactly from the approved investigation and implementation plan:

- `05_DEBUGGING/investigations/Driver_Dashboard_Trip_History_Investigation.md`
- `05_DEBUGGING/investigations/Driver_Dashboard_Trip_History_Implementation_Plan.md`

Do not expand scope beyond this instruction.

## 2. Objective

Update the authenticated Driver Dashboard so the driver can see three logical areas:

```text
Driver Dashboard
├── Available Trips
├── My / Active Trip
└── Past / Completed Trips
```

The existing active-trip workflow and available-trip claiming behavior must continue to work unchanged.

## 3. Only Intended Source Change

Primary source file:

```text
src/app/(authenticated)/page.tsx
```

Keep the implementation in this file. Do not introduce a new dashboard architecture or split the existing RSC boundary for this task.

## 4. Required Data Query

After the server has resolved the authenticated driver's database identity (`driverId`), fetch completed historical trips using the existing server-side Supabase client and authenticated driver identity.

Required query shape:

```typescript
const { data: completedTrips } = await supabaseServer
  .from('trips')
  .select('id, facility_name, destination_name, distance, duration, payout')
  .eq('driver_id', driverId)
  .eq('status', 'completed')
  .order('created_at', { ascending: false })
  .limit(10);
```

Do not accept a client-supplied driver ID for this query.

## 5. Required Dashboard Structure

For the driver branch only:

### A. My / Active Trip

If an active/claimed/in-progress trip exists, preserve the existing active-trip state machine and next-action CTA exactly as currently implemented.

The existing Node 5 lifecycle logic must remain intact:

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
→ Completion
```

Do not change event names, event ordering, completion logic, or workflow authorization.

### B. Available Trips

If there is no active/claimed/in-progress trip, preserve the current published-trip marketplace behavior and existing `ClaimTripButton`.

Do not modify atomic claim behavior.

### C. Past / Completed Trips

Render a dedicated section for `completedTrips` on the driver dashboard.

Each completed trip should expose the approved basic fields:

- Facility / pickup name
- Destination / dropoff name
- Payout
- Distance
- Duration

Each completed trip must provide a read-only navigation CTA to:

```text
/timeline?tripId=<id>
```

Use Next.js `Link` consistent with the existing source style.

## 6. Rendering / Empty State

The dashboard should render the Past / Completed Trips section even when there are no completed trips, with a clean empty state such as:

```text
No completed trips yet.
```

Do not hide or remove the Available Trips section when it is needed by the existing no-active-trip flow.

The intended result is that a driver without an active trip can see both:

```text
Available Trips
Past / Completed Trips
```

A driver with an active trip can see both:

```text
My / Active Trip
Past / Completed Trips
```

## 7. Security / Authorization Constraints

These are mandatory:

- Resolve `driverId` from the authenticated server-side user as the current implementation does.
- Historical trips must be filtered by that server-resolved `driverId`.
- Never use a URL/query parameter to select another driver's history.
- Do not weaken RLS, API authorization, or existing identity checks.
- Do not modify Node 4 claim authorization.
- Do not modify Node 5 completion authorization.

## 8. Company / Reviewer Isolation

Do not change company-dashboard behavior.

Do not change reviewer routing or reviewer behavior.

The dashboard enhancement applies only to the driver branch of the authenticated root page.

## 9. Non-Goals

Do not:

- redesign the dashboard visually beyond the required structural sections;
- create a new trip-history route;
- create new APIs;
- modify database schema or migrations;
- change trip status semantics;
- modify the canonical Node 5 event lifecycle;
- modify timeline authorization or event storage;
- change claim eligibility or atomic first-valid claim behavior;
- add pagination beyond the approved top-10 limit;
- add unrelated dashboard features.

## 10. Implementation Discipline

Before changing code:

1. Inspect the current `src/app/(authenticated)/page.tsx` on the current source branch.
2. Confirm the implementation plan still matches the actual source.
3. If the source has materially diverged from the plan, STOP and report the discrepancy instead of guessing.

After implementation:

1. Run:

```text
npx tsc --noEmit
```

2. Report the exact result.
3. Do not push to GitHub unless Ayush explicitly authorizes the push.
4. Do not claim manual verification; only Ayush can provide that evidence.

## 11. Required Implementation Report

Create/update the Antigravity implementation report under:

```text
03_IMPLEMENTATION/implementation_reports/Driver_Dashboard_Trip_History_Implementation_Report.md
```

The report must include:

- files changed;
- exact implementation summary;
- completed-trip query used;
- security/ownership approach;
- confirmation that Company/Reviewer behavior was not changed;
- `npx tsc --noEmit` result;
- any warnings or deviations;
- whether a push was performed (normally: NO unless explicitly authorized);
- status: IMPLEMENTED / BLOCKED / PARTIAL.

## 12. Success Criteria

The implementation is successful only when:

```text
[ ] Available Trips remains functional
[ ] My / Active Trip remains functional
[ ] Past / Completed Trips appears
[ ] Completed history is limited to authenticated driver
[ ] Maximum 10 completed trips, newest first
[ ] Required trip details are visible
[ ] View Timeline opens /timeline?tripId=<id>
[ ] Empty completed-history state works
[ ] Existing claim flow unchanged
[ ] Existing Node 5 workflow unchanged
[ ] Company dashboard unchanged
[ ] Reviewer behavior unchanged
[ ] npx tsc --noEmit passes
[ ] Implementation report written
[ ] No unauthorized push performed
```

## 13. Final Boundary

This is a focused UX/history enhancement following the approved Records investigation and plan. Do not treat it as a Node 4 rework or a Node 5 lifecycle change.

If implementation exposes a security, authorization, data-model, or architectural contradiction that cannot be safely resolved within this scope, STOP and report it rather than improvising a fix.
