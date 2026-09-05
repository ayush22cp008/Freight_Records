# Chat39 — Day 15 — Company Portal Final Remaining System-Presence Investigation Report

## Investigation Scope

This is the final focused investigation establishing the exact system-presence of Public Share and a complete inventory of the Company Portal flows.

---

### Investigation 1 — Exact Public Share System Presence and Exposed Data

1. **Does Public Share actually exist in the current system?**
   - **YES (VERIFIED)**.
   - Frontend Public Route: `src/app/share/[token]/page.tsx`
   - Data Lookup Logic: `src/lib/public-share-lookup.ts`
   - Management UI: `src/app/(authenticated)/company/PublicShareManager.tsx`
   - Management API: `src/app/api/trips/[tripId]/public-share/route.ts`

2. **How is a Public Share created, activated, revoked, and/or viewed?**
   - **Created/Revoked**: The Receiving Company uses the `PublicShareManager` component on their Dashboard to `POST` or `DELETE` to the management API. The API generates a token, hashes it, and stores the hash in `trip_public_shares` with status `ACTIVE` or `REVOKED`.
   - **Viewed**: A viewer visits `/share/[token]`. The server-side page reads the token, hashes it, looks up the active share, and returns a strict data projection.

3. **What exact data is returned to the public viewer?**
   The strictly enforced projection returns:
   - **Company**: Receiving Company's Name.
   - **Trip**: Status, Delivery Date (derived from `DELIVERY_DEPARTED` timestamp), Pickup City (`facility_name`), Destination City (`destination_name`).
   - **Evidence**: State (`COMPLETE`/`INCOMPLETE`) and a boolean checklist (Arrival, Checkin, Departure).
   - **AI Summary**: A text summary generated from events.
   - **Timeline**: An array of key events (`ARRIVED_AT_DELIVERY`, `RECEIVER_CHECKED_IN`, `DELIVERY_DEPARTED`) including event type and timestamp.

4. **Intentionally Excluded Fields (VERIFIED)**
   The `getPublicVerificationData` function strictly drops or ignores the following data, meaning it is definitively not exposed to the public:
   - **Exact GPS Coordinates**: Events have `latitude` and `longitude`, but the public timeline hardcodes `location: 'Location recorded'` instead of exposing them.
   - **Photos**: Event `photo_url` is not queried or exposed.
   - **Sending Company**: The creator company is not queried or exposed.
   - **Driver Information**: Driver name and ID are completely omitted.
   - **Financials / Logistics**: `distance`, `duration`, and `payout` are completely omitted.

5. **Public View Mechanism**
   - **Unauthenticated/Public**. Anyone with the cryptographically secure token link (`/share/[token]`) can view it.

6. **Frontend vs API Match**
   - **Matches Perfectly**. The server component explicitly requests the projection and renders exactly those fields.

7. **Sender vs Receiver Behavior**
   - **Verified**: Only the Receiving Company can create the link. The public view itself is identical regardless of who views it.

---

### Investigation 2 — Complete Company Frontend Flow Inventory

The following is the evidence-based inventory of current Company Portal flows:

1. **Company Dashboard**: **PRESENT** (`src/app/(authenticated)/page.tsx`). Shows incoming and completed deliveries (filtered strictly by `receiving_company_id`).
2. **Create Trip**: **PRESENT** (`src/app/(authenticated)/company/trips/create/CreateTripClient.tsx`). Allows creating a draft trip.
3. **Publish Trip**: **PRESENT** (`/api/trips/publish/route.ts`). Allows the sending company to publish their draft.
4. **Incoming Deliveries**: **PRESENT** (Inline on Dashboard).
5. **Receiver Check-in**: **PRESENT** (`/company/receiver-checkin/ReceiverCheckinClient.tsx`).
6. **Receiver Completion**: **PRESENT** (`/company/completion/ReceiverCompletionClient.tsx`). Has a verified frontend bug preventing the success state from rendering.
7. **Completed Deliveries**: **PRESENT** (Inline on Dashboard).
8. **Public Share Management**: **PRESENT** (`PublicShareManager.tsx`).
9. **Public Share Viewing**: **PRESENT** (`/share/[token]/page.tsx`).
10. **Company Profile / Account Settings**: **NOT PRESENT IN CURRENT SYSTEM**. There is absolutely no UI, route, or component for a Company to edit their profile, view account details, or configure settings. The only profile element is the global Navbar showing their email.
11. **Company Timeline / History**: **NOT PRESENT IN CURRENT SYSTEM**. The `/timeline` route explicitly blocks Companies by forcing a Driver identity check (redirecting them back to `/`). The Company has no dedicated history view beyond the basic 10-item completed list on the dashboard.
