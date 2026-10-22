# DeliveryProof — API, Database & Source-Code Crosswalk

**Record type:** Companion engineering reference  
**Version:** 1.0  
**Prepared:** 22 October 2026  
**Application repository:** https://github.com/ayush22cp008/DeliveryProof_hackathon  
**Records repository:** https://github.com/ayush22cp008/Freight_Records  
**Companion master record:** [DeliveryProof_Detailed_Project_Record.md](./DeliveryProof_Detailed_Project_Record.md)

> **Evidence boundary:** This document is derived from checked-in route handlers, helpers, UI files and SQL migrations on the application repository’s inspected main snapshot. It describes what those files show; it is not a live database introspection, deployment test, or assertion that every endpoint has been re-executed. Historical acceptance status remains in the curated final verification records. Where source details differ from those records, the difference is recorded rather than silently reconciled.

---

## 1. How to use this reference

Use this file for three purposes:

1. **API tracing:** find the route that serves a product action, what data it reads/writes, and the checks visible in the handler.
2. **Database reasoning:** understand the entities, foreign-key relationships, status/event vocabulary, and historical schema evolution represented by the migrations.
3. **Feature navigation:** move from a product capability to its page/client component, API, helper, and migration.

The main source folders are:

- Application UI: src/app/
- API handlers: src/app/api/
- Shared helpers: src/lib/
- SQL migrations: src/db/migrations/
- Existing test source: tests/

The source snapshot inspected during preparation showed application commit reference 7d7180569fec6e9a0bbe655218dc13716c55f7fb, with message “Change presentation deck link to Google Slides,” dated 15 September 2026. The curated Day 21 records were updated later in September. See Section 10 for explicit differences and review items.

---

## 2. API conventions and identity flow

### Session and role resolution

Most protected API handlers use the cookie/session-aware client from src/lib/supabase/server.ts to call Supabase Auth, then use server-side Supabase operations through src/lib/supabase-server.ts. The shared helper src/lib/auth.ts resolves the current user’s freight_identities row.

A common protected flow is:

~~~text
Request
  → read authenticated Supabase user
  → resolve FreightIdentity / trusted role where used
  → resolve Company or Driver row from authenticated user ID
  → verify that the user is related to the Trip/request
  → verify allowed current state and any prerequisite event
  → read/write through server-side Supabase operations
~~~

This is the intended shape, but each handler has its own exact guards. A route should not be described as checking a rule merely because another route checks it.

### Common response pattern

The route handlers use conventional HTTP responses such as:

| Status | Use visible in source |
|---|---|
| 400 | Missing/invalid request data, unmet lifecycle prerequisite or invalid state |
| 401 | Missing user session or missing actor profile in some handlers |
| 403 | Authenticated user lacks required role/resource access |
| 404 | Required resource/profile is not found or is intentionally hidden |
| 409 | Trip/request is no longer claimable/pending or duplicate single-occurrence event |
| 429 | Anonymous public verification rate limit exceeded |
| 500 | Unexpected database/provider/server failure |

These are examples from the actual handlers, not a promise that every endpoint returns every status.

---

## 3. API-by-API index

### 3.1 Authentication and onboarding

| API route | Method | Source files | Responsibility and visible checks |
|---|---|---|---|
| /api/auth/signup | POST | src/app/api/auth/signup/route.ts; migration 004 | Requires email, password and requested_role. Role is limited to DRIVER or COMPANY. Calls Supabase Auth signUp and places requested_role in user metadata; the database trigger is intended to create freight_identities. Returns success when an Auth user is returned. |
| /api/auth/login | POST | src/app/api/auth/login/route.ts | Requires email and password; calls signInWithPassword; returns 401 for invalid credentials. |
| /api/auth/logout | POST | src/app/api/auth/logout/route.ts | Calls Supabase Auth signOut for the request session. |
| /api/auth/forgot-password | POST | src/app/api/auth/forgot-password/route.ts | Requires email; constructs an update-password callback URL from the request origin and calls resetPasswordForEmail. It returns success even when the reset provider call errors, to avoid exposing whether an email exists. |
| /api/onboarding/submit | POST | src/app/api/onboarding/submit/route.ts; src/app/(authenticated)/onboarding/page.tsx; migrations 004–005 | Requires a current identity in an allowed submission state (PENDING or REJECTED); accepts document_type and storage_path; writes onboarding_evidence using the authenticated identity and its requested role. For a resubmission after REJECTED, the handler attempts to return identity state to PENDING. The actual file upload is a separate client/Storage flow; this endpoint records the evidence metadata. |
| /api/admin/review | POST | src/app/api/admin/review/route.ts; migrations 011–012 | Requires authenticated user and reviewer_authorizations membership. Accepts identity_id, action APPROVE or REJECT, and optional rejection reason; calls process_reviewer_decision through Supabase RPC. Maps missing identity/not-pending RPC errors to client responses. Historical test-only migration 013 was removed from current main on 22 Oct; deployed database state was not queried. |
| /api/admin/history | GET | src/app/api/admin/history/route.ts; migration 011 | Requires authenticated user and reviewer_authorizations membership. Queries reviewer_decisions newest-first and includes related identity/evidence. Accepts optional limit, offset and id query parameters; returns a transformed history data array. |

### 3.2 Company lookup and Receiver agreement

| API route | Method | Source files | Responsibility and visible checks |
|---|---|---|---|
| /api/companies/lookup | GET | src/app/api/companies/lookup/route.ts; Company Trip-create UI | Requires VERIFIED trusted COMPANY identity. Returns company id/name choices while excluding the current sender company where its profile is found. |
| /api/receiver-request/accept | POST | src/app/api/receiver-request/accept/route.ts; migration 009 | Requires authenticated VERIFIED COMPANY identity; derives the Company ID from the user session; accepts requestId; updates only a matching request whose state is PENDING and whose receiving_company_id matches the acting Company. On stale/non-owned requests it returns 409. |
| /api/receiver-request/reject | POST | src/app/api/receiver-request/reject/route.ts; migration 009 | Uses the same role, profile, receiver relationship and PENDING-state gates as accept; accepts requestId and optional reason; writes REJECTED, rejection_reason, decided_at and decided_by. |
| /api/trips/create | POST | src/app/api/trips/create/route.ts; migration 006_node3_trip_schema.sql; migration 009 | Requires authenticated VERIFIED COMPANY identity, resolves sender Company server-side, validates required Trip fields and receiving_company_id, rejects same-company Sender/Receiver for new Trips with HTTP 400, inserts a DRAFT Trip, then creates a PENDING receiver_delivery_requests record. If the request insert fails, the handler attempts to delete the newly created Trip as compensation; those are sequential REST calls, not evidence of a single database transaction. |
| /api/trips/publish | POST | src/app/api/trips/publish/route.ts; migration 009 | Requires verified Company; resolves the acting Company from session; fetches the Trip, checks sender ownership and DRAFT state, requires an ACCEPTED Receiver agreement, then changes the Trip status to published. |
| /api/trips/claim | POST | src/app/api/trips/claim/route.ts; ClaimTripButton.tsx | Requires authenticated VERIFIED DRIVER identity; resolves Driver profile through Auth ID; requires an ACCEPTED receiver request; conditionally updates only the requested Trip if status is published and driver_id is NULL. A failed/no-row conditional update returns 409, the no-longer-available response. |

### 3.3 Delivery-event APIs

All canonical event routes below are POST handlers. In the checked-in source, the Driver event handlers resolve the Driver from the authenticated user and query a Trip assigned to that Driver in active, claimed or in_progress state. They write an event row with event type, location fields and timestamp. The exact prerequisite check differs per endpoint; see the table rather than assuming every event has the same prerequisite.

| API route | Writes event type | Prerequisite shown in source | Role/relationship gate |
|---|---|---|---|
| /api/events/arrival | ARRIVED_AT_PICKUP | Requires latitude, longitude, server_timestamp and photo_url in its input validation; checks for an eligible Trip assigned to current Driver. | Authenticated Driver profile; eligible Trip must be assigned to that Driver. |
| /api/events/checkin | PICKUP_CHECKED_IN | Requires evidence fields and an eligible Trip; no separate earlier-event gate is visible in the inspected route summary. | Authenticated Driver profile; eligible Trip assigned to that Driver. |
| /api/events/load | GOODS_LOADED | Requires a prior check-in event: legacy checkin or canonical PICKUP_CHECKED_IN. | Authenticated Driver profile; eligible Trip assigned to that Driver. |
| /api/events/pickup-departed | PICKUP_DEPARTED | Requires GOODS_LOADED to exist. | Authenticated Driver profile; eligible Trip assigned to that Driver. |
| /api/events/in-transit | IN_TRANSIT | Requires pickup departure: legacy departure or canonical PICKUP_DEPARTED. | Authenticated Driver profile; eligible Trip assigned to that Driver. |
| /api/events/arrived-at-delivery | ARRIVED_AT_DELIVERY | Requires IN_TRANSIT to exist. | Authenticated Driver profile; eligible Trip assigned to that Driver. |
| /api/events/receiver-checkin | RECEIVER_CHECKED_IN | Requires a valid Trip ID, VERIFIED trusted COMPANY identity, the acting Company must equal receiving_company_id, and ARRIVED_AT_DELIVERY must exist. The event associates the event row with the Trip’s assigned Driver. | Receiving Company for that Trip—not an arbitrary Company. |
| /api/events/goods-unloaded | GOODS_UNLOADED | Requires RECEIVER_CHECKED_IN. | Authenticated Driver profile; eligible Trip assigned to that Driver. |
| /api/events/delivery-departed | DELIVERY_DEPARTED | Requires GOODS_UNLOADED. | Authenticated Driver profile; eligible Trip assigned to that Driver. |
| /api/events/departure | departure (legacy lowercase) | Historical event route; accepts evidence fields and an eligible Driver Trip. | Authenticated Driver profile; eligible Trip assigned to that Driver. |

**Event payload notes from inspected handlers**

- The event handlers accept latitude, longitude, gps_accuracy where supplied, server_timestamp, and optional photo_url; individual validation varies. The older arrival endpoint uniquely checks photo_url as required in its missing-field guard.
- Most Driver event routes do not visibly take a Trip ID in their active-Trip lookup; they resolve a currently eligible Trip for the authenticated Driver. The Receiver check-in route explicitly receives trip_id.
- Single-occurrence event conflicts are handled as a conflict response in the route handlers. The database also defines UNIQUE (trip_id, event_type).
- The legacy /api/events/departure route remains alongside canonical /api/events/pickup-departed and /api/events/delivery-departed. It writes the legacy lowercase value, which remains accepted by the schema and is recognized by compatibility logic in the AI summary route.

### 3.4 Completion, photo evidence, AI summary and public verification

| API route | Method | Source files | Responsibility and visible checks |
|---|---|---|---|
| /api/completion/driver | POST | src/app/api/completion/driver/route.ts; DriverCompletionClient.tsx; migration 008 | Requires authenticated Driver profile and a Trip associated with that Driver. Requires DELIVERY_DEPARTED before writing Driver completion confirmation. The handler then attempts to reconcile the two confirmation timestamps and Trip status. |
| /api/completion/receiver | POST | src/app/api/completion/receiver/route.ts; ReceiverCompletionClient.tsx; migration 008 | Requires authenticated VERIFIED COMPANY identity; resolves Company from Auth ID; verifies Trip belongs to that receiving Company; requires DELIVERY_DEPARTED; writes Receiver confirmation and checks the dual-confirmation state before Trip completion. |
| /api/upload-photo | POST | src/app/api/upload-photo/route.ts; src/lib/capture/uploadPhoto.ts; migration 007 | Accepts multipart photo and trip_id. It checks Trip relationship for a Receiver Company or Driver and uploads to the event-photos bucket. Returns a Storage public URL. Source-level guard nuance is recorded in Section 10. |
| /api/summary | POST | src/app/api/summary/route.ts; src/lib/summary.ts; AIEvidenceSummary.tsx | Requires an authenticated user with a Driver profile; accepts optional tripId; queries Trip status in active, claimed, in_progress or completed; loads ordered events; checks broad Arrival/Check-in/Departure evidence; passes structured event fields to Groq-backed summary generation. The current route does not visibly require trips.status to equal completed; see Section 10. |
| /api/trips/[tripId]/public-share | POST | src/app/api/trips/[tripId]/public-share/route.ts; PublicShareManager.tsx; migrations 006_create_trip_public_shares.sql | Requires an authenticated Company profile that is the Receiving Company on a completed Trip; requires DELIVERY_DEPARTED; creates a random token, stores its hash, revokes a previous ACTIVE share via a separate update, inserts a new ACTIVE share and returns the public URL. The sequence is not shown as one database transaction. |
| /api/trips/[tripId]/public-share | DELETE | Same as above | Authenticated Receiving Company for the Trip can revoke ACTIVE public shares. |
| /api/public/verify/[token] | GET | src/app/api/public/verify/[token]/route.ts; src/lib/public-share-lookup.ts; src/lib/public-share.ts; share/[token]/page.tsx | Anonymous read endpoint. Applies in-memory rate limiting, hashes/looks up the supplied token via the helper, returns 404 for invalid/revoked share tokens, and returns a limited public projection with no-store headers. |
| /api/server-time | GET | src/app/api/server-time/route.ts; src/lib/capture/getServerTime.ts | Returns the server’s current ISO timestamp. The frontend helper fetches it with cache: no-store. This endpoint itself does not require an authenticated session. |

---

## 4. API authorization / mutation details that deserve care

### 4.1 Driver claim is conditional, not “check then blindly write”

The inspected claim route conditions its write on all three predicates:

~~~text
id = requested Trip
status = published
driver_id IS NULL
~~~

The authenticated Driver ID is resolved server-side. If the conditional update returns no Trip, the route returns a 409 conflict response. This matches the recorded manual acceptance in which two authenticated Drivers competed for one published Trip and exactly one succeeded.

The project’s curated records say the separate automated local race-test framework was deferred. The currently checked-in tests/concurrency.test.ts contains a concurrent conditional-update test, but this documentation pass has not executed that test. See Section 10.

### 4.2 Receiver accept/reject is conditionally updated

Both Receiver request handlers condition the update on request ID, state PENDING and receiving_company_id equal to the acting Company. This avoids treating a stale request or a request for another Company as an accepted/rejected request.

### 4.3 Creation is not one demonstrated transaction

The Trip-create handler first inserts a draft Trip, then inserts a receiver request. If request creation fails, it attempts a compensating Trip delete. The source shows a failure-handling sequence, not a single Postgres transaction across both REST calls. Do not call this operation strictly atomic without a separate database transaction implementation or test evidence.

### 4.4 Reviewer decisions use a database RPC

The admin review route calls process_reviewer_decision in migration 012. The SQL function updates the evidence row, changes freight_identities verification state and role, inserts reviewer_decisions history and creates the Driver or Company business row as applicable. The curated Day 20 verification describes forced-exception rollback tests and a successful production retry. The separate checked-in test migration remains a source-record reconciliation issue.

---

## 5. Database schema reference from checked-in migrations

**Important:** This section describes the schema as reconstructed from migration files. The files include incremental and historical changes. It is not a dump of the current production database, and it should not be applied to a database from top to bottom without reviewing the migration history and current state.

### 5.1 Entity/table inventory

| Table / object | Important columns represented in migrations | Purpose |
|---|---|---|
| auth.users | Supabase-managed Auth table | Root identity for authenticated users. Referenced by identity, Driver, Company, Reviewer, evidence and share records. |
| freight_identities | id, auth_id UNIQUE, requested_role, verification_status, trusted_role, email, created_at, updated_at; reviewed_at added by migration 010 | One application identity and requested/trusted role state for an Auth user. Signup trigger creates the identity. |
| drivers | id, driver_code UNIQUE, name, created_at; auth_id UNIQUE references auth.users and is nullable in migration 003 | Operational Driver record. Created during approved Reviewer flow in the RPC. |
| companies | id, auth_id UNIQUE, name, created_at | Company profile associated to an Auth user. Sender and Receiver are per-Trip roles, not separate identity tables. |
| reviewer_authorizations | auth_id primary key references auth.users, granted_at | Authorization allowlist for Reviewer endpoints. |
| onboarding_evidence | id, auth_id, role_type, document_type, storage_path, mime_type, size_bytes, version, rejection_reason, status, created_at | Uploaded identity/role evidence metadata. File itself is in Storage. |
| trips | id, driver_id, facility_name, status, created_at; later fields include company_id, receiving_company_id, destination_name, distance, duration, payout, driver_completion_confirmed_at, receiver_delivery_confirmed_at | Central delivery object: sender, receiver, assigned Driver, lifecycle and final confirmations. |
| events | id, trip_id, driver_id, event_type, latitude, longitude, gps_accuracy, server_timestamp, photo_url, created_at; UNIQUE (trip_id, event_type) | Chronological delivery event/evidence record. |
| receiver_delivery_requests | id, trip_id, sender_company_id, receiving_company_id, state, rejection_reason, created_at, decided_at, decided_by | Explicit Receiver agreement gate before publication. State enum PENDING / ACCEPTED / REJECTED. |
| reviewer_decisions | id, identity_id, auth_id, evidence_id, decision, rejection_reason, reviewed_at | Persistent verification decision history. |
| trip_public_shares | id, trip_id, token_hash UNIQUE, status ACTIVE/REVOKED, created_by, created_at, revoked_at | Token-based public verification sharing for completed Trips. |
| Storage bucket onboarding_evidence | private bucket configured in migration 005 | Private Driver licence / Company GST evidence. |
| Storage bucket event-photos | bucket configured as public in migration 007 | Trip event photos; upload route returns a public URL. |

### 5.2 Relationship diagram

~~~text
auth.users
  ├── 0..1 freight_identities       (auth_id UNIQUE; CASCADE on Auth-user delete)
  ├── 0..1 drivers                  (auth_id UNIQUE; SET NULL on Auth-user delete)
  ├── 0..1 companies                (auth_id UNIQUE; CASCADE on Auth-user delete)
  ├── 0..1 reviewer_authorizations  (auth_id primary key)
  ├── many onboarding_evidence      (auth_id FK; CASCADE)
  ├── many reviewer_decisions       (auth_id FK; CASCADE)
  └── many trip_public_shares       (created_by FK; SET NULL)

companies
  ├── many trips as Sender           (trips.company_id)
  ├── many trips as Receiver         (trips.receiving_company_id)
  ├── many receiver requests sent    (sender_company_id)
  ├── many receiver requests received(receiving_company_id)
  └── receiver request decisions     (decided_by; SET NULL on Company delete)

drivers
  ├── many trips assigned            (trips.driver_id; nullable before claim)
  └── many events                    (events.driver_id)

trips
  ├── many events                    (events.trip_id)
  ├── receiver agreement request(s)  (receiver_delivery_requests.trip_id)
  └── public share records            (trip_public_shares.trip_id)

freight_identities
  └── reviewer_decisions              (identity_id; CASCADE)

onboarding_evidence
  └── reviewer_decisions              (evidence_id; SET NULL on evidence delete)
~~~

The database has a UNIQUE (trip_id, event_type) constraint for events in the migrations, so a Trip can have at most one row for each event_type. The receiver request table has a partial unique index allowing at most one PENDING request per Trip. The public-share table has a partial unique index allowing at most one ACTIVE share per Trip.

### 5.3 Table detail and schema-evolution notes

#### freight_identities

Created by migration 004. Important values:

- requested_role defaults to DRIVER and is initially copied from Auth user metadata on signup;
- verification_status defaults to PENDING;
- trusted_role is set on approval by the Reviewer RPC;
- auth_id is unique and references auth.users with ON DELETE CASCADE;
- reviewed_at is added by migration 010.

The policy visible in migration 004 allows users to SELECT their own identity. Protected write paths use server-side operations/RPCs.

#### drivers

Migration 001 defines id, driver_code, name and created_at. Migration 003 adds nullable unique auth_id with ON DELETE SET NULL. The approved Reviewer decision RPC inserts the business row for approved Driver identities.

#### companies

Migration 005 defines id, unique auth_id, name and created_at. Its table policy allows the user to read their own Company profile. The Reviewer RPC creates the Company business row on approval.

#### trips

Migration 001 initially defines driver_id as NOT NULL because the first prototype already assigned a Driver. Migration 006_node3_trip_schema.sql changes driver_id to nullable so a Trip can exist before marketplace assignment, adds company_id and receiving_company_id, destination_name, distance, duration and payout, and expands the status check to active, draft, published, claimed, in_progress and completed.

Migration 008 adds driver_completion_confirmed_at and receiver_delivery_confirmed_at. The currently inspected migrations do not add a database CHECK constraint for company_id != receiving_company_id; the inspected new-Trip route enforces that rule in server-side application logic, and the curated manual test records the API rejection.

#### events

Migration 002 initially supports lowercase arrival, checkin and departure. Migration 008 expands the allowed vocabulary to retain those historic values and accept the nine canonical milestones. Each event has a Trip FK, Driver FK, event type, latitude and longitude, optional gps_accuracy and photo_url, server_timestamp and created_at. UNIQUE (trip_id, event_type) preserves single occurrence for a canonical event within this one-delivery model.

Migration 002 enables RLS and explicitly revokes UPDATE and DELETE on events from PUBLIC, anon, authenticated and service_role. This reflects the append-only evidence intent in the migration. Any claim about the live effective grants still requires checking the deployed database.

#### receiver_delivery_requests

Migration 009 defines a PENDING / ACCEPTED / REJECTED enum and the request table. It links the Trip, sending Company and receiving Company. decided_by references companies and becomes NULL if that Company is deleted. The partial index one_pending_request_per_trip allows at most one PENDING request per Trip; additional indexes cover trip_id, receiving_company_id and sender_company_id.

The migration backfills an ACCEPTED request for existing non-draft Trips that have no request record, using the Trip’s created_at as the historical decision timestamp.

#### reviewer_decisions

Migration 011 adds decision history and SELECT policies for the Reviewer allowlist and the subject user. decision is text with a source comment that expected values are VERIFIED or REJECTED; this migration does not express those values as a CHECK constraint. evidence_id becomes NULL if the referenced onboarding_evidence row is deleted.

Migration 012 defines process_reviewer_decision as a SECURITY DEFINER function. The helper is responsible for mutating evidence, identity state, reviewer_decisions and the approved Driver/Company row. Migration 013 is the test-only forced-failure function and is a reconciliation item, not accepted evidence of current production state.

#### trip_public_shares

Migration 006_create_trip_public_shares.sql defines token_hash as unique, a status enum ACTIVE / REVOKED, an optional created_by Auth user, creation/revocation timestamps and a Trip FK. The partial unique index unique_active_share allows only one ACTIVE share per Trip. Its table is RLS-enabled, with a SELECT policy for the receiving Company related to the Trip.

#### Storage buckets

- onboarding_evidence is configured with public=false, with policies for the owner and authorized Reviewer to insert/read their onboarding files.
- event-photos is configured with public=true. The upload route obtains a public URL from Storage after upload. This distinction is important when reviewing privacy assumptions around evidence photos.

---

## 6. SQL migration index and history

| Migration file | Schema change represented |
|---|---|
| 001_create_core_tables.sql | Initial drivers and trips tables; RLS enabled. |
| 002_create_events_table.sql | Initial event table; event fields; trip/event uniqueness; evidence update/delete revocations. |
| 003_add_auth_id_to_drivers.sql | Adds unique Driver → Auth user relationship, nullable on Auth-user deletion. |
| 004_create_freight_identities.sql | Adds freight_identities, own-row SELECT policy and Auth signup trigger. |
| 005_v2_onboarding_evidence.sql | Creates/recreates companies, reviewer_authorizations and onboarding_evidence; private onboarding Storage bucket and Storage policies. |
| 006_create_trip_public_shares.sql | Adds public share tokens, ACTIVE / REVOKED status and active-share uniqueness. |
| 006_node3_trip_schema.sql | Makes Trip driver_id nullable; adds Sender/Receiver and route fields; expands Trip status values. |
| 007_event_photos_bucket.sql | Creates public event-photos bucket. |
| 008_node5_delivery_evidence_schema.sql | Expands event_type vocabulary and adds dual completion confirmation timestamps. |
| 009_receiver_delivery_requests.sql | Adds Receiver agreement request enum/table, state index, relationship indexes and legacy backfill. |
| 010_add_reviewed_at.sql | Adds freight_identities.reviewed_at. |
| 011_reviewer_decisions.sql | Adds persistent Reviewer decision history. |
| 012_reviewer_decision_rpc.sql | Defines process_reviewer_decision RPC. |
| 013_reviewer_decision_test_rpc.sql | Historical test-only forced-exception RPC. Removed from current main in commit `3c61efc...`; file remains in Git history. Removing the source file does not query or change a deployed database function. |

The files describe migrations with manual Supabase SQL Editor comments and include historical/incremental schema changes. Do not assume the listed files can be blindly re-run against any environment; inspect the deployed schema and applied migration state first.

---

## 7. Feature-to-source-code crosswalk

### 7.1 Shared authentication / identity

| Feature | UI / client | API / helper | Database basis |
|---|---|---|---|
| Signup / login / logout / reset | src/app/signup/page.tsx; src/app/login/page.tsx; src/app/forgot-password/page.tsx; src/app/update-password/page.tsx | src/app/api/auth/signup/route.ts; login/route.ts; logout/route.ts; forgot-password/route.ts | auth.users; freight_identities and signup trigger in migration 004 |
| Identity lookup / trusted role | Shared authenticated layouts and role pages | src/lib/auth.ts | freight_identities |
| Onboarding form / evidence submission | src/app/(authenticated)/onboarding/page.tsx | src/app/api/onboarding/submit/route.ts; Supabase browser/server clients | freight_identities; onboarding_evidence; private onboarding_evidence bucket |
| Role rejection guard / navigation | src/app/(authenticated)/RejectedGuard.tsx; Navbar.tsx | Shared auth helpers and API-specific checks | freight_identities.verification_status / trusted_role |

### 7.2 Reviewer identity verification

| Feature | UI / client | API / helper | Database basis |
|---|---|---|---|
| Verification queue | src/app/(authenticated)/reviewer/queue/page.tsx; reviewer/page.tsx | Reviewer list/read calls are routed through current app API and database operations | freight_identities; onboarding_evidence; reviewer_authorizations |
| Applicant detail and evidence review | src/app/(authenticated)/reviewer/verify/[id]/page.tsx; ApplicantVerificationClient.tsx; src/app/(authenticated)/reviewer/history/[id]/EvidenceViewerClient.tsx | src/app/api/admin/review/route.ts | process_reviewer_decision in migration 012 |
| Persistent decision history | src/app/(authenticated)/reviewer/history/page.tsx and history/[id]/page.tsx | src/app/api/admin/history/route.ts | reviewer_decisions joins to freight_identities and onboarding_evidence |
| Forced rollback test | Not a user-facing feature | Migration 013 function process_reviewer_decision_test | DP-REC-01; do not assume it is deployed or removed without database verification |

### 7.3 Company / Receiver / marketplace

| Feature | UI / client | API / helper | Database basis |
|---|---|---|---|
| Company dashboard | src/app/(authenticated)/page.tsx; company profile UI | Server-side authenticated queries | companies; trips |
| Create Trip | src/app/(authenticated)/company/trips/create/page.tsx; CreateTripClient.tsx | /api/companies/lookup; /api/trips/create | companies; trips |
| Receiver inbox / accept or reject | src/app/(authenticated)/company/incoming/page.tsx; ReceiverRequestActions.tsx if present in the current tree | /api/receiver-request/accept; /api/receiver-request/reject | receiver_delivery_requests |
| Publish Trip | Company created/history and Trip detail pages | /api/trips/publish | trips; receiver_delivery_requests |
| Driver marketplace / claim | src/app/(authenticated)/driver/available/page.tsx; ClaimTripButton.tsx | /api/trips/claim | drivers; trips; receiver_delivery_requests |
| Company Trip detail / history | src/app/(authenticated)/company/trips/[id]/page.tsx; company/created/page.tsx; company/history/page.tsx; CompanyHistoryClient.tsx | Server-side Supabase queries; associated action APIs | companies; trips; events |
| Receiver check-in / completion | src/app/(authenticated)/company/receiver-checkin/page.tsx; ReceiverCheckinClient.tsx; company/completion/page.tsx; ReceiverCompletionClient.tsx | /api/events/receiver-checkin; /api/completion/receiver | trips; events; receiver relationship; dual confirmation timestamps |

### 7.4 Driver delivery execution

| Feature | UI / client | API / helper | Database basis |
|---|---|---|---|
| Active Trip | src/app/(authenticated)/driver/active/page.tsx | Server-side Trip resolution and event APIs | drivers; trips; events |
| Trip history / timeline | driver/history/page.tsx; driver/trip/[id]/page.tsx; timeline/page.tsx | Queries events and Trip state | trips; events |
| Arrival | events/arrival/page.tsx; ArrivalClient.tsx | /api/events/arrival | events; trip/driver relation |
| Pickup check-in | events/checkin/page.tsx; CheckinClient.tsx | /api/events/checkin | events |
| Load goods | events/load/page.tsx; LoadClient.tsx | /api/events/load | events; check-in prerequisite |
| Pickup departure | events/pickup-departed/page.tsx; PickupDepartedClient.tsx | /api/events/pickup-departed | events; GOODS_LOADED prerequisite |
| Transit | events/in-transit/page.tsx; InTransitClient.tsx | /api/events/in-transit | events; pickup-departure prerequisite |
| Destination arrival | events/arrived-at-delivery/page.tsx; ArrivedAtDeliveryClient.tsx | /api/events/arrived-at-delivery | events; IN_TRANSIT prerequisite |
| Unload goods | events/goods-unloaded/page.tsx; GoodsUnloadedClient.tsx | /api/events/goods-unloaded | events; Receiver check-in prerequisite |
| Delivery departure | events/delivery-departed/page.tsx; DeliveryDepartedClient.tsx | /api/events/delivery-departed | events; GOODS_UNLOADED prerequisite |
| Legacy departure | events/departure/page.tsx; DepartureClient.tsx | /api/events/departure | Legacy lowercase departure event support |
| Completion | src/app/(authenticated)/completion/driver/page.tsx; DriverCompletionClient.tsx | /api/completion/driver | trips completion confirmation fields; DELIVERY_DEPARTED event |

### 7.5 Evidence capture, AI and external verification

| Feature | UI / client | API / helper | Database basis |
|---|---|---|---|
| GPS capture | Event-specific client components | src/lib/capture/getGpsLocation.ts | Captured coordinates and accuracy stored on events |
| Time capture | Event-specific client components | src/lib/capture/getServerTime.ts; /api/server-time | ISO timestamp is fetched and then included in event requests; current handlers accept it as input |
| Photo upload | Event-specific components; src/lib/capture/uploadPhoto.ts | /api/upload-photo | event-photos Storage bucket; event row has photo_url |
| AI Evidence Summary | src/components/AIEvidenceSummary.tsx; timeline and Trip surfaces | /api/summary; src/lib/summary.ts | Events queried chronologically; Groq generated summary is response data, not a deterministic database event |
| Create/revoke public evidence share | src/app/(authenticated)/company/PublicShareManager.tsx | /api/trips/[tripId]/public-share | trip_public_shares |
| Public evidence page | src/app/share/[token]/page.tsx | /api/public/verify/[token]; src/lib/public-share-lookup.ts; src/lib/public-share.ts | token hash, Trip, Receiving Company, event timeline |
| Source manifest / docs | README.md; package.json; migrations | Source tree and curated Freight_Records | Project-specific; README itself states future scope and known security caveat |

Paths in the crosswalk describe the visible tree; if a referenced optional client filename differs in the checked-out revision, use the current repository tree as the authority. The business responsibility stays tied to the API/DB path described above.

---

## 8. Event capture sequence and prerequisite map

~~~text
Driver Arrival
  └─ writes ARRIVED_AT_PICKUP
       ↓
Pickup Check-in
  └─ writes PICKUP_CHECKED_IN
       ↓
Goods Loaded
  └─ requires PICKUP_CHECKED_IN (or legacy checkin)
       ↓
Pickup Departed
  └─ requires GOODS_LOADED
       ↓
In Transit
  └─ requires PICKUP_DEPARTED (or legacy departure)
       ↓
Arrived at Delivery
  └─ requires IN_TRANSIT
       ↓
Receiver Check-in
  └─ Receiving Company for this Trip; requires ARRIVED_AT_DELIVERY
       ↓
Goods Unloaded
  └─ requires RECEIVER_CHECKED_IN
       ↓
Delivery Departed
  └─ requires GOODS_UNLOADED
       ↓
Driver + Receiving Company confirmation
  └─ both confirmation timestamps recorded
       ↓
Trip status completed
~~~

This is the accepted canonical workflow and the checked-in handler prerequisite map. The older lowercase arrival/checkin/departure values remain in the schema for historical compatibility.

---

## 9. Source-level behavior notes (read before relying on wording)

### 9.1 Event timestamp path

The UI helper src/lib/capture/getServerTime.ts calls /api/server-time and receives an ISO timestamp. The event handlers also read server_timestamp from the POST body and insert that value into events. Thus the helper obtains a server-generated time, but the inspected event route does not visibly recompute/overwrite the timestamp inside the same mutation. For any statement requiring a strictly tamper-resistant server receipt timestamp, treat this as a verification/review question rather than assume that the database timestamp column name makes the submitted value immutable.

### 9.2 AI summary endpoint completion gate — source and Vercel check successful; runtime test pending

The accepted project records define AI Evidence Summary as a post-completion feature. The prior source route queried active, claimed, in_progress and completed Trips, which did not match that rule. On 22 October 2026, commit `bc4a2e76d57e772f54bd89722f21d7745aa3d6e5` changed the query to `.eq('status', 'completed')` and changed the no-match message to “No completed trip found.” The current Git source contains the completed-only condition, and a follow-up GitHub check reports Vercel success for the commit. Direct authenticated behavior of the later post-submission source edit was not manually re-tested during this records review. Ayush confirms the submitted version's functional workflow was tested and working before submission.

### 9.3 Arrival photo requirement differs from other event handlers

The inspected /api/events/arrival route’s input guard requires photo_url alongside latitude, longitude and server_timestamp. Other inspected canonical event routes allow the photo_url field to be absent. This is a source-level behavior difference worth preserving when troubleshooting capture UX.

### 9.4 Photo bucket visibility

Migration 007 creates event-photos with public=true and /api/upload-photo returns a public URL. The public verification projection currently emits only a filtered timeline and does not include raw photo URLs, but uploaded object visibility is a separate Storage concern. Confirm the intended privacy policy before changing bucket visibility or describing photos as private.

### 9.5 In-memory anonymous rate limiting

src/lib/public-share.ts stores rate-limit records in a module-level Map with a 60-second window and a limit of 10 requests for each IP+token key. The public verify API uses the x-forwarded-for header and falls back to anonymous if absent. The implementation is in-memory, not a shared database/Redis counter; its behavior can vary across server instances/restarts. Describe it as a basic anonymous rate limiter, not globally coordinated enforcement.

### 9.6 Public share projection is deliberately narrower than the private full timeline

src/lib/public-share-lookup.ts validates an ACTIVE hashed token and returns a projection with receiving-company name, Trip status, delivery date, pickup/destination labels, evidence checklist, a timeline filtered to ARRIVED_AT_DELIVERY / RECEIVER_CHECKED_IN / DELIVERY_DEPARTED, AI summary and isPublicVerification=true. It does not return the full nine-event sequence or raw event GPS/photo fields. The summary generation failure path falls back to “AI summary unavailable.” and does not fail the public verification response.

### 9.7 New Trip Sender/Receiver rule is visibly enforced in route code

The inspected /api/trips/create handler checks creatorCompany.id === receiving_company_id and returns HTTP 400 for new same-company Trips. The inspected migrations do not add a database CHECK constraint for the invariant. The curated Day 21 manual record separately reports the direct API test. Do not describe the invariant as a schema-level CHECK unless a current migration/database inspection provides that evidence.

### 9.8 Reviewer test-only migration reconciliation

Migration 013_reviewer_decision_test_rpc.sql was removed from the current main tree in commit `3c61efc...`. Historical Git still contains the original file, as expected. This resolves the source-tree discrepancy; the deployed database was not queried in this pass, so the presence or absence of a previously applied test function is not independently confirmed here.

### 9.9 Public share replacement uses sequential operations

The share-creation API revokes an existing ACTIVE record, then inserts a new ACTIVE record. The partial unique index protects the one-active-share invariant, but the two calls are not shown as one transaction. Treat the unique index as a safeguard; do not call the entire operation fully atomic.

---

## 10. Open reconciliation and verification register

Items below are technical review points discovered while building this reference. They are not statements that production is currently broken.

| ID | Observation | What the source proves | What remains unproven / next check |
|---|---|---|---|
| API-DB-01 | Test-only Reviewer migration | Migration 013 exists in the checked-in source and defines a deliberately failing RPC. | Whether that migration/function exists in the deployed DB; reconcile the source file with the curated claim that it was removed. |
| API-TEST-02 | Claim-concurrency test | tests/concurrency.test.ts exists and contains a concurrent conditional-update test requiring seed data/environment. | Whether the test has actually run successfully; keep manual two-Driver acceptance separate from automated execution. |
| API-SEC-03 | Event timestamps | UI helper fetches server time; event APIs accept server_timestamp in submitted JSON. | Whether an attacker can alter the timestamp in the deployed flow; decide whether the server route/DB should assign authoritative ingestion time. |
| API-AI-04 | Direct summary eligibility | Fixed in source commit `bc4a2e76d57e772f54bd89722f21d7745aa3d6e5`: `/api/summary` now filters to `status = completed`; Vercel check reports success. | A separate manual endpoint test after this later source edit was not performed in this records review; this is distinct from Ayush's pre-submission test evidence. |
| API-STOR-05 | Event-photo bucket public flag | Migration 007 sets event-photos public=true; upload endpoint returns a public URL. | Whether this is intended for every event photo and matches current privacy expectations. |
| API-RATE-06 | Anonymous verification rate limiter | A module-local Map limits requests by IP+token key. | Whether this is sufficient for multi-instance deployment; no shared store is shown in the inspected helper. |
| API-SHARE-07 | Share replacement sequence | API updates old share, then inserts replacement; database has partial unique index. | Whether a transactional RPC is necessary for the desired consistency guarantees. |
| API-STATUS-08 | Source snapshot vs final records | Source cleanup and completed-only Summary gate were committed to main on 22 October; Vercel check reports success for the latest source change. | Exact commit used by the historical final E2E and direct post-reconciliation live API verification remain unconfirmed. |
| API-DOC-09 | Demo-account documentation | README contains test-account credentials; final curated status says public credentials were deferred to final pre-submission. | Verify these are disposable, intentionally public demo accounts and not access to production/personal data before distributing README. |

Do not mark an item closed just because a comment or later summary says it was fixed. Close it with a current source diff, actual database/deployment check, or new test record appropriate to the question.

---

## 11. Interview-level technical narratives

### “How did you prevent two Drivers from claiming the same Trip?”

The API resolves the Driver from the authenticated session and submits a conditional update that succeeds only when the Trip is published and has no Driver. The response returns a conflict when no row is available. The curated project verification records two authenticated sessions competing for the same Trip and exactly one successful claim. A separate automated concurrency file is present in source, but its successful execution is not established by this documentation pass.

### “Why are Trip status and event type separate?”

Trip status represents the overall operational state such as draft, published, claimed, in_progress and completed. The events table records detailed milestones—arrival, check-in, goods loading, transit and departure—with evidence metadata. IN_TRANSIT is a canonical event milestone, not a separate major trips.status in the accepted architecture. This separation allows the high-level state and chronological evidence to remain distinct.

### “What does the AI do?”

The /api/summary route loads structured events in timestamp order and sends a defined evidence payload to Groq. The prompt says to summarize only facts present, preserve sequence/timestamps, and avoid invented photo details or blame/causality. AI provides a readable narrative; deterministic state transitions and the underlying evidence records remain the operational source of truth.

### “How does identity verification work?”

Supabase Auth signup includes requested_role metadata. A trigger creates freight_identities in PENDING state. The applicant submits evidence metadata backed by Storage. An allowlisted Reviewer inspects the applicant and commits Approve/Reject through process_reviewer_decision. Approval sets the trusted role and creates the corresponding Driver or Company profile. Requested role is not itself authorization.

### “What happens if Reviewer approval fails part-way through?”

The curated Day 20 record says the production decision path uses a database RPC and forced-failure tests validated rollback for both Approve and Reject; the normal production retry succeeded. When discussing the current source, also acknowledge the migration 013 discrepancy, which needs source/database reconciliation.

---

## 12. Change-control rule for future maintenance

Before changing an API, table or authorization rule:

1. Read the corresponding locked architecture / accepted technical decision.
2. Inspect the handler and every client that calls it.
3. Inspect the migration and current deployed database state instead of assuming the migration file equals production.
4. Keep user identity, trusted role, resource relationship, Trip state and transition prerequisites server-side.
5. Add or update the appropriate automated test and record whether it was actually executed.
6. Update the implementation report, verification evidence and this crosswalk only after the change is confirmed.
7. Preserve superseded decisions as history and mark them superseded; do not present them as current capabilities.

---

## 13. Authoritative companion records

This crosswalk should be read alongside:

- [DeliveryProof_Detailed_Project_Record.md](./DeliveryProof_Detailed_Project_Record.md)
- [DeliveryProof_Approved_Final_Architecture.md](./DeliveryProof_Approved_Final_Architecture.md)
- [DeliveryProof_Important_Technical_Decisions.md](./DeliveryProof_Important_Technical_Decisions.md)
- [DeliveryProof_Major_Verified_Challenges_and_Fixes.md](./DeliveryProof_Major_Verified_Challenges_and_Fixes.md)
- [DeliveryProof_Final_Verification.md](./DeliveryProof_Final_Verification.md)
- [DeliveryProof_Final_Project_Handoff.md](./DeliveryProof_Final_Project_Handoff.md)
- [DeliveryProof_Source_Records_Reconciliation.md](./DeliveryProof_Source_Records_Reconciliation.md)
- [DeliveryProof_Documentation_Index.md](./DeliveryProof_Documentation_Index.md)

Those curated records preserve accepted architecture and recorded test results. This file adds source navigation and current-source observations; it does not override an accepted decision, and it does not convert source presence into test evidence.
