# Chat39 — Day 15 — Company Portal Frontend API / Data Dependencies Investigation Report (Updated)

## Investigation Findings

This report traces the existing Company frontend to its actual API and database dependencies to establish exactly what data the Company portal reads and writes, how the relationship model works, and how authorization is handled.

### Critical Relationship Model Question

> *Are “Sending Company” and “Receiving Company” trip-specific relationships between existing Company entities, rather than permanent Company types/categories?*

**YES. (VERIFIED)**

- **Schema Evidence**: 
  - The `companies` table (`005_v2_onboarding_evidence.sql`) contains only `id`, `auth_id`, `name`, and `created_at`. There is absolutely no permanent "sender" or "receiver" classification/type stored on the Company entity itself.
  - The `trips` table (`006_node3_trip_schema.sql`) explicitly defines two distinct foreign keys pointing to the same table:
    - `company_id uuid REFERENCES companies(id)` (This acts as the Sender/Creator)
    - `receiving_company_id uuid REFERENCES companies(id)` (This acts as the Receiver)
- **Conclusion**: A Company is just a generic Company. "Sender" and "Receiver" are strictly dynamic, trip-specific relationships. A single Company entity can be the sender (`company_id`) on Trip A and the receiver (`receiving_company_id`) on Trip B simultaneously.

### Critical Public Share Question

> *Within that trip-specific relationship model, explicitly verify whether only the Receiving Company for a particular completed trip can create/manage its Public Share link, while the Sending Company for that same trip cannot.*

**YES. (VERIFIED)**

- **Data Path Evidence**: In `src/app/api/trips/[tripId]/public-share/route.ts`, when a company attempts to POST (create) or DELETE (revoke) a public share, the API explicitly checks: `trip.receiving_company_id !== company.id`. If this evaluates to true, it returns `404 Trip not found or unauthorized`.
- **Conclusion**: The **Sending Company (`company_id`) has absolutely no authorization** to create or revoke Public Shares for a trip they sent. Only the **Receiving Company (`receiving_company_id`)** for that specific trip is authorized to manage its Public Share link.

### 1. Company Dashboard Data Dependencies

**Trace Findings (VERIFIED)**
- **Incoming Deliveries**: 
  - *Source*: Fetched directly in `src/app/(authenticated)/page.tsx` via `supabaseServer.from('trips')`.
  - *Filter*: `.eq('receiving_company_id', company.id).in('status', ['active', 'claimed', 'in_progress'])`.
  - *Joined Data*: Fetches related events via `events ( event_type )` to calculate frontend status text (e.g., "Arrived - Action Required").
- **Completed Deliveries**: 
  - *Source*: Fetched in `page.tsx` via `supabaseServer.from('trips')`.
  - *Filter*: `.eq('receiving_company_id', company.id).eq('status', 'completed')`.
  - *Joined Data*: Fetches related shares via `trip_public_shares ( status )` to determine if a share is currently active.
- **Massive Discovery / Missing Functionality**: Because both the Incoming and Completed queries hardcode `.eq('receiving_company_id', company.id)`, **the Sending Company completely loses all visibility of a trip the moment it is published**. The dashboard has no queries fetching trips by `company_id` (the sender). Therefore, the trip-specific relationship dictates that only receivers see the trip on their dashboard.

### 2. Create Trip / Publish Trip Dependencies

**Trace Findings (VERIFIED)**
- **Draft Creation (`/api/trips/create`)**:
  - The frontend `CreateTripClient.tsx` POSTs to `/api/trips/create`.
  - The backend derives the `creatorCompany.id` from the logged-in Auth User.
  - It inserts the trip into the database, hardcoding `company_id` to the creator, and using the `receiving_company_id` passed from the frontend form. The status is initialized to `'draft'`.
- **Review/Readback**:
  - The frontend holds the returned draft trip in local React state (`createdTrip`) to display the review screen. No further database fetch is made for the review step.
- **Publish Operation (`/api/trips/publish`)**:
  - The frontend POSTs the `trip_id` to `/api/trips/publish`.
  - The backend fetches the trip and enforces a strict ownership check: `existingTrip.company_id !== actingCompany.id`. **Only the Sending Company can publish the draft.**
  - Updates the status from `'draft'` to `'published'`. 
- **The Disconnect**: After the Sending Company successfully publishes the trip, they are redirected to the Dashboard. As discovered in Section 1, the Dashboard does not query by `company_id`. Therefore, the Sending Company can never see the trip they just created ever again.
