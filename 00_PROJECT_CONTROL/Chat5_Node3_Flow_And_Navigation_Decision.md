# Chat5 Node3 — Flow & Navigation Decision

**Status:** LOCKED DECISION
**Decision basis:** Current source audit + project roadmap + Claude architecture review

## 1. Scope decision

For the current hackathon MVP, keep the **single-facility, fixed 3-event evidence workflow**.

Do NOT redesign the current MVP into a multi-stop architecture at this stage.

The broader product vision may later support a complete multi-stop driver journey, but that is deferred until the current end-to-end MVP is working and only reconsidered if genuine slack remains.

## 2. Current MVP journey

```text
Login
  ↓
Trip Hub (/)
  ↓
Arrival
  ↓
Check-in
  ↓
Departure
  ↓
Timeline
  ↓
AI Evidence Summary
```

The purpose is to create verifiable evidence that the driver reached the facility, checked in, and later departed.

## 3. Navigation principle

The **Trip Hub (`/`) is the single source of truth for the driver's current workflow state and next required action.**

The Hub must determine the next action from authoritative database state rather than trusting client-side state.

Expected states:

```text
Arrival not completed
→ Start Arrival

Arrival completed
→ Start Check-in

Check-in completed
→ Start Departure

Departure completed
→ View Timeline

Trip complete / timeline viewed
→ AI Evidence Summary
```

## 4. Page behavior contract

### `/login`

- Driver enters credentials/code.
- Successful login → `/`.
- Unauthenticated user can access login.
- Authenticated-user behavior should be handled explicitly by the implementation.

### `/` — Trip Hub

- Requires authentication.
- Shows trip/driver information and progress.
- Shows completed events.
- Shows exactly one clear next primary action based on database state.
- Does not contain multiple competing event CTAs.
- Refresh recomputes state from the database.

### `/events/arrival`

- Accessible only when Arrival is the next required event.
- Submit Arrival evidence.
- Successful submission → `/`.
- Back before submission → `/`.
- If Arrival is already completed, do not allow duplicate submission; redirect/block and return the user to the correct Hub state.

### `/events/checkin`

- Accessible only after Arrival is completed and before Check-in is completed.
- Submit Check-in evidence.
- Successful submission → `/`.
- Back before submission → `/`.
- Direct/out-of-order access must be blocked.

### `/events/departure`

- Accessible only after Check-in is completed and before Departure is completed.
- Submit Departure evidence.
- Successful submission → `/timeline` as the next evidence-review stage.
- Back before submission → `/`.
- Direct/out-of-order access must be blocked.

### `/timeline`

- Requires authentication.
- Shows chronological Arrival → Check-in → Departure records.
- Shows relevant timestamp, GPS, and photo evidence.
- Refresh re-fetches from authoritative data.
- Returns to `/` when leaving the timeline.

### AI Evidence Summary

- Generated only after all required events exist.
- Must summarize factual stored evidence only.
- Must not invent intent, blame, or unsupported facts.

## 5. Out-of-order access rule

The application must enforce event sequence using authoritative server/database state.

Example:

```text
Arrival not complete
→ /events/checkin is blocked
→ user is returned to Hub
→ Hub shows Start Arrival
```

Likewise:

```text
Check-in not complete
→ /events/departure is blocked
→ user is returned to Hub
→ Hub shows Start Check-in
```

## 6. Browser refresh / duplicate submission principle

Refresh must reconstruct the current state from the database.

The application must not depend on temporary client state to decide which event is next.

Already-completed events must not be submitted again as duplicate immutable event records.

## 7. Back navigation principle

Before an event is submitted, Back/Return should take the driver to the Trip Hub.

After an event is successfully submitted, the application should use the defined success destination rather than relying on browser history to advance the workflow.

Browser Back must not provide a way to bypass event ordering or create duplicate submissions.

## 8. Current implementation state

The source audit found:

- `/login` active
- `/` active but currently has no event navigation
- `/events/arrival` active but currently orphaned from `/`
- `/test-day2` exists as a testing utility
- Check-in and Departure pages do not yet exist
- Timeline and AI summary pages do not yet exist
- Current Arrival implementation does not yet prevent duplicate Arrival submissions

These are implementation facts, not design decisions.

## 9. Execution strategy

Do not use rigid one-calendar-day assumptions as the primary planning unit.

Use **3–4 day execution blocks** and track:

- planned scope
- actual hours
- completed work
- problems discovered
- verification status
- next block

The user's AI-agent workflow has already demonstrated significantly faster execution than the original day-by-day estimates.

## 10. Next implementation priority

Before starting Check-in implementation:

1. Fix `/` into the Trip Hub.
2. Connect `/` → Arrival.
3. Define and enforce the current next-event state.
4. Define safe Back/direct-route/refresh behavior.
5. Prevent duplicate Arrival submission.
6. Verify the navigation manually.
7. Only then implement Check-in.

## 11. Deferred product vision

The broader product vision remains:

```text
Pickup
→ Arrival
→ Check-in
→ Load
→ Depart
→ In Transit
→ Delivery Arrival
→ Check-in
→ Unload/Delivery
→ Depart
→ Delivery Complete
→ Evidence + AI Summary
```

This is **not part of the current implementation scope**. It is a future direction and should not cause a foundation redesign during the current MVP build.

## 12. Final decision

**LOCK:** Keep the current single-facility, fixed 3-event MVP.

**LOCK:** Trip Hub is the navigation/state source of truth.

**LOCK:** Complete and verify the current full loop before considering multi-stop expansion.

**DEFER:** Multi-stop / full pickup-to-delivery journey until the current MVP is complete and only if remaining time makes it sensible.

**NEXT:** Implement and verify Hub/navigation foundation, then proceed to Check-in.
