# Driver P1 Fixes Implementation Plan

This plan addresses the three P1 UI discrepancies identified in the post-implementation investigation, keeping entirely within the frontend boundary.

## Proposed Changes

### 1. Fix "Profile Not Found"
The `/profile` route fails because it was implemented as a Client Component using `supabase/client`, which is blocked by RLS from querying the `drivers` table.

#### [MODIFY] `page.tsx` (in `/profile`)
- Convert to a Server Component.
- Replace `createClient` (client-side) with `supabaseServer` and server-side auth retrieval.
- Remove `useState` and `useEffect`. 
- Ensure it mirrors the exact data fetching pattern that was proven to work in the original monolithic `page.tsx`.

### 2. Demonstrate Delivery Progress
The `/driver/active` route currently only shows the "Current Status" and "Next Required Action" but lacks a visual checklist of completed vs. remaining stages.

#### [MODIFY] `page.tsx` (in `/driver/active`)
- Map the retrieved `eventTypes` array against the canonical Node 5 delivery stages (Arrival at Pickup, Check-in, Goods Loaded, etc.).
- Render a vertical progress checklist or step indicator showing which stages are completed, which is current, and which are upcoming.

### 3. Demonstrate Evidence Status
The active trip UI needs to reflect whether required evidence (photos) has been collected, using existing recorded evidence.

#### [MODIFY] `page.tsx` (in `/driver/active`)
- Update the Supabase query to `select('event_type, photo_url')` from the `events` table.
- Within the Delivery Progress checklist (from Fix 2), indicate if a `photo_url` is present for the completed stages that require it (like Check-in and Goods Loaded).
- Add an "Evidence Status" summary block showing how many evidence photos have been collected so far.

## Verification Plan

- Check `/profile` while authenticated as a driver to ensure Name and Email are rendered correctly.
- Check `/driver/active` to verify the new visual Delivery Progress checklist and Evidence Status appear without causing frontend errors.
- Confirm no backend APIs, schema, or RLS policies were touched.
