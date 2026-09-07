# Chat43 — Day 17 — Node 7 — Phase 1b Driver Case C Scene 1 Not Appearing Resolution Report

## 1. Investigation Conclusion

The investigation into why Case C (Scene 1) fails to appear after a trip completes has successfully identified the root cause. The issue is a **ROOT CAUSE A — Data/query failure**.

## 2. Root Cause Analysis

In `src/app/(authenticated)/driver/active/page.tsx`, when no active trip is found, a fallback query attempts to fetch the most recently completed trip for the Driver:

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

However, the `trips` table **does not have an `updated_at` column**. This causes the Supabase query to fail with the following PostgreSQL error:
`42703: column trips.updated_at does not exist`

Because the Next.js server code only destructured `data` and ignored the `error` object (`const { data: lastCompleted }`), the failure was silent. `lastCompleted` was evaluated as `null`, preventing the `RecentCompletionBanner` from ever being rendered.

## 3. Evidence 

When manually executing the equivalent query via a test script against the active database:
```javascript
Error: {
  code: '42703',
  hint: 'Perhaps you meant to reference the column "trips.created_at".',
  message: 'column trips.updated_at does not exist'
}
Data: null
```

When modifying the script to order by `receiver_delivery_confirmed_at` instead of `updated_at`:
```javascript
Error: null
Data: {
  id: 'dde50a7a-b896-4a2b-ad48-728b5b924f53',
  destination_name: 'punjab',
  receiver_delivery_confirmed_at: '2026-09-07T18:43:16.331+00:00'
}
```
The modified query successfully returned the expected completed trip.

## 4. Verification of Other Boundaries
- **Render-path (Q3):** The `RecentCompletionBanner` component logic is sound. The initial state default of `true` successfully prevents hydration mismatches, and it correctly renders if a valid trip object is passed.
- **Premature Acknowledgement (Q4/Q5):** The client-side logic correctly checks for `acked_completed_trip_<tripId>`. Because the banner was never mounted, it was impossible for the UI to be acknowledged.
- **Hydration/state (Q6):** No issues found. The banner gracefully degrades to `null` initially and sets state within `useEffect`.

## 5. Recommendation for Fix
The fallback query in `src/app/(authenticated)/driver/active/page.tsx` should be updated to select and sort by `receiver_delivery_confirmed_at` rather than the non-existent `updated_at` column. 

**Proposed Query Fix:**
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
This requires exactly **0** database schema changes, **0** API changes, and strictly preserves all Node 5 semantics and boundaries. 

## 6. Decision Status
**STATUS: INVESTIGATION COMPLETE — READY FOR IMPLEMENTATION**
