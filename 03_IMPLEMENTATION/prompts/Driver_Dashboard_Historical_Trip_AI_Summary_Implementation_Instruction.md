# Antigravity Implementation Instruction — Historical Trip AI Evidence Summary

**Project:** Freight — AI Builders Hackathon  
**Source repository:** `ayush22cp008/freight_hackathon`  
**Records repository:** `ayush22cp008/Freight_Records`  
**Scope:** Historical completed-trip AI Evidence Summary only  
**Status:** READY FOR ANTIGRAVITY EXECUTION

## 1. Authority

Read and follow the approved Records documents first:

```text
05_DEBUGGING/investigations/Driver_Dashboard_Historical_Trip_AI_Summary_Investigation.md
05_DEBUGGING/investigations/Driver_Dashboard_Historical_Trip_AI_Summary_Implementation_Plan.md
```

Do not expand the scope beyond this instruction.

## 2. Objective

Make the existing AI Evidence Summary work for the exact historical completed trip currently displayed by Timeline.

Required flow:

```text
Past / Completed Trips
        ↓
View Timeline
        ↓
/timeline?tripId=<specific-trip-id>
        ↓
Timeline loads exact trip
        ↓
Generate AI Summary
        ↓
AI summarizes that exact trip's deterministic evidence
```

The existing active/claimed/in-progress AI Summary behavior must remain functional.

## 3. Required Files

Only these source files are expected to require modification unless a concrete source-level dependency makes another file necessary:

```text
src/app/(authenticated)/timeline/page.tsx
src/components/AIEvidenceSummary.tsx
src/app/api/summary/route.ts
```

Do not modify the Dashboard implementation for this fix.

## 4. Step A — Propagate Exact tripId from Timeline

Current Timeline already resolves the exact trip and renders the AI Summary component.

Change the component usage so that the selected trip ID is passed to the component:

```tsx
<AIEvidenceSummary tripId={trip.id} />
```

Do not change Timeline trip selection or event-loading behavior except what is strictly required to pass the already-resolved ID into the summary component.

## 5. Step B — Update AIEvidenceSummary Request

Update:

```text
src/components/AIEvidenceSummary.tsx
```

to accept an optional `tripId` prop.

When generating a summary, include the selected `tripId` in the POST body sent to `/api/summary`.

Expected request shape:

```typescript
await fetch('/api/summary', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ tripId }),
});
```

Preserve the existing loading, error, summary, and retry UI behavior.

Do not move deterministic evidence construction into the client.

## 6. Step C — Update AI Summary API Trip Selection

Update:

```text
src/app/api/summary/route.ts
```

The API must continue to authenticate the request and resolve the current driver's server-side identity exactly as it does today.

Parse the request body for the optional `tripId`.

When `tripId` is supplied, the trip query must select that exact trip while preserving driver ownership:

```text
trip.id = requested tripId
AND
trip.driver_id = authenticated driver.id
```

The existing supported trip statuses must remain compatible with:

```text
active
claimed
in_progress
completed
```

When no `tripId` is supplied, preserve a safe fallback by selecting the most recent matching trip rather than using `.single()` over an unrestricted historical result set.

The API must never select another driver's trip because of a client-supplied ID.

## 7. Step D — Node 5 Event Vocabulary Compatibility

The current Node 5 schema supports both legacy and canonical event names.

Canonical lifecycle values include:

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

Legacy values include:

```text
arrival
checkin
departure
```

The current AI Summary route contains a legacy validation gate requiring:

```text
arrival + checkin + departure
```

This gate must be updated only as much as necessary so valid completed Node 5 trips using the canonical lifecycle are accepted as valid evidence for AI Summary generation.

Use the approved implementation plan's compatibility approach:

```typescript
const hasLegacy =
  eventTypes.includes('arrival') &&
  eventTypes.includes('checkin') &&
  eventTypes.includes('departure');

const hasCanonical =
  eventTypes.includes('ARRIVED_AT_PICKUP') &&
  eventTypes.includes('PICKUP_CHECKED_IN') &&
  eventTypes.includes('PICKUP_DEPARTED');

if (!hasLegacy && !hasCanonical) {
  return NextResponse.json(
    { error: 'Evidence summary requires the completed event sequence (Arrival, Check-in, Departure).' },
    { status: 400 }
  );
}
```

Do not change the Node 5 event vocabulary, database schema, event insertion APIs, or lifecycle state machine.

Do not weaken the evidence-grounded AI prompt.

## 8. Evidence Integrity

The AI Summary must continue to be generated exclusively from server-loaded deterministic events for the authorized trip.

Do not allow the client to submit arbitrary event data as the summary source.

The existing evidence payload structure and factual summarization prompt should remain unchanged unless a minimal compatibility correction is strictly necessary.

The AI must summarize only the selected trip's server-side events.

## 9. Security Requirements

Mandatory:

- authenticate the current user server-side;
- resolve `driver.id` server-side from the authenticated user;
- constrain the requested trip by both `trip.id` and authenticated `driver.id`;
- do not trust a client-provided driver ID;
- do not expose another driver's events or AI summary;
- preserve existing RLS and authorization boundaries.

A malformed, missing, nonexistent, or unowned `tripId` must not result in another trip being substituted.

## 10. Non-Goals

Do not:

- modify the Dashboard completed-trip history query;
- change Node 4 claiming logic;
- change Node 5 trip status semantics;
- change Node 5 event names;
- modify migrations;
- modify database schema;
- modify RLS/policies;
- redesign the Timeline UI;
- redesign the AI Summary UI;
- introduce a second summary endpoint;
- add client-side evidence generation;
- make unrelated refactors.

## 11. Validation

Run:

```text
npx tsc --noEmit
```

If the project has an existing focused build/lint/test command that is practical for these files, run it and record the result.

Verify source behavior:

```text
[ ] Timeline passes the selected tripId
[ ] AIEvidenceSummary sends tripId
[ ] /api/summary parses tripId
[ ] Exact trip is selected by tripId + authenticated driver.id
[ ] No-tripId fallback remains safe
[ ] Canonical Node 5 event sequence is accepted
[ ] Legacy event sequence remains accepted
[ ] Evidence payload still comes from server-loaded events
```

## 12. Required Ayush Manual Verification

Do not claim these as completed unless Ayush actually performs them.

### Historical completed trip

```text
[ ] Open Past / Completed Trips
[ ] Select completed trip A
[ ] Timeline loads trip A
[ ] Click Generate AI Summary
[ ] Summary is generated for trip A
```

### Multiple completed trips

```text
[ ] Select completed trip A and generate summary
[ ] Select completed trip B and generate summary
[ ] Each summary corresponds only to its selected trip's events
```

### Existing workflow

```text
[ ] Active/claimed/in-progress summary remains functional
```

### Authorization

```text
[ ] Replace tripId in URL/request with another driver's trip ID
[ ] Confirm no unauthorized summary is returned
[ ] Confirm another driver's events are not exposed
```

## 13. Source-Control Discipline

Do not push source changes.

Record:

```text
Source commit before:
<exact SHA>

Source commit after:
<exact SHA or NOT COMMITTED>
```

Only Ayush can authorize a source push.

## 14. Required Records Report

Create exactly one implementation report:

```text
03_IMPLEMENTATION/implementation_reports/Driver_Dashboard_Historical_Trip_AI_Summary_Implementation_Report.md
```

The report must include:

1. Source baseline
2. Files changed
3. Exact historical-trip propagation fix
4. Exact API authorization/ownership fix
5. Event-vocabulary compatibility fix
6. Validation commands and results
7. Manual Ayush verification status
8. Scope/non-changes
9. Commit status
10. VERIFIED / INFERRED / UNKNOWN summary
11. Remaining action

## 15. Stop Conditions

Stop and report instead of guessing if:

- the current source differs materially from this plan;
- a required database/schema change appears necessary;
- existing authorization behavior would have to be weakened;
- the canonical Node 5 event sequence differs from the approved Records evidence;
- the implementation requires changes outside this focused scope.

## 16. Final Response Format

Return only:

```text
HISTORICAL AI SUMMARY FIX COMPLETE

Report:
03_IMPLEMENTATION/implementation_reports/Driver_Dashboard_Historical_Trip_AI_Summary_Implementation_Report.md

Source commit before:
<exact SHA>

Source commit after:
<exact SHA or NOT COMMITTED>

Validation:
PASS / FAIL / PARTIAL

Manual Ayush verification:
NOT PERFORMED

Push:
NOT PERFORMED
```

Do not paste source code or the complete Records report into chat.
