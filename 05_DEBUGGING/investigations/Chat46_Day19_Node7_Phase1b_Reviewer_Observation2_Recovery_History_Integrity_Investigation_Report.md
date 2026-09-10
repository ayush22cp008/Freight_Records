# Reviewer Observation 2 — Recovery History Integrity Investigation Report
**Task:** Chat46 / Day19 / Node 7 / Phase 1b — Reviewer Observation 2 Follow-up
**Date:** 2026-09-10
**Owner:** Antigravity

---

## 1. Investigation Objective
Determine the root cause for why a rejected applicant's recovery submission results in an empty Reviewer Queue while simultaneously corrupting the existing Reviewer History record (showing "No reason recorded" and "No evidence document found"), and verify why the applicant sees no clear success feedback on the UI.

## 2. Exact Manual Observation
As reported by Ayush:
1. The rejected applicant resubmits evidence via the recovery flow.
2. The UI remains on the onboarding flow without obvious success feedback.
3. Reviewer Queue shows "All clear" (empty).
4. Reviewer History shows the applicant as "Rejected", but with "No reason recorded" and "No evidence document found for this record."

## 3. Environment/Runtime Tested
- **Files inspected:** `src/app/api/onboarding/submit/route.ts`, `src/db/migrations/004_create_freight_identities.sql`, `src/db/migrations/005_v2_onboarding_evidence.sql`, `src/app/(authenticated)/reviewer/history/[id]/page.tsx`, `src/app/api/admin/history/route.ts`
- **Context:** Post Observation 2 recovery implementation.

## 4. Original Rejection State
Before recovery, the database holds:
- `freight_identities`: `verification_status = 'REJECTED'`, `reviewed_at = <timestamp>`
- `onboarding_evidence`: `status = 'REJECTED'`, `rejection_reason = <reason string>`

## 5. Recovery Submission Data Flow & Atomicity Assessment
The submission endpoint (`POST /api/onboarding/submit/route.ts`) attempts three operations sequentially:
1. `DELETE` the old `onboarding_evidence` record.
2. `INSERT` the new `PENDING` `onboarding_evidence` record.
3. `UPDATE` the `freight_identities` record to `PENDING` and clear `reviewed_at`.

These operations are performed as independent API calls via the Supabase client without an atomic database transaction. If one fails, the system enters a partially-written inconsistent state.

## 6. Authentication/Authorization Assessment (RLS Root Cause)
The `supabase` client instantiated in `api/onboarding/submit/route.ts` is an authenticated user client bound by Row Level Security (RLS).
1. **Evidence Deletion (Failed):** The `onboarding_evidence` table has RLS enabled but **lacks a `DELETE` policy** for users (`005_v2_onboarding_evidence.sql`). Therefore, the `.delete()` operation fails silently (deleting 0 rows). The old `REJECTED` evidence is preserved.
2. **Evidence Insertion (Succeeded):** The user has `INSERT` rights, so the new `PENDING` evidence is successfully inserted. The table now contains **two rows** for this `auth_id`.
3. **Identity Update (Failed):** The `freight_identities` table has RLS enabled but **lacks an `UPDATE` policy** for users (`004_create_freight_identities.sql`). The attempt to change `verification_status` to `PENDING` throws an RLS permission error.

## 7. Submission Success/Failure Behavior
Because the identity update throws a permission error, the API catches it and returns a `500 Internal Server Error`.
Consequently, the frontend UI does not transition to a success state or redirect. The user appears trapped on the onboarding form, explaining Observation #2.

## 8. Reviewer Queue Data Path
Because the identity `UPDATE` failed, `freight_identities.verification_status` remains `REJECTED`. 
The Reviewer Queue (`reviewer/queue/page.tsx`) explicitly queries for `PENDING` identities. Thus, the applicant never re-enters the queue, explaining Observation #3.

## 9. Verification History Data Path
Because the status is still `REJECTED`, the applicant correctly continues to appear in the Reviewer History (`api/admin/history/route.ts`).
However, when the Reviewer clicks into the History detail page (`history/[id]/page.tsx`), the page queries the evidence:
```typescript
  const { data: evidence } = await supabaseServer
    .from('onboarding_evidence')
    .select('*')
    .eq('auth_id', identity.auth_id)
    .single();
```
Because the `DELETE` failed and the `INSERT` succeeded, there are now **two rows** for this applicant. The `.single()` function enforces that exactly one row must be returned. It throws a `PGRST116: The result contains 2 rows` error, returning `data: null`.

Because `evidence` evaluates to `null`:
1. The UI fallback renders "No reason recorded."
2. `EvidenceViewerClient` receives `null` and renders "No evidence document found for this record."
This explains Observation #4 perfectly.

## 10. Rejection Reason Preservation Behavior
The original rejection reason is **preserved**. It was not deleted. It simply cannot be displayed because the `.single()` query fails due to the duplicate row created by the partial-write failure.

## 11. Evidence Classification
- Partial-write failure due to RLS limitations: **VERIFIED** (Source/DB inspection)
- `onboarding_evidence` missing DELETE policy: **VERIFIED**
- `freight_identities` missing UPDATE policy: **VERIFIED**
- API returns 500 on status update: **VERIFIED**
- History UI `.single()` failure on duplicate rows: **VERIFIED**
- Original rejection reason preservation: **VERIFIED**

## 12. Root Cause
The root cause is an **RLS-induced partial-write failure combined with strict query expectations**.
The submission API attempts mutations that the user's RLS policies do not permit (`DELETE` evidence, `UPDATE` identity). The API fails halfway through, leaving the database with duplicate evidence rows and a stale `REJECTED` status. The duplicate evidence rows crash the Reviewer History detail page's `.single()` query, masking the preserved historical data.

## 13. Contributing Factors
1. **Lack of transactions:** Supabase REST data API calls execute independently.
2. **UI Error Handling:** The frontend form swallows the 500 error instead of showing a clear failure message.

## 14. Ruled-out Hypotheses
- **Data deletion:** The old evidence and rejection reason were NOT deleted or overwritten. They still exist in the database.
- **Queue query bug:** The queue query is correct; the data simply never transitioned to `PENDING`.
- **History query bug:** The history list query works correctly, only the detail page `.single()` fetch fails.

## 15. Recovery/History Compatibility Assessment
The current recovery approach (deleting old evidence) fundamentally contradicts the Reviewer History's implicit requirement for immutable historical decisions. If the old evidence row is successfully deleted, the historical record loses the rejection reason and the linked document. The architecture currently couples the "current" evidence state directly to the historical decision, which means recovering an application destructively mutates its history.

## 16. Recommended Fix Scope
A robust correction likely requires:
1. **State-transition correction (Service Role):** The `api/onboarding/submit/route.ts` must use `supabaseServer` (service role) to bypass RLS for the state transition, or the RLS policies must be explicitly expanded.
2. **Reviewer History model correction:** If the system is to maintain an immutable history, the `onboarding_evidence` schema might need a redesign (e.g., maintaining a history table or querying the specific evidence version) rather than simply calling `.delete()` on old rows.
3. **Transaction correction:** Use a stored Postgres RPC function to atomically swap the evidence and update the identity status.

## 17. Final Investigation Status
Investigation complete. Root cause verified as an RLS-induced partial-write failure. No code has been modified. Awaiting governance decision.
