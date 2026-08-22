# Freight — Chat6 Node3 Implementation Instruction: Timeline

## Task

Implement the **Timeline** feature as the read-only historical view of the completed Freight trip.

The verified core workflow is:

**Arrival → Check-in → Departure → Trip Complete → View Timeline**

Arrival, Check-in, and Departure are already implemented and personally verified by Ayush. **Do not modify or re-investigate them.**

## Source of Truth

Read and follow:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_TimelineSourceReadiness.md`

The investigation confirms:

- Dashboard already routes `View Timeline` to `/timeline`.
- No Timeline implementation currently exists.
- The existing `events` table contains the required event evidence.
- No database/schema change is required.
- Timeline can be implemented as a Server Component using the existing authenticated Supabase server access pattern.
- The smallest safe implementation surface is one new file:
  `src/app/(authenticated)/timeline/page.tsx`

## What Timeline Means

Timeline is a **read-only chronological history of the trip's recorded events**.

It should show the factual sequence:

**Arrival → Check-in → Departure**

using the evidence already stored in the `events` table.

Timeline does NOT create, edit, delete, or modify events.

## Required Timeline Content

For the authenticated driver's completed/current trip, retrieve the relevant events for the correct `trip_id` and display them in chronological order using `server_timestamp ASC`.

For each recorded event, display the available factual evidence:

- Event type: Arrival / Check-in / Departure
- Server timestamp
- GPS latitude/longitude
- Photo evidence when `photo_url` exists

Do not invent information that is not stored in the event.

## Implementation Scope

Add only:

`src/app/(authenticated)/timeline/page.tsx`

Do not change Dashboard or any existing Arrival, Check-in, or Departure files.

Do not add an API route unless the actual source proves it is strictly required. The investigation indicates a Server Component can query Supabase directly using the established SSR pattern.

## Data Access

Reuse the established authenticated server-side pattern:

1. Resolve the authenticated user from the existing server-side Supabase session.
2. Resolve that user to the appropriate driver using the existing `auth_id` relationship.
3. Resolve the driver's relevant trip using the established project pattern.
4. Query `events` for that specific `trip_id`.
5. Order by `server_timestamp ASC`.

Do not accept a client-supplied `trip_id` for authorization purposes.

Do not expose another driver's events.

## UI Requirements

Keep the UI simple and consistent with the existing Freight application.

The page should clearly communicate that this is the **Trip Timeline**.

Render the events in a clear chronological layout. A simple vertical timeline/list is sufficient; do not over-design it.

For each event:

- clearly label the event
- show the server timestamp
- show latitude/longitude
- show the photo when available
- preserve the factual order

Handle an event without a photo gracefully.

If no relevant events are found, show a clear empty state rather than fabricated data.

## Important Boundary: Read-Only

Timeline must NOT:

- create events
- update events
- delete events
- modify GPS data
- modify timestamps
- modify photos
- modify trip state

It is strictly a **display/read feature**.

## Explicit Non-Goals

Do NOT:

- Modify Arrival.
- Modify Check-in.
- Modify Departure.
- Modify Dashboard.
- Change the events schema.
- Add migrations.
- Change RLS policies.
- Add new event types.
- Implement AI Evidence Summary.
- Add multi-stop functionality.
- Add editing/deletion controls.
- Add unrelated features.
- Fix unrelated architecture/security issues.

## Verification Required

Before reporting completion:

1. Run the project's appropriate type/lint/build checks.
2. Verify `npm run build` succeeds with no errors.
3. Verify `/timeline` now resolves instead of returning 404.
4. Verify the page is accessible through the existing Dashboard `View Timeline` CTA.
5. Verify the authenticated trip's events are displayed in chronological order.
6. Verify Arrival appears before Check-in and Check-in before Departure when all three exist.
7. Verify timestamps shown come from the stored server timestamps.
8. Verify GPS values shown come from the stored event data.
9. Verify photos are displayed only when `photo_url` exists.
10. Verify no event mutation is performed by the Timeline page.
11. Confirm no existing event/Dashboard files were changed unnecessarily.

If manual browser verification cannot be completed by Antigravity, report exactly what was verified and what remains for Ayush to manually verify. Do not claim manual verification that was not performed.

## Change Discipline

The investigation identified a one-file implementation surface. Keep it that way unless the actual source proves another file is strictly required.

If any additional file must be changed, explain exactly why in the implementation report before considering the task complete.

## Required Implementation Report

After implementation, create:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_TimelineImplementation.md`

The report must include:

1. Summary of implementation.
2. Exact files added/changed.
3. Data-access/query behavior.
4. Timeline UI behavior.
5. Confirmation that the view is read-only.
6. Build/type/lint results.
7. Manual verification status.
8. Any deviations from this instruction.
9. Remaining work / Ayush manual verification steps.

## Completion Boundary

This task is complete when **Timeline is implemented, `/timeline` resolves successfully, the stored trip events are displayed chronologically, and automated checks pass**.

Do not proceed to AI Evidence Summary after completing Timeline.

**Implement Timeline only.**
