# Node 3 — Claimed → Start Arrival Fix

## Objective

Fix and verify the Node 3 transition where a driver who has successfully claimed a published trip can click **Start Arrival** and reach the arrival event flow without receiving **“No active trip found.”**

## Current confirmed state

The following flow is already working and must not be regressed:

1. Company creates a trip draft.
2. Company publishes the trip.
3. Published trip becomes visible to an eligible driver.
4. Driver clicks **Claim Trip**.
5. Driver reaches the active-trip state showing **“Trip Claimed - Arrival Pending”**.
6. The dashboard presents **Start Arrival**.

## Problem to investigate

Clicking **Start Arrival** currently reaches `/events/arrival`, but in the previously observed test it displayed:

> No active trip found

The implementation must determine why the arrival page cannot resolve the driver's currently claimed/active trip, even though the authenticated driver's dashboard can resolve it.

## Required investigation

Inspect the current production implementation before changing anything. Trace the complete data path for:

- authenticated Supabase user
- `drivers.auth_id`
- driver record / driver ID
- `trips.driver_id`
- trip status (`claimed`, `active`, `in_progress`, etc.)
- `/events/arrival` page/server logic
- arrival-event insertion logic
- any RLS policies involved in reading the active trip or inserting the arrival event

Compare the active-trip lookup used by the authenticated dashboard with the lookup used by `/events/arrival`.

## Required behavior

For a driver with a successfully claimed trip and no arrival event:

**Claim Trip**
→ trip is assigned to that driver
→ dashboard shows **Trip Claimed - Arrival Pending**
→ click **Start Arrival**
→ `/events/arrival` resolves the same active trip
→ driver can submit/start the arrival event
→ arrival event is persisted against the correct `trip_id`
→ driver returns to the dashboard
→ dashboard shows the next state, **Arrival Complete / Start Check-in**.

## Security requirements

- Never identify the active trip from a client-supplied arbitrary `trip_id` alone.
- Resolve the trip from the authenticated Supabase user and the driver's authorized record.
- Do not expose another driver's trip.
- Preserve existing RLS/security boundaries.
- Do not use the Supabase service-role key in client-side code.
- Do not weaken or bypass RLS merely to make the page work.

## Scope restrictions

Do **not** change:

- authentication or password recovery
- company trip creation
- published-trip visibility
- Claim Trip behavior, unless the investigation proves it is the direct cause
- reviewer authorization/routing
- unrelated UI styling
- later check-in/departure workflow

Make the smallest correct implementation change necessary for the Node 3 transition.

## Verification checklist

### Automated

- Run lint/build/tests available in the project.
- Confirm no TypeScript/build errors.

### Manual production verification

Using a real test driver account and a published test trip:

1. Confirm the trip is visible under Available Trips.
2. Click **Claim Trip**.
3. Confirm **Trip Claimed - Arrival Pending** appears.
4. Click **Start Arrival**.
5. Confirm `/events/arrival` does **not** show “No active trip found”.
6. Submit/start arrival.
7. Confirm the arrival event is created for the claimed trip.
8. Confirm the dashboard advances to **Arrival Complete / Start Check-in**.
9. Confirm another driver cannot access the trip through the arrival flow.

## Completion evidence

Record:

- root cause
- files changed
- database/RLS changes, if any
- automated verification results
- production/manual verification result
- commit SHA

Do not mark Node 3 complete until the end-to-end **Claimed → Start Arrival → Arrival Complete** path has been manually verified.
