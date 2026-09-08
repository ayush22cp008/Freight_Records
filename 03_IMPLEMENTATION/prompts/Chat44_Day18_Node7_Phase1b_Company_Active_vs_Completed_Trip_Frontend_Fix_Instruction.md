# Chat44 — Day 18 — Node 7 — Phase 1b
## Company Active vs Completed Trip Frontend Fix Instruction

**Status:** FIX AUTHORIZED — FRONTEND ONLY
**Portal:** Company
**Finding:** My Created Trips renders active and completed created trips in one flat list.

---

## 1. Authorization Context

This instruction is authorized for the **minimal frontend-only fix** identified by the completed investigation:

`05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Active_vs_Completed_Trip_Investigation_Report.md`

The investigation verified:

- Dashboard `Active Created Trips` already excludes `completed` trips and requires **no query change**.
- My Created Trips intentionally retrieves all trips created by the Company.
- The defect is presentation/information architecture: active/current and completed trips are rendered together in one flat list.
- History correctly contains completed trips.
- No backend, API, database, RLS, auth, or lifecycle-semantic change is required.

Implement only the fix described below.

---

## 2. Source-of-Truth Documents

Before editing, read:

1. `02_ARCHITECTURE/locked_blueprints/Company_Locked_Blueprint.md`
2. `00_PROJECT_CONTROL/DECISIONS/Chat44_Day18_Node7_Phase1b_Company_Implementation_Boundary.md`
3. `03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md`
4. `05_DEBUGGING/investigations/Chat44_Day18_Node7_Phase1b_Company_Active_vs_Completed_Trip_Investigation_Report.md`
5. `03_IMPLEMENTATION/implementation_reports/Chat44_Day18_Node7_Phase1b_Company_Implementation_Report.md`

---

## 3. Exact Target

Primary target:

`src/app/(authenticated)/company/created/page.tsx`

The change must remain frontend-only.

---

## 4. Required Behavior

Partition the already-fetched `createdTrips` array in frontend memory into two presentation groups:

- **Active Trips:** `status !== 'completed'`
- **Completed Trips:** `status === 'completed'`

Render the groups as clearly distinct UI sections.

Recommended presentation structure:

### Active Trips

Show current/non-completed created trips first.

### Completed Trips

Show completed created trips separately as historical/completed work.

The exact visual treatment may follow the existing Company shared design language, but the semantic distinction must be obvious to the user.

Do not remove completed trips from the Company's created-trip history unless the locked Blueprint explicitly requires that behavior. The verified investigation says the Company-created list represents all created trips, while the problem is the lack of visual separation.

---

## 5. Dashboard — DO NOT CHANGE

Do **not** modify the Dashboard's active-trip query/filter as part of this fix.

The verified Dashboard filter is:

`.eq('company_id', company.id).in('status', ['active', 'claimed', 'in_progress', 'draft'])`

It already excludes `completed`.

Do not change this logic merely to make the Dashboard look different.

---

## 6. History — DO NOT CHANGE

Do not change `/company/history` filtering or lifecycle semantics.

Completed trips are expected to remain visible in History.

The fact that a completed trip appears in both:

- My Created Trips → Completed Trips
- History → completed history

is not itself a defect.

---

## 7. Lifecycle Semantics — PROTECTED

Do not alter what `trip.status` means.

Do not introduce new lifecycle states.

Do not automatically transition trips from `claimed` to `completed`.

Do not derive lifecycle state from operational timeline events.

The investigation explicitly established that `trip.status` is the existing lifecycle categorization used by the frontend.

---

## 8. Protected Scope

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
- Driver Portal behavior;
- shared backend behavior.

Do not change API queries merely to implement the UI separation when the existing data is sufficient.

---

## 9. Shared Component Safety

If modifying a shared component is considered necessary, STOP before doing so unless the change is demonstrably Company-safe and does not alter Driver behavior.

Prefer a localized change in:

`src/app/(authenticated)/company/created/page.tsx`

No second navigation, authorization, or role system may be introduced.

---

## 10. Empty-State Behavior

Handle empty groups cleanly.

Examples:

- If there are active trips but no completed trips, show Active Trips without an awkward empty Completed section.
- If there are completed trips but no active trips, clearly show Completed Trips and communicate that there are no active created trips.
- If there are no created trips, preserve an appropriate existing empty state and Create New Trip path.

Do not invent new backend data.

---

## 11. Preserve Existing Functionality

Preserve:

- existing trip data source;
- trip cards/details;
- Create New Trip navigation;
- Trip Detail navigation;
- existing status display;
- driver-claim information;
- payout/distance information;
- Company authentication context;
- existing responsive behavior unless a small local adjustment is necessary for the new sections.

Do not use this task to redesign unrelated Company surfaces.

---

## 12. Build / Test Requirements

After the frontend change:

1. Run the project's appropriate build/type/lint checks.
2. Confirm no protected files were changed.
3. Confirm Dashboard Active Created Trips still excludes `completed`.
4. Confirm My Created Trips visibly separates active and completed trips.
5. Confirm completed trips remain accessible through History.
6. Confirm clicking an active or completed created-trip card still opens the existing Company Trip Detail route.
7. Confirm no Driver Portal behavior was changed.

If any check reveals an API/backend/lifecycle issue, STOP and report rather than expanding scope.

---

## 13. Required Implementation Report

Update/create the implementation report at:

`03_IMPLEMENTATION/implementation_reports/Chat44_Day18_Node7_Phase1b_Company_Implementation_Report.md`

The report must include:

- exact files changed;
- exact frontend behavior changed;
- confirmation that Dashboard query was not changed;
- confirmation that History was not changed;
- confirmation that backend/API/DB/RLS/auth/lifecycle were not changed;
- build/test results;
- manual verification steps for Ayush;
- any remaining UNKNOWNs;
- final implementation status.

Do not claim Company acceptance. Only Ayush can provide manual acceptance.

---

## 14. Stop Conditions

STOP immediately and report if:

- the requested separation cannot be achieved with existing frontend data;
- an API contract must change;
- a backend route must change;
- database/schema/RLS changes appear necessary;
- authentication/authorization must change;
- lifecycle semantics must change;
- claiming behavior must change;
- evidence behavior must change;
- Driver behavior would be affected;
- Reviewer behavior would be affected;
- the locked Blueprint is contradicted;
- a new product behavior is required beyond this presentation separation.

Do not work around a protected boundary.

---

## 15. Final Gate

After implementation and automated checks are complete:

**STOP. Do not perform final acceptance. Do not declare the Company Portal accepted.**

Wait for Ayush to manually verify:

- Dashboard active list;
- My Created Trips active/completed separation;
- History;
- Trip Detail navigation from both active and completed created trips;
- responsive presentation if applicable.

Ayush remains the final manual verification authority.
