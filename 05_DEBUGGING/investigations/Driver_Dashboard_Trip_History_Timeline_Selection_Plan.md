# Timeline Trip Selection Implementation Plan

## 1. Goal
Fix the Timeline page (`src/app/(authenticated)/timeline/page.tsx`) so that it can display the correct historical completed trip when navigated to via `/timeline?tripId=<id>`, without breaking the existing active trip workflow or exposing trips across drivers.

## 2. Approach

We will modify `TimelinePage` to accept Next.js `searchParams` and use the `tripId` to precisely select the trip if provided.

### `src/app/(authenticated)/timeline/page.tsx`
- **[MODIFY]** Update the component signature to accept Next.js Page props:
  ```typescript
  type Props = {
    searchParams?: { [key: string]: string | string[] | undefined };
  };
  export default async function TimelinePage({ searchParams }: Props) {
  ```
- **[MODIFY]** Update the Supabase trip query to dynamically apply `.eq('id', tripId)` if the `tripId` is provided, while still enforcing `driver_id = driver.id` to prevent unauthorized access.
  ```typescript
  const tripId = searchParams?.tripId;

  let query = supabaseServer
    .from('trips')
    .select('id, facility_name')
    .eq('driver_id', driver.id)
    .in('status', ['active', 'claimed', 'in_progress', 'completed']);

  if (typeof tripId === 'string') {
    query = query.eq('id', tripId);
  } else {
    // If no specific trip is requested, we need to ensure we only get the most recent/active one
    // to avoid .single() failing when there are multiple historical completed trips.
    query = query.order('created_at', { ascending: false }).limit(1);
  }

  const { data: trip } = await query.single();
  ```

## 3. Security & Boundaries
- The query inherently enforces `driver_id = driver.id` using the server-resolved authenticated driver identity, protecting against ID manipulation.
- Node 4 claiming and Node 5 lifecycles are entirely avoided and unchanged.
- The dashboard code remains untouched.

## 4. Verification Plan
- **Automated:** Run `npx tsc --noEmit`
- **Manual Verification (by Ayush):**
  - Verify that navigating from the Dashboard's "Past / Completed Trips" CTA correctly loads the exact historical trip.
  - Verify that if a different driver's `tripId` is inserted into the URL, it safely returns "No active trip found."
  - Verify the existing active trip CTA (`/timeline` without a trip ID) still loads the driver's current trip.
