# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Completed Trip Timeline Wrong-Trip Resolution Report

## 1. Investigation Conclusion

The investigation into the Timeline navigation issue has successfully identified the root cause. When a driver clicks `View Timeline` on a historical completed trip, the correct URL is generated (e.g. `/timeline?tripId=...`), but the Timeline page ignores the `tripId` and instead loads the driver's most recently created trip (e.g., `navsari`). 

This is a **Timeline selection regression** caused by a compatibility issue with Next.js 15 route parameters.

## 2. Root Cause Analysis

In `src/app/(authenticated)/driver/history/page.tsx`, the `View Timeline` link correctly generates the expected URL:
```tsx
href={`/timeline?tripId=${ct.id}`}
```

However, in `src/app/(authenticated)/timeline/page.tsx`, the `searchParams` prop is consumed synchronously:
```typescript
type Props = {
  searchParams?: { [key: string]: string | string[] | undefined };
};

export default async function TimelinePage({ searchParams }: Props) {
  // ...
  const tripId = searchParams?.tripId;
```

**Why this fails:**
The project uses `Next.js 16.3.1` (App Router). In modern Next.js versions (15+), `searchParams` provided to page components are **Promises**, not plain synchronous objects. 

Because `searchParams` is a Promise, `searchParams?.tripId` resolves to `undefined`. 

With `tripId` evaluating to `undefined`, the code executes its fallback path:
```typescript
  if (typeof tripId === 'string') {
    query = query.eq('id', tripId);
  } else {
    // If no specific trip is requested, get the most recent active/claimed/in_progress/completed trip
    query = query.order('created_at', { ascending: false }).limit(1);
  }
```
This fallback queries the database for the driver's most recent trip (`order('created_at', { ascending: false })`), which happened to be the `navsari` trip in the manual test, completely ignoring the historical trip ID in the URL.

## 3. Boundary Verification
- **History Link:** VERIFIED. The `history/page.tsx` correctly embeds the historical `tripId`.
- **Security Boundary:** VERIFIED. The `timeline/page.tsx` query correctly retains the `.eq('driver_id', driver.id)` ownership constraint. The failure does not bypass tenant security; it merely selects a different trip owned by the same driver.
- **Other Pages:** The `completion/driver/page.tsx` correctly awaits `searchParams` (`const { tripId } = await searchParams;`), which is why the completion flow did not experience this regression.

## 4. Recommendation for Fix
The fix requires a narrowly scoped modification in `src/app/(authenticated)/timeline/page.tsx` to properly await the `searchParams` Promise before extracting the `tripId`.

**Proposed Fix:**
Change the type signature and consumption in `timeline/page.tsx`:
```typescript
type Props = {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
};

export default async function TimelinePage({ searchParams }: Props) {
  // ...
  const resolvedParams = await searchParams;
  const tripId = resolvedParams?.tripId;
```

This ensures the `tripId` is correctly extracted from the URL and injected into the `.eq('id', tripId)` Supabase query, restoring exact historical trip selection while preserving the authenticated driver ownership constraint.

## 5. Decision Status
**STATUS: INVESTIGATION COMPLETE — READY FOR IMPLEMENTATION**
The investigation confirms the issue is a Next.js 15+ prop consumption defect. A focused implementation prompt can now authorize this fix without risking any protected backend boundaries.
