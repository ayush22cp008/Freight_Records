# Chat44 — Day 18 — Node 7 — Phase 1b — Company Trip Detail Evidence Visibility Investigation Report

## 1. Investigation Status
INVESTIGATION COMPLETE — NO SOURCE CHANGES MADE

## 2. Executive Result
The Company Unified Trip Detail page **already retrieves the evidence/photo data** from the database and is already authorized to display it for both sending and receiving companies. The current absence of evidence display is strictly a **frontend presentation gap**. No backend, API, or database changes are required.

## 3. Source Files Inspected
- `src/app/(authenticated)/company/trips/[id]/page.tsx` (Company Unified Trip Detail)
- `src/app/(authenticated)/driver/active/page.tsx` (Driver Active Trip - Reference)

## 4. Company Trip Detail Evidence Path
### Q1 — Does the Company Trip Detail currently retrieve events?
**YES.**
File: `src/app/(authenticated)/company/trips/[id]/page.tsx`
The page fetches the trip and events using:
```tsx
  const { data: trip } = await supabaseServer
    .from('trips')
    .select(`*, events (*)`)
    .eq('id', id)
    .single();
```

### Q2 — Does it currently retrieve `photo_url`?
**YES.**
By using the wildcard `events (*)`, all event columns, including `photo_url`, are retrieved.

### Q3 — If `photo_url` is retrieved, is it rendered?
**NO.**
The "Event Timeline" section maps over `sortedEvents` but only renders `evt.event_type` and `evt.created_at`. It explicitly does not render `evt.photo_url` or any `<img>` tag.

### Q4 — If it is not retrieved, does another existing Company-accessible query already provide it?
Not applicable, as it is already retrieved by the existing query.

## 5. Existing Evidence Data Availability
### Q7 — Does the existing evidence model use `events.photo_url`?
**YES.**
In the Driver portal reference (`src/app/(authenticated)/driver/active/page.tsx`), evidence is counted using:
```tsx
const { data: events } = await supabaseServer.from('events').select('event_type, photo_url').eq('trip_id', trip.id);
const photosCollected = events?.filter(e => e.photo_url).length || 0;
```
This confirms that `events.photo_url` is the established model for evidence.

## 6. Sender vs Receiver Access Assessment
### Q5 — Can both Company relationships access relevant evidence using existing supported data access?
- **Sending Company:** **YES.** Authorized via `const isSender = trip.company_id === company.id;`
- **Receiving Company:** **YES.** Authorized via `const isReceiver = trip.receiving_company_id === company.id;`
Both relationships share the exact same data retrieval logic, which already fetches `events (*)`. There is no data barrier preventing either from seeing the evidence.

## 7. Driver Reference Comparison
### Q6 — Is there an existing Company timeline/event surface that already contains the evidence information?
**NO.**
The Company Trip Detail has an "Event Timeline" section, but it only contains text descriptions of events and timestamps. It lacks visual evidence rendering.

## 8. Public Share Boundary
The Public Share logic remains untouched by this investigation. Since the Company Unified Trip Detail operates strictly within the authenticated `(authenticated)` route and checks explicit relationships (`isSender` or `isReceiver`), rendering the evidence here will not expose it to Public Share.

## 9. Protected-Boundary Assessment
### Q8 — Would implementing Company Delivery Evidence require any of the following?
- API contract change: **NO**
- API response-shape change: **NO**
- new API endpoint: **NO**
- database schema change: **NO**
- RLS/security policy change: **NO**
- authentication/authorization change: **NO**
- evidence-model change: **NO**
- evidence-storage change: **NO**
- new business rule: **NO**

## 10. Root Cause / Classification
### Q9 — Is the Company evidence gap frontend-only?
**VERIFIED FRONTEND-ONLY.**
The data (including `events.photo_url`) is successfully and safely loaded into frontend memory by the server component. The only missing piece is a frontend `<img>` tag or similar component to render the `photo_url` string.

## 11. VERIFIED / INFERRED / UNKNOWN Summary
- **VERIFIED:** Data retrieval includes `events.photo_url`.
- **VERIFIED:** Authorization logic permits both sending and receiving companies to view the loaded data.
- **VERIFIED:** The gap is purely visual rendering.
- **UNKNOWN:** None.

## 12. Recommended Next Action
**1. Create a frontend-only implementation decision/prompt for Company Trip Detail evidence.**
The fix will involve mapping over `sortedEvents` and rendering an `<img src={evt.photo_url} />` (or equivalent accessible visual) if the string exists.

## 13. Explicit Implementation Authorization Status
**NO SOURCE CHANGES MADE DURING THIS INVESTIGATION.** 
Awaiting next-step authorization from the user.
