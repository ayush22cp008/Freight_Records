# Chat5 Node3 — Implementation Plan: Fix Hub Navigation + State Foundation

## Goal Description
Transform the Trip Hub (`/`) into the single source of truth for the active trip's workflow state. The Hub will query authoritative database state (the `events` table) to determine progress (Arrival -> Check-in -> Departure) and present a single primary call-to-action (CTA) for the next required event. We will also implement server-side protection on the `/events/arrival` route to prevent duplicate arrivals.

## Proposed Changes

### Dashboard (Hub)
#### [MODIFY] `src/app/page.tsx`
- Update `Home` server component to fetch the active trip for the driver from the `trips` table.
- If an active trip exists, fetch all associated events for that `trip_id` from the `events` table.
- Determine the current state based on the presence of events:
  - **No Arrival**: Display "Arrival Pending", CTA: "Start Arrival" (`/events/arrival`)
  - **Arrival exists, no Check-in**: Display "Arrival Complete", CTA: "Start Check-in" (`/events/checkin`)
  - **Arrival & Check-in exist, no Departure**: Display "Check-in Complete", CTA: "Start Departure" (`/events/departure`)
  - **All three exist**: Display "Trip Complete", CTA: "View Timeline" (`/timeline`)
- Add basic UI showing the trip details (e.g., Facility Name) and a progress indicator.

### Arrival Event Route
#### [MODIFY] `src/app/events/arrival/page.tsx`
- After fetching the active trip, also query the `events` table for this `trip_id`.
- If an event with `event_type === 'arrival'` already exists, call Next.js `redirect('/')` to bounce the user back to the correct Hub state. This enforces duplicate protection at the page level (API level is already protected by DB constraints).

## Open Questions
- The instruction specifies to only provide the final navigation target for Check-in, Departure, and Timeline, without implementing them yet. Therefore, clicking these buttons will currently result in a 404 until those features are built in a future step.

## Verification Plan
### Manual Verification
- Log in with a valid driver.
- Verify the Hub shows the active trip and "Start Arrival".
- Click "Start Arrival", submit an arrival event.
- Return to the Hub. Verify the Hub now shows "Start Check-in".
- Attempt to navigate manually to `/events/arrival` via URL. Verify it instantly redirects back to `/`.
