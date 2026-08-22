# Freight — Chat6 Node3 Implementation Instruction: Check-in

## Task

Implement **Event 2 — Check-in** as the next event in the locked Freight workflow:

**Arrival → Check-in → Departure**

Arrival is already implemented and personally verified by Ayush. **Do not modify or re-investigate Arrival.**

The Check-in investigation is complete and reports that the feature is ready to implement by reusing the existing Arrival/event infrastructure.

## Source of Truth

Read and follow:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_CheckInSourceReadiness.md`

Also respect the existing Freight project control records and architecture. Do not redesign the workflow.

## Required Check-in Flow

Implement this exact flow:

`Dashboard → Start Check-in → Check-in page → GPS + server timestamp → optional photo → immutable check-in event insert → Check-in Recorded → Dashboard → Start Departure`

After Check-in succeeds, the Dashboard must continue naturally to **Departure** as the next event.

## Implementation Scope

Create only these files unless the existing source proves that a minimal additional change is strictly required:

1. `src/app/(authenticated)/events/checkin/page.tsx`
2. `src/app/(authenticated)/events/checkin/CheckinClient.tsx`
3. `src/app/api/events/checkin/route.ts`

The investigation found that Dashboard already contains the Check-in state and CTA logic, so **do not change Dashboard unless the actual current source contradicts the report**.

## Reuse Existing Infrastructure

Reuse the existing utilities/patterns already used by Arrival:

- `src/lib/capture/getGpsLocation.ts`
- `src/lib/capture/getServerTime.ts`
- `src/lib/capture/uploadPhoto.ts`
- Existing authenticated event page/client patterns.
- Existing Arrival API/server insertion pattern.

Do not duplicate these utilities.

## Functional Requirements

### Check-in UI

- Follow the established Arrival UI pattern and project styling.
- Show that this is the **Check-in** step.
- Capture foreground GPS.
- Obtain authoritative server time.
- Allow an optional photo.
- Allow Check-in to be submitted without a photo.
- Show clear loading/submission state.
- Show success/error feedback consistent with the existing Arrival flow.
- After success, provide the natural path back to the Dashboard.

### Check-in API

Create a server/API route following the existing Arrival route pattern.

Requirements:

- Require authenticated user.
- Resolve the authenticated user to the driver server-side using the established pattern.
- Accept the relevant trip/event evidence payload required by the existing architecture.
- Set `event_type` to **`checkin` server-side**; never trust the client to choose the event type.
- Insert into the existing `events` table using the established server-side Supabase pattern.
- Preserve immutability.
- Handle the existing unique constraint (`trip_id`, `event_type`) cleanly so duplicate Check-in does not create a second event.
- Treat `photo_url` as optional.
- Return clear success/error responses consistent with Arrival.

## Workflow Boundary

The intended product sequence is:

1. Arrival completed.
2. Dashboard shows **Start Check-in**.
3. Driver completes Check-in.
4. Dashboard shows **Check-in Complete** / **Start Departure**.
5. Departure is the next event.

Do not add permanent Navbar shortcuts for Check-in.

Do not implement Departure in this task.

## Explicit Non-Goals

Do NOT:

- Modify Arrival.
- Rewrite existing GPS/server-time/photo utilities.
- Change the Dashboard workflow unless absolutely required by the actual source.
- Change the database schema/migrations.
- Change RLS policies.
- Add new event types.
- Implement Departure.
- Implement Timeline.
- Implement AI Evidence Summary.
- Add multi-stop functionality.
- Fix unrelated security/architecture issues discovered during investigation.
- Change the locked MVP architecture.

The investigation identified missing server-side trip ownership and event-order validation as out-of-scope findings. **Do not expand this task to fix them.**

## Verification Required

Before reporting completion:

1. Run the project's appropriate type/lint/build checks.
2. Verify the Check-in route builds successfully.
3. Verify the API route compiles and follows the existing Arrival pattern.
4. Verify the Dashboard's existing `Start Check-in` link reaches the new Check-in page.
5. Verify successful Check-in inserts `event_type = checkin`.
6. Verify optional photo behavior works without requiring a photo.
7. Verify duplicate Check-in is handled safely by the existing unique constraint.
8. Verify successful Check-in returns the workflow to Dashboard where **Start Departure** is now the next action.
9. Confirm existing Arrival files were not changed unnecessarily.

If manual browser/database verification cannot be completed by Antigravity, report exactly what was verified and what remains for Ayush to manually verify. Do not claim manual verification that was not performed.

## Change Discipline

Keep the implementation minimal and directly derived from the existing Arrival implementation.

Before changing any additional file outside the three target files, explain in the report why it is required. Do not make unrelated changes.

## Required Implementation Report

After implementation, create/update the report in:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_CheckInImplementation.md`

The report must include:

1. Summary of implementation.
2. Exact files added/changed.
3. How the implementation reuses Arrival infrastructure.
4. API behavior.
5. Check-in UI behavior.
6. Validation performed.
7. Build/type/lint results.
8. Manual verification status.
9. Any deviations from this instruction.
10. Remaining work / Ayush manual verification steps.

## Completion Boundary

This task is complete when **Check-in is implemented and passes the available automated checks**, with the Dashboard ready to continue to Departure.

Do not proceed to Departure after completing Check-in.

**Implement Check-in only.**
