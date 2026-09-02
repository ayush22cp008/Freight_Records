# Driver Dashboard Historical Trip AI Summary Implementation Plan

## 1. Goal
Fix the AI Evidence Summary feature so that it successfully generates summaries for historical completed trips without losing the exact trip identity and without failing due to canonical Node 5 event vocabulary validation.

## 2. Approach

### A. Pass `tripId` to the AI Summary Component
1. **`src/app/(authenticated)/timeline/page.tsx`**
   - Update `<AIEvidenceSummary />` to pass the current trip's ID: `<AIEvidenceSummary tripId={trip.id} />`.

2. **`src/components/AIEvidenceSummary.tsx`**
   - Update the component signature to accept a `tripId?: string` prop.
   - When generating the summary, send `tripId` in the POST request body to the API.

### B. Update AI Summary API to Support Historical Selection & Node 5 Events
3. **`src/app/api/summary/route.ts`**
   - **Trip Selection:** Parse the request body for `tripId`. Update the Supabase trip lookup to enforce `.eq('id', tripId)` if provided, while keeping the `.eq('driver_id', driver.id)` ownership boundary intact. If no `tripId` is provided, fallback to `.order('created_at', { ascending: false }).limit(1)`.
   - **Event Vocabulary Compatibility:** Update the validation logic to accept EITHER the legacy event sequence (`arrival`, `checkin`, `departure`) OR the canonical Node 5 sequence (e.g., `ARRIVED_AT_PICKUP`, `PICKUP_CHECKED_IN`, `PICKUP_DEPARTED`).
     ```typescript
     const hasLegacy = eventTypes.includes('arrival') && eventTypes.includes('checkin') && eventTypes.includes('departure');
     const hasCanonical = eventTypes.includes('ARRIVED_AT_PICKUP') && eventTypes.includes('PICKUP_CHECKED_IN') && eventTypes.includes('PICKUP_DEPARTED');
     
     if (!hasLegacy && !hasCanonical) {
       return NextResponse.json({ error: 'Evidence summary requires the completed event sequence (Arrival, Check-in, Departure).' }, { status: 400 });
     }
     ```

## 3. Security & Boundaries
- Historical trips continue to be strictly protected by server-side resolution of the authenticated driver.
- The deterministic evidence-to-AI prompt behavior remains entirely unchanged, we merely ensure it operates on the correct exact trip.

## 4. Verification Plan
- **Automated:** Run `npx tsc --noEmit`
- **Manual Verification (by Ayush):**
  - Verify that opening a historical trip and clicking "Generate AI Summary" successfully yields a factual AI summary for that specific trip.
  - Verify the existing active trip summary flow still works.
  - Check that a driver cannot spoof the `tripId` for an unowned trip.
