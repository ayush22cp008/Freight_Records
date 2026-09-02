# Driver Dashboard Historical Trip AI Summary — Mixed Event Vocabulary Investigation

**Date:** 2026-09-02  
**Status:** INVESTIGATION IN PROGRESS  
**Scope:** Historical completed-trip Timeline → AI Evidence Summary  
**Affected trip:** `4376665c-e7c6-41b2-98bd-373600b66b48`

---

## 1. Observation

The Driver Dashboard historical-trip flow successfully opens a selected completed trip using:

```text
/timeline?tripId=4376665c-e7c6-41b2-98bd-373600b66b48
```

The Timeline displays a complete nine-step Node 5 lifecycle, but the AI Evidence Summary returns:

```text
Evidence summary requires the completed event sequence (Arrival, Check-in, Departure).
```

This error was reproduced manually from the deployed application.

---

## 2. Manual Evidence — Exact Historical Trip

The screenshots supplied during investigation show the following event sequence for the selected historical trip:

```text
Step 1  ARRIVAL
Step 2  CHECKIN
Step 3  GOODS_LOADED
Step 4  PICKUP_DEPARTED
Step 5  IN_TRANSIT
Step 6  ARRIVED_AT_DELIVERY
Step 7  RECEIVER_CHECKED_IN
Step 8  GOODS_UNLOADED
Step 9  DELIVERY_DEPARTED
```

Therefore the trip has a complete lifecycle from pickup arrival through delivery departure.

**Important:** Steps 1 and 2 are displayed using legacy event names (`ARRIVAL` / `CHECKIN`), while later Node 5 milestones use canonical names such as `GOODS_LOADED` and `PICKUP_DEPARTED`.

---

## 3. Source Evidence

The current AI summary API at commit `a10dbc9694a6c141d31400a6bc2628a1cc4249fe` validates either of two complete pickup sequences:

```text
Legacy:
arrival + checkin + departure

OR

Canonical:
ARRIVED_AT_PICKUP + PICKUP_CHECKED_IN + PICKUP_DEPARTED
```

The API returns the observed error when neither complete sequence is present.

The current event writers confirm the mixed vocabulary:

### Arrival

`src/app/(authenticated)/events/arrival/page.tsx` and its client submit to `/api/events/arrival`. The API inserts:

```text
event_type: 'arrival'
```

### Check-in

`src/app/api/events/checkin/route.ts` inserts:

```text
event_type: 'checkin'
```

### Pickup departure

`src/app/api/events/pickup-departed/route.ts` inserts:

```text
event_type: 'PICKUP_DEPARTED'
```

### Database schema

Node 5 migration `008_node5_delivery_evidence_schema.sql` explicitly permits both the legacy values:

```text
arrival
checkin
departure
```

and canonical Node 5 values including:

```text
ARRIVED_AT_PICKUP
PICKUP_CHECKED_IN
GOODS_LOADED
PICKUP_DEPARTED
IN_TRANSIT
ARRIVED_AT_DELIVERY
RECEIVER_CHECKED_IN
GOODS_UNLOADED
DELIVERY_DEPARTED
```

---

## 4. Immediate Root Cause

For the affected historical trip, the AI summary validation receives a mixed event vocabulary:

```text
arrival
checkin
GOODS_LOADED
PICKUP_DEPARTED
IN_TRANSIT
ARRIVED_AT_DELIVERY
RECEIVER_CHECKED_IN
GOODS_UNLOADED
DELIVERY_DEPARTED
```

Consequently:

```text
hasLegacy:
arrival + checkin + departure
             ↑
             missing legacy departure

hasCanonical:
ARRIVED_AT_PICKUP + PICKUP_CHECKED_IN + PICKUP_DEPARTED
↑                  ↑
missing            missing
```

Both validation branches are false, so the API returns the observed 400 error.

**Root cause status: CONFIRMED for the reproduced failure.**

---

## 5. Deeper Architectural Question

The immediate failure is understood, but the correct fix has **not** yet been decided.

We must determine the intended canonical event vocabulary for the entire Node 5 pickup lifecycle.

Specifically:

1. Should Arrival be migrated from `arrival` to `ARRIVED_AT_PICKUP`?
2. Should Check-in be migrated from `checkin` to `PICKUP_CHECKED_IN`?
3. Should legacy `departure` remain supported anywhere, or has `PICKUP_DEPARTED` replaced it?
4. Are existing completed trips expected to contain mixed legacy/canonical events because they were created during an earlier implementation stage?
5. Should the AI Summary normalize both historical vocabularies, or should the underlying event writers be made canonical first?
6. What behavior is required for already-completed historical trips that contain legacy/mixed event names?

These questions must be answered before another implementation is authorized.

---

## 6. Security Constraint

Any historical AI Summary fix must preserve the existing ownership boundary:

```text
Authenticated session
        ↓
server-resolved driver.id
        ↓
selected tripId
        ↓
trip query constrained by driver_id
```

The client-provided `tripId` must never become an unrestricted trip selector.

---

## 7. Non-Goals During This Investigation

Do not yet:

- modify the AI prompt/model;
- weaken authorization;
- remove authenticated-driver ownership checks;
- reopen Node 4 claiming/assignment logic;
- redesign the Driver Dashboard;
- change the Node 5 lifecycle ordering;
- add unrelated event types;
- blindly accept any three vaguely named events;
- create a second source of truth for trip ownership.

---

## 8. Required Next Investigation

Before writing another implementation plan, inspect the complete Node 5 event-writer architecture and determine the intended canonical source for:

```text
ARRIVED_AT_PICKUP
PICKUP_CHECKED_IN
PICKUP_DEPARTED
```

Then classify existing legacy event writers and historical records as one of:

```text
CANONICAL
LEGACY-BUT-SUPPORTED
MIGRATION-COMPATIBILITY
BUG / INCONSISTENCY
```

The final decision must explicitly state whether the correct solution is:

```text
A. Canonicalize event writers,
B. Normalize legacy + canonical events at the AI-summary boundary,
C. Support both with a documented compatibility layer,
D. Another evidence-backed approach.
```

No implementation should begin until this decision is evidence-backed.

---

## 9. Investigation Status

| Item | Status |
|---|---|
| Historical trip selection | VERIFIED |
| Complete nine-step lifecycle visible | VERIFIED |
| Mixed event vocabulary | VERIFIED |
| AI Summary validation failure | VERIFIED |
| Immediate root cause | CONFIRMED |
| Intended canonical event vocabulary | UNKNOWN |
| Correct long-term fix | UNKNOWN |
| New implementation authorized | NO |

---

## 10. Evidence References

- Historical Timeline screenshots supplied during manual verification.
- `src/app/api/summary/route.ts` at `a10dbc9694a6c141d31400a6bc2628a1cc4249fe`.
- `src/app/api/events/arrival/route.ts` at `a10dbc9694a6c141d31400a6bc2628a1cc4249fe`.
- `src/app/api/events/checkin/route.ts` at `a10dbc9694a6c141d31400a6bc2628a1cc4249fe`.
- `src/app/api/events/pickup-departed/route.ts` at `a10dbc9694a6c141d31400a6bc2628a1cc4249fe`.
- `src/db/migrations/008_node5_delivery_evidence_schema.sql`.
- `05_DEBUGGING/investigations/Driver_Dashboard_Historical_Trip_AI_Summary_Investigation.md`.
- `05_DEBUGGING/investigations/Driver_Dashboard_Historical_Trip_AI_Summary_Implementation_Plan.md`.
- `03_IMPLEMENTATION/implementation_reports/Driver_Dashboard_Historical_Trip_AI_Summary_Implementation_Report.md`.
