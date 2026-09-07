# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Case C Scene 1 Query Fix Implementation Prompt

**Status:** READY FOR IMPLEMENTATION
**Portal:** Driver
**Phase:** Node 7 — Phase 1b
**Scope:** Focused frontend/server-query defect correction only

## 1. Objective

Implement ONLY the verified Case C — Scene 1 query fix identified in:

`05_DEBUGGING/investigations/Chat43_Day17_Node7_Phase1b_Driver_Case_C_Scene1_Not_Appearing_Resolution_Report.md`

The goal is to restore the existing Case C Scene 1 behavior on **My Active Trip** after a completed trip is no longer active.

Do not redesign the Case C UX. The locked V2.1 one-time Scene 1 acknowledgement behavior remains the governing UX contract.

## 2. Verified Root Cause

In:

`src/app/(authenticated)/driver/active/page.tsx`

the fallback query for the most recently completed Driver trip currently attempts to select and order by `updated_at`:

```typescript
const { data: lastCompleted } = await supabaseServer
  .from('trips')
  .select('id, destination_name, updated_at')
  .eq('driver_id', driverId)
  .eq('status', 'completed')
  .order('updated_at', { ascending: false })
  .limit(1)
  .maybeSingle();
```

The verified database error is:

`42703: column trips.updated_at does not exist`

Because the query result's `error` is not surfaced and `data` becomes `null`, the existing `RecentCompletionBanner` never receives a completed trip and Scene 1 is skipped.

The investigation verified that `receiver_delivery_confirmed_at` is an existing usable field and that ordering by it successfully returns the expected completed trip. fileciteturn239file0

## 3. Required Fix

Modify ONLY the fallback completed-trip query in:

`src/app/(authenticated)/driver/active/page.tsx`

Replace the invalid `updated_at` selection/order with:

```typescript
const { data: lastCompleted } = await supabaseServer
  .from('trips')
  .select('id, destination_name, receiver_delivery_confirmed_at')
  .eq('driver_id', driverId)
  .eq('status', 'completed')
  .order('receiver_delivery_confirmed_at', { ascending: false, nullsFirst: false })
  .limit(1)
  .maybeSingle();
```

### Important

- Preserve the existing `!trip` fallback structure.
- Preserve `RecentCompletionBanner` behavior.
- Preserve the exact completed-trip `tripId` used by the banner's timeline link.
- Preserve the one-time browser acknowledgement mechanism already implemented.
- Do not change the Scene 1 wording unless required by the existing implementation.
- Do not change the normal Scene 2 empty state.
- Do not change completion-page behavior.
- Do not change Timeline behavior.

## 4. Required Runtime/Error Handling Check

During implementation, inspect the surrounding query code for accidental swallowing of query errors.

Do NOT redesign error handling globally.

If a minimal local check is appropriate, it may be added only if it does not change user-facing behavior or protected boundaries. Otherwise, leave the existing error-handling pattern unchanged and report any limitation.

Do not hide a new query failure by assuming `lastCompleted === null` means there was no completed trip.

## 5. Protected Boundaries

Do NOT modify:

- `/api/completion/driver`
- any API contract
- database schema or migrations
- Supabase RLS/security policies
- authentication or role rules
- trip lifecycle semantics
- dual-confirmation completion logic
- claiming/marketplace behavior
- evidence model or evidence requirements
- Timeline data model or timeline access rules
- `RecentCompletionBanner` acknowledgement semantics
- `TimelineAcknowledgement` behavior
- Company portal
- Reviewer portal
- unrelated Driver UI

This fix is specifically authorized by the completed Case C investigation and requires **0 DB schema changes** and **0 API changes**. fileciteturn239file0

## 6. V2.1 UX Contract That Must Remain Intact

The query correction must restore the existing intended flow:

```text
Receiving Company confirms
        ↓
trip.status = completed
        ↓
Driver has no active trip
        ↓
lastCompleted is found
        ↓
RecentCompletionBanner evaluates acknowledgement
        ↓
Not acknowledged → Scene 1
        ↓
View Recent Trip Timeline
        ↓
Timeline opens for exact tripId
        ↓
Acknowledgement recorded in browser
        ↓
Return to My Active Trip
        ↓
Scene 1 gone → normal Scene 2
```

The completed trip itself must remain accessible after acknowledgement. Only the Case C notification is dismissed. This is part of the locked V2.1 plan. fileciteturn236file0

## 7. Verification — Mandatory

### A. Query/Data Verification

1. Confirm the previously failing `updated_at` query is no longer used for the Case C fallback.
2. Confirm the fallback query uses `receiver_delivery_confirmed_at`.
3. Confirm the query returns the expected completed trip for the Driver.
4. Confirm no Supabase `42703` error occurs.

### B. Case C — Scene 1

5. Use a test trip that has:
   - Driver completion confirmed.
   - Receiving Company completion confirmed.
   - `status = completed`.
6. Ensure the Driver has NOT previously viewed that trip's timeline in the current browser, or clear only the relevant acknowledgement key for the test trip.
7. Navigate directly to or refresh `/driver/active`.
8. Verify Scene 1 appears:
   - **No Active Trip**
   - "Your recent delivery to [destination] has been fully completed."
   - **View Recent Trip Timeline**
9. Confirm the CTA contains the exact completed `tripId`.

### C. Timeline Acknowledgement

10. Click **View Recent Trip Timeline**.
11. Verify the exact completed trip timeline opens.
12. Verify `acked_completed_trip_<tripId>` is created only after the timeline is rendered/viewed.
13. Return to `/driver/active` or refresh.
14. Verify Scene 1 is gone and normal Scene 2 appears.

### D. Other Entry Point

15. Open the same completed trip through the completion page's **View Timeline** CTA.
16. Verify that this valid timeline entry point also acknowledges Scene 1 for the same `tripId`.
17. Verify the completed timeline remains accessible repeatedly afterward.

### E. Regression / Build

18. Verify waiting-state navigation still uses **View Completion Status** → `/completion/driver?tripId=...`.
19. Verify fully completed trips remain absent from Active Trip.
20. Verify no unrelated Driver behavior changed.
21. Run:

```bash
npm run build
```

22. Check for runtime/console errors related to the changed query or Case C flow.

## 8. Required Implementation Report

After implementation, create/update the implementation report in:

`03_IMPLEMENTATION/implementation_reports/`

The report must include:

- Files changed.
- Exact query correction.
- Confirmation that `updated_at` was removed from this fallback query.
- Confirmation that `receiver_delivery_confirmed_at` is used for selection/order.
- Build result.
- Runtime verification result.
- Case C Scene 1 result.
- Timeline acknowledgement result.
- Scene 2 post-acknowledgement result.
- Regression findings.
- Screenshots/evidence where useful.
- Any limitations.
- Explicit `VERIFIED / INFERRED / UNKNOWN` classification.

Do not claim Case C is fixed solely because the build passes.

## 9. Stop Conditions

Stop and report back immediately if:

- the proposed query cannot be executed against the current schema;
- `receiver_delivery_confirmed_at` is not available as verified;
- the fix requires a DB/schema change;
- the fix requires an API change;
- RLS/authentication behavior must change;
- lifecycle semantics must change;
- the acknowledgement mechanism must be redesigned;
- the exact completed trip cannot be safely identified;
- an unexpected architectural dependency appears.

Do not silently cross a protected boundary.

## 10. Final Implementation Gate

**Implementation decision:** READY FOR IMPLEMENTATION.

**Source resolution:**
`05_DEBUGGING/investigations/Chat43_Day17_Node7_Phase1b_Driver_Case_C_Scene1_Not_Appearing_Resolution_Report.md`

**Governing UX plan:**
`05_DEBUGGING/investigations/Chat43_Day17_Node7_Phase1b_Driver_Final_Completion_Implementation_Plan_V2.md` (V2.1 content)

**Scope:** One focused Case C query correction only.

No Company or Reviewer work is authorized by this prompt.
