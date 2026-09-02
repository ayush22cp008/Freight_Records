# Driver Dashboard Historical Trip AI Summary — Mixed Event Vocabulary Implementation Plan

## 1. Goal
Fix the AI Evidence Summary validation so it successfully accepts historical trips containing a mix of legacy and canonical Node 5 event names, while simultaneously upgrading the event writers so future trips use a strict canonical vocabulary.

## 2. Deep Architectural Answers
1. **Should Arrival/Check-in be migrated?** Yes. The underlying event writers (`/api/events/arrival` and `/api/events/checkin`) should be migrated to insert `ARRIVED_AT_PICKUP` and `PICKUP_CHECKED_IN` to keep future data clean.
2. **Should legacy departure remain supported?** `departure` was replaced by `PICKUP_DEPARTED` in the Node 5 flow, but historical legacy trips using `departure` still exist.
3. **Are existing trips expected to contain mixed events?** Yes. Because the Node 5 flow introduced new canonical events (`GOODS_LOADED`, `PICKUP_DEPARTED`) without immediately rewriting the old arrival/checkin routes, trips completed during this window have a mixed vocabulary (e.g. `arrival`, `checkin`, `PICKUP_DEPARTED`).
4. **Final Decision:** **Solution A + B**. Canonicalize event writers going forward, AND normalize legacy+canonical events at the AI-summary boundary to support already-completed trips.

## 3. Approach

### A. Normalize Validation at the AI Summary Boundary
**`src/app/api/summary/route.ts`**
Instead of strictly enforcing all-legacy or all-canonical, validate that the trip has *at least one valid representation* of each required milestone:
```typescript
const hasArrival = eventTypes.includes('arrival') || eventTypes.includes('ARRIVED_AT_PICKUP');
const hasCheckin = eventTypes.includes('checkin') || eventTypes.includes('PICKUP_CHECKED_IN');
const hasDeparture = eventTypes.includes('departure') || eventTypes.includes('PICKUP_DEPARTED') || eventTypes.includes('DELIVERY_DEPARTED');

if (!hasArrival || !hasCheckin || !hasDeparture) {
  return NextResponse.json({ error: 'Evidence summary requires the completed event sequence.' }, { status: 400 });
}
```
*(No prompt redesign is needed, as the LLM inherently understands both `arrival` and `ARRIVED_AT_PICKUP` contextually.)*

### B. Canonicalize the Event Writers
1. **`src/app/api/events/arrival/route.ts`**
   Change the inserted `event_type` from `'arrival'` to `'ARRIVED_AT_PICKUP'`.
2. **`src/app/api/events/checkin/route.ts`**
   Change the inserted `event_type` from `'checkin'` to `'PICKUP_CHECKED_IN'`.

## 4. Security & Boundaries
- Resolving the driver identity securely remains unchanged.
- Relying entirely on deterministically recorded `event_type`s remains unchanged.

## 5. Verification Plan
- **Automated:** Run `npx tsc --noEmit`.
- **Manual Verification (by Ayush):**
  - Verify that the specific affected historical trip (`4376665c-e7c6-41b2-98bd-373600b66b48`) now successfully summarizes.
  - Test a new active trip (Arrival → Check-in), observing that the newly inserted events are properly named `ARRIVED_AT_PICKUP` and `PICKUP_CHECKED_IN` in the timeline.
