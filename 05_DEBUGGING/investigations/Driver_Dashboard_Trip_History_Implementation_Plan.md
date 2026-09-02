# Driver Dashboard Trip History UX Implementation Plan

## 1. Goal
Implement a structural update to the Driver Dashboard so that it clearly displays "Available Trips", "My / Active Trip", and "Past / Completed Trips" natively without altering the locked trip-claim authorization model or the canonical event lifecycle.

## 2. Answers to Implementation Questions
1. **Fields to display:** Facility name, destination name, payout, distance, and duration.
2. **Link behavior:** Yes, completed trips will link to the read-only `/timeline?tripId=<id>`.
3. **Pagination/Count:** Load the top 10 historical trips ordered by `created_at` descending.
4. **Non-completed historical states:** Ignored for this scope. We only query `.eq('status', 'completed')`.
5. **Ownership protection:** Query uses `.eq('driver_id', driverId)` explicitly relying on the server-resolved driver identity.
6. **Component splitting:** Kept within `page.tsx` for simplicity to avoid redesigning the current Next.js RSC boundary right now.
7. **Company/Reviewer behavior:** Remains completely unchanged.

## 3. Proposed Changes

### `src/app/(authenticated)/page.tsx`
We will fetch the completed trips for the authenticated driver and display them in a dedicated section at the bottom of the driver dashboard view. 

#### [MODIFY] page.tsx
- Add a new Supabase query to fetch the driver's historical trips:
  ```typescript
  const { data: completedTrips } = await supabaseServer
    .from('trips')
    .select('id, facility_name, destination_name, distance, duration, payout')
    .eq('driver_id', driverId)
    .eq('status', 'completed')
    .order('created_at', { ascending: false })
    .limit(10);
  ```
- **Structural Update:** We will reorganize the driver's layout returning block so that the dashboard always renders:
  1. **Top Section:** Renders "My / Active Trip" (if an active/in-progress trip exists).
  2. **Top Section (Alternative):** Renders "Available Trips" (if no active trip exists, fetching published trips as it does now).
  3. **Bottom Section:** Renders "Past / Completed Trips" showing the `completedTrips` list. Each completed trip will show basic details (pickup, dropoff, payout) and a CTA link to "View Timeline".

## 4. Verification Plan

### Automated Tests
- Run `npx tsc --noEmit` to ensure type safety.

### Manual Verification
- Driver with published trips and no active trip → Available Trips shown + Completed Trips shown.
- Driver with an active trip → My / Active Trip shown + Completed Trips shown.
- Driver with completed history → Past / Completed Trips populated, ordered newest-first.
- Existing claim flow and workflow remains unaffected.
