# Chat44 — Day 18 — Node 7 — Phase 1b
## Company — My Created Trips Active-Only Correction Instruction

**Status:** FIX AUTHORIZED — FRONTEND ONLY  
**Portal:** Company  
**Scope:** Single targeted correction to My Created Trips

---

## 1. Objective

Apply one correction only:

> **My Created Trips → Active Trips must contain only non-completed trips.**

This is a frontend presentation/filtering correction. Do not redesign the Company portal and do not change Dashboard, History, Trip Detail, backend behavior, or lifecycle semantics.

---

## 2. Verified Context

The completed Day18 investigation established that:

- Company My Created Trips currently exposes created trips in a way that allows completed trips to appear in the active/current surface.
- Dashboard Active Created Trips already uses the correct active-state filtering and must not be changed for this correction.
- History already provides the Company's historical/completed-trip surface.
- Company Trip Detail is already working and must remain unchanged.
- The correction can be achieved using the existing frontend trip data.
- No backend, API, database, RLS, authentication, authorization, or lifecycle change is required.

The user's manual verification has already covered Dashboard, History, and Trip Detail. Therefore this implementation instruction is limited to the remaining My Created Trips correction.

---

## 3. Source-of-Truth Documents

Before editing, read:

1. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
2. `00_PROJECT_CONTROL/DECISIONS/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary.md`
3. `03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md`
4. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Active_vs_Completed_Trip_Investigation_Report.md`
5. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Trip_Detail_Investigation_Report.md`
6. `03_IMPLEMENTATION/implementation_reports/Chat44_Day18_Node7_Phase1b_Company_Active_vs_Completed_Trip_Implementation_Report.md`

Also inspect the current implementation before editing so the change is based on the actual source state.

---

## 4. Exact Target

Primary target:

`src/app/(authenticated)/company/created/page.tsx`

Prefer a localized change in this file.

Do not modify unrelated Company surfaces.

---

## 5. Required Behavior

For **My Created Trips**:

### Active Trips

Render only trips whose existing lifecycle status is **not** `completed`.

Equivalent presentation rule:

`status !== 'completed'`

Therefore, a trip with:

- `active`
- `claimed`
- `in_progress`
- or another existing non-completed status

may remain in Active Trips according to the existing application semantics.

A trip with:

`status === 'completed'`

must not appear in the Active Trips/current list.

---

## 6. Completed Trips and History

Do **not** create a new Completed Trips section inside My Created Trips for this correction.

The intended result for this build is:

- **My Created Trips → Active Trips:** only non-completed created trips.
- **History:** completed/historical trips remain available there.

Do not delete completed trip records.

Do not modify the History query or filtering.

Do not change the meaning of `trip.status`.

---

## 7. Dashboard — STRICTLY UNCHANGED

Do not modify Dashboard Active Created Trips.

The verified Dashboard filtering already excludes completed trips:

`.eq('company_id', company.id).in('status', ['active', 'claimed', 'in_progress', 'draft'])`

No Dashboard query or lifecycle logic change is part of this task.

---

## 8. Trip Detail — STRICTLY UNCHANGED

Do not modify the existing Company Trip Detail implementation or route as part of this correction.

Existing Trip Detail navigation and rendering must remain intact for the trips that remain visible in My Created Trips.

Do not introduce a new Trip Detail implementation.

---

## 9. Protected Scope

Do NOT modify:

- database/schema;
- Supabase/RLS policies;
- authentication/authorization logic;
- role assignment;
- backend business rules;
- lifecycle/state-transition logic;
- claiming/marketplace behavior;
- evidence requirements or integrity;
- AI behavior;
- Reviewer authority;
- API contracts or response shapes;
- `src/app/api/completion/route.ts`;
- Dashboard filtering;
- History filtering;
- Trip Detail behavior;
- Driver Portal behavior;
- shared backend behavior.

Do not change an API query when the existing frontend data is sufficient to filter the presentation.

---

## 10. No Lifecycle Changes

This task is **not** a lifecycle fix.

Do not:

- introduce a new status;
- rename a status;
- automatically transition a trip;
- infer completion from timeline events;
- alter claiming behavior;
- alter backend completion behavior.

Only the rendered My Created Trips list changes.

---

## 11. Empty-State Requirement

If filtering leaves no non-completed trips:

- do not show completed trips as active substitutes;
- show an appropriate active/current empty state;
- preserve the existing Create New Trip path.

Do not invent backend data or new workflow behavior.

---

## 12. Preserve Existing Card Behavior

For every trip that remains in Active Trips, preserve:

- route/identity information;
- current status display;
- driver-claim information;
- payout information;
- distance information;
- existing card styling unless a minimal adjustment is required;
- existing Trip Detail navigation;
- existing responsive behavior.

Do not redesign the card component unless strictly necessary for this correction.

---

## 13. Implementation Procedure

1. Inspect the current `company/created/page.tsx` implementation.
2. Identify the existing `createdTrips` data source and current rendering path.
3. Apply frontend-only filtering so the Active Trips rendering receives only `status !== 'completed'` trips.
4. Do not change the source API contract merely to achieve this filtering.
5. Do not add a separate Completed Trips section.
6. Do not modify Dashboard, History, or Trip Detail.
7. Run the appropriate project build/type/lint checks.
8. Confirm the change is limited to the intended frontend scope.

If the existing data is insufficient or a protected subsystem must be changed, **STOP and report** instead of expanding scope.

---

## 14. Verification Requirements

Automated/source verification must confirm:

- My Created Trips Active Trips excludes `status === 'completed'`.
- Existing non-completed trips remain eligible for the Active Trips list.
- Dashboard code/filter was not changed.
- History code/filter was not changed.
- Trip Detail code/behavior was not changed.
- No backend/API/DB/RLS/auth/lifecycle files were changed.
- No Driver Portal behavior was changed.

Manual verification for Ayush:

1. Open **My Created Trips**.
2. Confirm the page's **Active Trips** list contains only non-completed trips.
3. Confirm previously visible completed trips no longer appear in that Active Trips list.
4. Open **History** and confirm completed trips remain available there.
5. Confirm an active My Created Trip still opens the existing Trip Detail page.

Do not claim Company Portal acceptance. Ayush remains the final manual verification authority.

---

## 15. Required Implementation Report

Update/create:

`03_IMPLEMENTATION/implementation_reports/Chat44_Day18_Node7_Phase1b_Company_My_Created_Trips_Active_Only_Correction_Implementation_Report.md`

The report must contain:

- exact files changed;
- exact filtering/presentation change;
- confirmation that completed trips are excluded from My Created Trips Active Trips;
- confirmation that Dashboard was not changed;
- confirmation that History was not changed;
- confirmation that Trip Detail was not changed;
- confirmation that backend/API/DB/RLS/auth/lifecycle were not changed;
- build/type/lint/test results;
- manual verification steps for Ayush;
- remaining UNKNOWNs, if any;
- implementation status.

Do not claim final acceptance.

---

## 16. Stop Conditions

STOP immediately if:

- frontend filtering cannot achieve the required result with existing data;
- an API contract must change;
- a backend route must change;
- database/schema/RLS changes appear necessary;
- authentication/authorization must change;
- lifecycle semantics must change;
- claiming behavior must change;
- evidence behavior must change;
- Driver behavior could be affected;
- Reviewer behavior could be affected;
- Dashboard, History, or Trip Detail must be modified to achieve the requested correction;
- the locked Blueprint is contradicted;
- a new product behavior is required beyond the stated active-only presentation rule.

Do not work around a protected boundary.

---

## 17. Final Gate

After implementation and automated checks:

**STOP. Do not declare the Company Portal accepted.**

Wait for Ayush to manually verify the My Created Trips Active Trips list.

The intended final state is exactly:

> **My Created Trips → Active Trips → only non-completed trips.**

Completed trips remain available through **History**.

No other Company surface should be changed by this correction.
