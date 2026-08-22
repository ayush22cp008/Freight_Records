# Freight — Chat6 Node3 Implementation Instruction: Departure

## Task

Implement **Event 3 — Departure** as the next and final event in the locked core workflow:

**Arrival → Check-in → Departure**

Arrival and Check-in are already implemented and personally verified by Ayush. **Do not modify or re-investigate them.**

## Source of Truth

Read and follow:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_DepartureSourceReadiness.md`

The readiness investigation confirms that Departure is ready to implement, the Dashboard already routes to `/events/departure`, the database already supports `departure`, and no Dashboard/schema changes are required.

## Required Departure Flow

Implement this exact flow:

`Dashboard → Start Departure → Departure page → GPS + server timestamp + mandatory photo → immutable departure event insert → Departure Recorded → Dashboard → Trip Complete → View Timeline`

Departure is the final core trip event.

## Implementation Scope

Add only these files unless the actual current source proves that a minimal additional change is strictly required:

1. `src/app/(authenticated)/events/departure/page.tsx`
2. `src/app/(authenticated)/events/departure/DepartureClient.tsx`
3. `src/app/api/events/departure/route.ts`

The investigation found that Dashboard already contains the required Departure state/CTA and post-Departure `Trip Complete → View Timeline` transition. **Do not change Dashboard.**

## Reuse Existing Infrastructure

Use the established Arrival/Check-in infrastructure and patterns:

- `src/lib/capture/getGpsLocation.ts`
- `src/lib/capture/getServerTime.ts`
- `src/lib/capture/uploadPhoto.ts`
- Existing authenticated event page patterns.
- Existing Arrival/Check-in client patterns.
- Existing server-side event insertion pattern.

Do not duplicate capture utilities.

## Functional Requirements

### Departure UI

- Follow the established Arrival/Check-in UI pattern and project styling.
- Clearly identify this as the **Departure** step.
- Capture foreground GPS.
- Obtain authoritative server time.
- **Require a photo before submission.**
- Do not allow successful Departure submission without photo evidence.
- Show clear loading/submission state.
- Show success/error feedback consistent with the existing event flows.
- After successful recording, provide the natural path back to Dashboard.

### Departure API

Create the server/API route using the established event route pattern.

Requirements:

- Require authenticated user.
- Resolve authenticated user to driver server-side using the established pattern.
- Accept the required trip/evidence payload.
- Set `event_type` to **`departure` server-side**; never trust the client to choose the event type.
- Insert into the existing `events` table using the established server-side Supabase pattern.
- Preserve event immutability.
- Handle the existing unique constraint (`trip_id`, `event_type`) cleanly so duplicate Departure does not create a second event.
- Require a valid `photo_url` for Departure.
- Return clear success/error responses consistent with Arrival/Check-in.

## Workflow Boundary

The intended product sequence is:

1. Arrival completed. ✅
2. Check-in completed. ✅
3. Dashboard shows **Start Departure**.
4. Driver completes Departure with required photo + GPS + server timestamp.
5. Dashboard shows **Trip Complete**.
6. Dashboard provides **View Timeline**.

Do not implement Timeline in this task.

## Explicit Non-Goals

Do NOT:

- Modify Arrival.
- Modify Check-in.
- Rewrite GPS/server-time/photo utilities.
- Modify Dashboard.
- Change database schema/migrations.
- Change RLS policies.
- Add new event types.
- Implement Timeline.
- Implement AI Evidence Summary.
- Add multi-stop functionality.
- Fix unrelated security/architecture issues discovered during investigation.

The investigation identified missing server-side event-order validation as an existing out-of-scope issue. **Do not expand this task to fix it.**

## Verification Required

Before reporting completion:

1. Run the project's appropriate type/lint/build checks.
2. Verify `npm run build` succeeds with no errors.
3. Verify the Departure route builds successfully.
4. Verify Dashboard's existing **Start Departure** link reaches the new Departure page.
5. Verify Departure cannot be successfully submitted without a photo.
6. Verify Departure captures GPS and server timestamp.
7. Verify successful insertion uses `event_type = 'departure'`.
8. Verify duplicate Departure is handled safely by the existing unique constraint.
9. Verify successful Departure returns to Dashboard.
10. Verify Dashboard changes to **Trip Complete** and shows **View Timeline**.
11. Confirm Arrival and Check-in files were not changed unnecessarily.

If manual browser/database verification cannot be completed by Antigravity, report exactly what was verified and what remains for Ayush to manually verify. Do not claim manual verification that was not performed.

## Change Discipline

Keep the implementation minimal and directly derived from the already-proven Arrival/Check-in patterns.

Before changing any additional file outside the three target files, explain in the report why it is required. Do not make unrelated changes.

## Required Implementation Report

After implementation, create:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_DepartureImplementation.md`

The report must include:

1. Summary of implementation.
2. Exact files added/changed.
3. How the implementation reuses existing event infrastructure.
4. Departure UI behavior.
5. API behavior and validation.
6. Mandatory photo behavior.
7. Build/type/lint results.
8. Manual verification status.
9. Confirmation of Dashboard `Trip Complete` transition.
10. Any deviations from this instruction.
11. Remaining work / Ayush manual verification steps.

## Completion Boundary

This task is complete when **Departure is implemented and passes the available automated checks**, with the Dashboard correctly ready to show **Trip Complete → View Timeline** after a successful Departure.

Do not implement Timeline after completing Departure.

**Implement Departure only.**
