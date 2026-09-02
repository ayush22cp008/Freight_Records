# Driver Dashboard Historical Trip AI Summary — Investigation

**Project:** Freight — AI Builders Hackathon  
**Scope:** AI Evidence Summary for historical completed trips opened from Driver Dashboard Timeline  
**Status:** INVESTIGATION COMPLETE — implementation plan required before fix

## 1. Purpose

Investigate why the AI Evidence Summary fails on a historical completed trip even though the selected trip's Timeline now loads correctly.

The observed error is:

```text
No active trip found.
```

The investigation must determine whether the failure is caused only by trip selection/status lookup or whether additional compatibility issues exist between the AI Summary API and the current Node 5 event vocabulary.

This investigation does not reopen Node 4 or the accepted Node 5 lifecycle. Any required AI-summary compatibility fix must remain narrowly scoped to the AI Summary path.

## 2. Manual Observation

Ayush manually opened a completed trip from the Driver Dashboard's `Past / Completed Trips` section.

The Dashboard navigated to a specific Timeline URL:

```text
/timeline?tripId=<specific completed trip id>
```

The Timeline successfully loaded the selected trip and displayed its recorded events, including current Node 5 lifecycle events such as:

```text
ARRIVAL
CHECKIN
GOODS_LOADED
RECEIVER_CHECKED_IN
GOODS_UNLOADED
DELIVERY_DEPARTED
```

At the bottom of the Timeline, the AI Evidence Summary section displayed the `Generate AI Summary` button. Clicking it returned:

```text
Error
No active trip found.
```

Status: **VERIFIED by Ayush manual observation.**

## 3. Current Timeline Selection State

The current Timeline page now accepts `searchParams` and reads `tripId` from the URL. When a string `tripId` is supplied, the trip query applies:

```typescript
.eq('id', tripId)
```

while also applying:

```typescript
.eq('driver_id', driver.id)
```

and the existing allowed status set:

```text
active
claimed
in_progress
completed
```

When no `tripId` is supplied, the Timeline falls back to the most recent matching trip using ordering and a limit rather than relying on `.single()` across multiple historical rows.

Status: **VERIFIED from current source inspection.**

## 4. AI Summary Client Evidence

Current source file:

```text
src/components/AIEvidenceSummary.tsx
```

The component's `generateSummary()` function sends:

```typescript
fetch('/api/summary', { method: 'POST' })
```

No `tripId` is sent in the request body, URL, or another explicit request field.

Therefore the AI Summary component does not identify which Timeline trip the user is currently viewing.

Status: **VERIFIED from current source inspection.**

## 5. AI Summary API Trip Lookup Evidence

Current source file:

```text
src/app/api/summary/route.ts
```

The API authenticates the current user and resolves the corresponding driver using:

```typescript
.from('drivers')
.select('id')
.eq('auth_id', user.id)
.single()
```

It then independently looks up a trip using the authenticated driver's ID and status:

```typescript
.from('trips')
.select('id')
.eq('driver_id', driver.id)
.in('status', ['active', 'claimed', 'in_progress', 'completed'])
.single()
```

The API does not consume the `tripId` selected by the Timeline.

Because a driver can now have multiple completed trips, this driver-only lookup is not a reliable selector for a historical trip. The use of `.single()` is also incompatible with an unrestricted set containing multiple matching historical trips.

Status: **VERIFIED from current source inspection.**

## 6. Immediate Root Cause

**ROOT CAUSE A — Selected trip identity is lost before AI Summary generation.**

The Timeline knows the exact historical trip from `tripId`, but `AIEvidenceSummary` does not receive or transmit that ID. `/api/summary` consequently performs a separate driver/status lookup instead of summarizing the exact trip displayed on the Timeline.

This is the direct explanation for why the historical-trip AI Summary path can return `No active trip found.` despite the Timeline itself successfully displaying the trip.

## 7. Security Requirement for the Fix

The requested `tripId` must never be treated as sufficient authorization.

The correct security boundary is:

```text
authenticated user
      ↓
server-resolved driver.id
      ↓
requested tripId
      ↓
trip.id = requested tripId
AND
trip.driver_id = authenticated driver.id
```

The API must continue resolving the driver from the authenticated session. A client-supplied `tripId` must only select a trip already owned by that authenticated driver.

A different driver's trip ID must not permit access to that trip's events or AI-generated summary.

## 8. Event Vocabulary Investigation

The current Node 5 database migration explicitly allows both legacy and canonical event values. The canonical Node 5 values include:

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

Legacy values remain allowed:

```text
arrival
checkin
departure
```

This is confirmed by:

```text
src/db/migrations/008_node5_delivery_evidence_schema.sql
```

Status: **VERIFIED from source migration.**

## 9. AI Summary Event Validation Evidence

The current `/api/summary` route still contains a validation check requiring the event list to include:

```text
arrival
checkin
departure
```

before AI generation proceeds.

This is important because the manually displayed historical Timeline contains canonical Node 5 event names such as `GOODS_LOADED`, `RECEIVER_CHECKED_IN`, `GOODS_UNLOADED`, and `DELIVERY_DEPARTED`.

Therefore, fixing only the historical trip selection is **not sufficient to declare historical AI Summary support complete**. After exact-trip selection is corrected, the event validation must be evaluated against the current accepted Node 5 lifecycle and the actual event sequence used by completed trips.

The earlier Chat25 AI Summary bug-fix instruction explicitly treated the legacy event check as out of scope for the earlier status-only fix. That constraint applied to that earlier bug fix; this investigation identifies the same check as a separate compatibility issue that must now be resolved deliberately if it blocks the current accepted Node 5 historical-summary path.

Status: **VERIFIED that the legacy check exists; compatibility impact is VERIFIED as a technical concern but requires implementation-level validation against the exact completed event sequence.**

## 10. Existing AI Summary Contract

The AI Summary API already follows an evidence-grounded model:

1. authenticate the user;
2. resolve the driver;
3. load trip events;
4. construct a deterministic evidence payload containing event type, server timestamp, coordinates, GPS accuracy, and whether photo evidence was provided;
5. instruct the AI to summarize only the supplied structured evidence.

The investigation does not identify a need to redesign this evidence-grounded prompt/model behavior.

The required change is primarily to ensure that the correct selected trip's deterministic evidence is supplied to the existing summary process.

## 11. Root Cause Classification

| Finding | Classification |
|---|---|
| Timeline loads selected historical trip | VERIFIED |
| AI Summary button is present | VERIFIED |
| AI Summary request contains no tripId | VERIFIED |
| API selects trip by authenticated driver + status | VERIFIED |
| API uses `.single()` without exact historical trip ID | VERIFIED |
| Multiple completed trips make this selection unsafe/ambiguous | VERIFIED |
| Server-side driver identity is already available | VERIFIED |
| Exact trip ownership must remain server-enforced | VERIFIED requirement |
| Legacy `arrival/checkin/departure` validation remains | VERIFIED |
| Canonical Node 5 event vocabulary exists | VERIFIED |
| Legacy validation may reject canonical-only completed trips | VERIFIED technical risk; exact runtime result requires validation |
| AI prompt/model redesign required | NOT INDICATED |
| Dashboard history query is defective | NOT INDICATED |
| Timeline selection is defective after current fix | NOT INDICATED by current screenshots/source |

## 12. Scope Decision

This should be treated as a **focused AI Evidence Summary historical-trip compatibility fix**.

The implementation should address both required layers:

### Layer A — Exact trip selection

Pass the selected Timeline `tripId` into the AI Summary request and make `/api/summary` select that exact trip while enforcing authenticated driver ownership.

### Layer B — Current Node 5 evidence compatibility

Verify the AI Summary's event-sequence gate against the current accepted Node 5 canonical lifecycle. If the gate blocks valid canonical completed trips, update only the necessary validation/mapping logic so the existing evidence-grounded summary can operate on the canonical lifecycle without changing the Node 5 event model itself.

Do not assume Layer B is solved merely because Layer A is fixed.

## 13. Non-Goals

Do not:

- modify the Dashboard completed-trip query;
- modify Node 4 claiming or authorization;
- modify Node 5 trip status semantics;
- modify database schema or migrations;
- change canonical event vocabulary;
- change event insertion behavior;
- weaken RLS or authorization;
- trust a client-provided driver ID;
- redesign the AI prompt unnecessarily;
- replace the deterministic evidence payload with client-provided evidence;
- create a new summary system;
- redesign the Timeline UI;
- introduce unrelated dashboard features.

## 14. Required Implementation Verification

### Historical completed trip

```text
[ ] Open a completed trip from Past / Completed Trips
[ ] Timeline loads the selected trip
[ ] Generate AI Summary targets that exact trip
[ ] AI Summary successfully generates from that trip's deterministic events
```

### Multiple historical trips

```text
[ ] Generate summary for completed trip A
[ ] Generate summary for completed trip B
[ ] Each summary uses only its selected trip's events
[ ] No trip substitution occurs
```

### Authorization

```text
[ ] Authenticated driver identity is resolved server-side
[ ] Requested trip must belong to that driver
[ ] Another driver's tripId cannot return their summary
```

### Event vocabulary

```text
[ ] Current canonical Node 5 events are accepted where required
[ ] Existing legacy events remain supported where required
[ ] No event names are changed by this fix
```

### Regression

```text
[ ] Existing active/claimed/in-progress summary flow remains functional
[ ] Timeline remains functional
[ ] Dashboard remains unchanged
[ ] Node 4 claim flow remains unchanged
[ ] Node 5 lifecycle remains unchanged
[ ] npx tsc --noEmit passes
```

### Manual evidence

```text
[ ] Ayush manually verifies historical AI summary
[ ] Ayush verifies cross-driver tripId protection
[ ] Implementation report records actual manual result
```

## 15. Investigation Conclusion

**Observation:** Historical Timeline works, but Generate AI Summary returns `No active trip found.`  
**Root Cause A:** The selected Timeline `tripId` is not propagated to the AI Summary API, which independently searches by driver/status.  
**Root Cause B / Compatibility Risk:** The AI Summary API still requires legacy `arrival/checkin/departure` event names, while current Node 5 completed trips use canonical lifecycle event names.  
**Security requirement:** Exact trip selection must remain constrained by the server-resolved authenticated driver ID.  
**Decision:** Create a focused implementation plan covering exact trip propagation/ownership and current Node 5 event-validation compatibility.  
**Status:** READY FOR IMPLEMENTATION PLAN.
