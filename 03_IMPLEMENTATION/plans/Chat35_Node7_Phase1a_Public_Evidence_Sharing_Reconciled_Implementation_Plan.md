# Antigravity Execution Prompt — Chat35 Node 7 Phase 1a

Execute the **approved Chat35 reconciled Phase 1a implementation plan** for the Freight — AI Builders Hackathon project.

## AUTHORITATIVE EXECUTION SCOPE

Primary execution plan:

`03_IMPLEMENTATION/plans/Chat35_Node7_Phase1a_Public_Evidence_Sharing_Reconciled_Implementation_Plan.md`

Supporting authoritative sources:

* `03_IMPLEMENTATION/prompts/Chat34_Node7_Phase1a_Public_Evidence_Sharing_Antigravity_Implementation_Prompt.md`
* `02_ARCHITECTURE/Chat34_Node7_Phase1a_Public_Evidence_Sharing_Architecture_Finalization.md`
* `00_PROJECT_CONTROL/CHECKPOINTS/Chat30_Node7_Phase1a_Public_Evidence_Sharing_Decision_Checkpoint.md`
* `05_DEBUGGING/investigations/Chat32_Node7_Phase1a_Remaining_Source_Reconciliation_Investigation.md`

**Chat35 is the reconciled execution plan. Follow it together with the locked decisions and architecture in the supporting sources above.**

---

## CRITICAL FILE-PROTECTION RULE

**DO NOT MODIFY, REWRITE, DELETE, RENAME, OR OVERWRITE the existing Chat34 implementation plan.**

The following file must remain unchanged:

`03_IMPLEMENTATION/plans/Chat34_Node7_Phase1a_Public_Evidence_Sharing_Implementation_Plan.md`

Chat35 is a **separate plan**, not a replacement or revision of Chat34.

If you discover that Chat35 and an existing source appear inconsistent, **STOP and report the discrepancy** rather than modifying Chat34 or silently inventing a resolution.

---

## EXECUTION MODE

Implement **Node 7 — Phase 1a only**.

Do not begin or implement:

* Phase 1b UI/UX redesign
* Phase 3 conditional add-ons
* unrelated refactoring
* new product features outside Phase 1a
* reopening or changing Nodes 1–6

The execution target is strictly:

**AI + Shareable Public Evidence — Phase 1a**

---

# 1. INSPECT CURRENT SOURCE FIRST

Before making any changes:

1. Inspect the current repository structure.
2. Inspect the existing trip, driver, company, evidence, completion, timeline, AI-summary, authentication/authorization, audit, rate-limiting, and database implementations.
3. Identify existing source-of-truth tables, types, helpers, routes, and conventions.
4. Inspect existing migrations and indexes.
5. Inspect existing tests and test conventions.
6. Inspect existing deployment/environment configuration relevant to public URLs.
7. Inspect existing Node 6 security/rate-limiting conventions.

Do not assume the planned file structure already matches the repository.

If implementation reality conflicts with the locked Phase 1a architecture or Chat35 plan:

**STOP. Report the exact discrepancy and do not silently invent a replacement architecture.**

---

# 2. IMPLEMENT ONLY THE LOCKED PHASE 1a DESIGN

Implement the following Phase 1a capabilities exactly according to Chat35 and the locked architecture:

### Public share lifecycle

* Company-controlled share creation.
* Company-controlled share replacement.
* Company-controlled share revocation.
* Only authorized company users may manage shares.
* Enforce company → trip authorization.
* Share creation requires the trip to satisfy the locked completion and required-evidence rules.
* Maximum one ACTIVE share per trip.
* Replacement must invalidate the previous share.
* Revocation must permanently invalidate the share.

### Database

Implement the `trip_public_shares` model according to the locked architecture.

Required conceptual fields:

* `id`
* `trip_id`
* `token_hash`
* `status`
* `created_by`
* `created_at`
* `revoked_at`

Required constraints:

* trip foreign key with cascade behavior as specified by the architecture
* `token_hash` NOT NULL
* unique token hash
* status exactly `ACTIVE` / `REVOKED`
* `created_by` foreign key with the locked SET NULL behavior
* database-generated timestamps
* partial unique index enforcing only one ACTIVE share per trip
* do not create an ordinary unconditional UNIQUE constraint on `trip_id`
* do not add lifecycle fields to unrelated trip/event tables

Use an appropriate migration consistent with the existing migration conventions.

---

# 3. TOKEN SECURITY

Generate share tokens using cryptographically secure randomness.

Required properties:

* 32 cryptographically secure random bytes.
* Base64URL representation for the public bearer token.
* SHA-256 hash for persistence/lookup.
* Persist **only the hash**.
* Never persist the raw bearer token.
* Never log the raw bearer token.
* Never place the raw token in audit records.
* A replacement must generate a completely new token.
* Old tokens must immediately become invalid after replacement.
* Revoked tokens must remain invalid permanently.

Construct the public URL server-side using the configured deployment host.

Do not hard-code a development-only host.

---

# 4. SHARE MANAGEMENT API

Implement the locked share-management endpoint:

`/api/trips/[tripId]/public-share`

Support the required lifecycle operations from Chat35/Chat34 architecture.

Enforce:

* authenticated company role
* company → trip authorization
* completed-trip requirement
* required-evidence requirement

Replacement must be concurrency-safe.

The transaction must:

1. Lock the relevant trip/share state as required.
2. Re-check authorization and eligibility.
3. Revoke the existing ACTIVE share.
4. Generate a fresh secure token.
5. Hash the token.
6. Insert the replacement share.
7. Depend on the partial unique index as the final database invariant.
8. Handle a defensive unique-violation/concurrency failure safely.

Never allow concurrent requests to leave two ACTIVE shares for the same trip.

Return the raw token only through the intended server-generated public URL response.

---

# 5. PUBLIC VERIFICATION API

Implement:

`GET /api/public/verify/[token]`

This endpoint is anonymous.

It must:

1. Apply anonymous rate limiting before expensive verification/database/AI work.
2. Hash the supplied token with SHA-256.
3. Look up the share by hash.
4. Require `ACTIVE` status.
5. Resolve the permitted trip/evidence data.
6. Produce only the explicitly approved public-safe projection.
7. Produce the evidence/timeline state.
8. Produce the evidence-grounded AI summary according to the locked freshness rules.

Invalid, malformed, nonexistent, and revoked tokens must use the same generic unavailable behavior.

Do not disclose whether a token existed previously.

Expected behavior:

* Valid share → HTTP 200.
* Invalid/revoked/malformed/nonexistent share → generic HTTP 404 unavailable.
* Unexpected server failure → generic safe HTTP 500 response.

Never expose:

* SQL errors
* stack traces
* provider errors
* model errors
* internal IDs
* authorization details
* security implementation details

---

# 6. PUBLIC DATA ALLOWLIST

The public response must contain only data explicitly permitted by the locked architecture.

Allowed categories:

### Company

* company name
* safe existing logo if already supported
* safe existing public reference if permitted

### Trip / Delivery

* delivery date derived from the approved completion/departure timestamp
* completed status
* pickup city/area if safe
* destination city/area if safe

### Evidence

* COMPLETE / INCOMPLETE state
* human-readable required-evidence checklist

Required evidence is based on the locked evidence model, including the approved Arrival, Check-in, and Departure evidence logic.

### AI Summary

* evidence-grounded summary
* approved generic unavailable state when AI cannot be produced

### Timeline

Chronological:

* Arrival
* Check-in
* Departure

Include only approved timestamp and safe city/area information.

Include a read-only/public indicator as appropriate.

---

# 7. STRICT PRIVACY BOUNDARY

Do NOT expose:

* driver identity
* raw photos
* photo URLs
* exact GPS coordinates
* GPS accuracy
* street addresses
* full addresses
* internal database IDs
* private customer/company information
* vehicle identifiers
* vehicle registration
* AI provider details
* AI model details
* audit/security details

Do not introduce:

* reverse geocoding
* route maps
* a new vehicle data model
* a second evidence source of truth

---

# 8. AI SUMMARY + FRESHNESS

The AI summary must be grounded in the current authoritative evidence and timeline.

The implementation must ensure:

* summary corresponds to the current evidence state
* stale AI output is never served as current
* if caching is used, cache validity is bound to the appropriate evidence fingerprint/version
* evidence changes invalidate the applicability of previous AI output
* AI failure does not break public verification

When AI is unavailable:

* return HTTP 200 for a valid share
* return deterministic evidence/timeline information
* show a generic AI-unavailable state
* do not expose provider/model/error details

Do not introduce an AI provider/model-selection feature.

---

# 9. ANONYMOUS RATE LIMITING

Implement the anonymous public verification rate limiting specified by the finalized Phase 1a architecture.

Required design:

* requester IP + supplied opaque share token as the composite protection strategy
* IP-level protection must also cover malformed/invalid token requests
* rate limiting must occur before expensive verification/AI processing
* follow the established Node 6 security/rate-limiting conventions where applicable
* do not invent an unrelated rate-limiting policy
* do not persist raw bearer tokens as part of rate limiting

Important:

Node 6's authenticated surfaces used the platform-native authenticated rate-limiting model. The public anonymous endpoint is different and therefore must implement the anonymous application-level protection explicitly required by the Phase 1a architecture.

If the current source does not contain enough information to implement a required threshold or mechanism without inventing policy:

**STOP and report the exact unknown rather than silently choosing one.**

---

# 10. AUDIT

Implement the required minimal audit behavior for:

* share created
* share accessed
* share revoked
* share replaced

Follow the existing Node 6 audit conventions.

Never record raw bearer tokens.

Do not expose audit/security information through the public API.

---

# 11. PUBLIC SHARE PAGE

Implement the dedicated read-only page:

`/share/[token]`

Requirements:

* no login required
* read-only
* consumes the dedicated public verification API
* displays only the approved public-safe projection
* displays evidence/timeline information
* handles AI-unavailable state gracefully
* generic unavailable state for invalid/revoked/malformed shares
* no photos
* no GPS
* no street/full addresses
* no public download
* no PDF/export
* no management controls
* no direct database access

The page must not expose private portal functionality.

---

# 12. CACHE / INDEXING / DEPLOYMENT

Public share pages and verification responses represent **live/current permitted data**, not immutable snapshots.

Ensure:

* dynamic behavior where required by the framework
* no inappropriate caching of bearer-token responses
* appropriate no-cache behavior
* `noindex`
* deployment-aware public URL construction
* compatibility with the repository's Next.js App Router/Vercel conventions

Do not create an immutable public snapshot system.

---

# 13. TESTING AND VERIFICATION

Run all required verification from Chat35.

At minimum verify:

### Database

* migration succeeds
* constraints work
* ACTIVE/REVOKED lifecycle works
* partial unique ACTIVE-per-trip invariant works

### Authorization

* authorized company user succeeds
* unauthorized company/user fails
* cross-company trip access fails
* incomplete/ineligible trips cannot create public shares

### Token security

* secure token generation
* only SHA-256 hash persisted
* raw token never persisted/logged/audited
* replacement invalidates old token
* revocation invalidates token

### Concurrency

* concurrent replacement/creation cannot leave multiple ACTIVE shares

### Public API

* valid token
* invalid token
* malformed token
* revoked token
* generic 404 behavior
* safe 500 behavior
* anonymous access

### Privacy

Verify that prohibited fields are absent from the public API and UI.

### Evidence / AI

* correct evidence completeness
* chronological timeline
* AI summary grounded in current evidence
* stale AI result is not served
* AI failure produces deterministic fallback

### Rate limiting

* valid-token abuse protection
* invalid-token abuse protection
* malformed-token IP protection
* rate limiting occurs before expensive work

### Cache / indexing

* dynamic response behavior
* no inappropriate caching
* noindex behavior

Run the repository's appropriate lint/typecheck/test/build commands.

Do not claim a test is VERIFIED unless it was actually executed and evidence exists.

---

# 14. IMPLEMENTATION REPORT

Create a new implementation report under:

`03_IMPLEMENTATION/implementation_reports/`

The report must document:

* exact files created
* exact files modified
* exact files deleted, if any
* migration name and database constraints/indexes
* API routes
* authorization behavior
* token lifecycle
* concurrency handling
* evidence source of truth
* AI freshness/cache behavior
* AI failure behavior
* anonymous rate limiting
* audit behavior
* public privacy allowlist
* UI behavior
* dynamic/no-cache/noindex behavior
* tests executed
* build/typecheck/lint results
* manual verification checklist
* unresolved UNKNOWN items
* any implementation-vs-architecture discrepancies
* final Git commit SHA

Use status terminology accurately:

* `VERIFIED` only when directly verified by execution/evidence.
* `INFERRED` when derived from source but not directly tested.
* `UNKNOWN` when not established.

Do not manufacture verification evidence.

---

# 15. SCOPE GUARDRAIL

Do NOT implement:

* Phase 1b
* Phase 3
* public photos
* public GPS/maps
* PDF/export/download
* expiration
* configurable sharing policies
* configurable AI providers/models
* numeric AI scores
* immutable snapshots
* new vehicle model
* analytics
* unrelated refactoring
* unrelated security changes

Do not reopen Nodes 1–6.

Do not change locked architecture decisions.

---

# 16. FINAL STOP CONDITION

After implementation:

1. Run the required verification.
2. Create the implementation report.
3. Commit the Phase 1a implementation.
4. Push the changes.
5. Report the commit SHA and verification results.
6. Clearly list any UNKNOWN or unresolved items.
7. Clearly identify what requires manual verification by Ayush.

**STOP HERE.**

Do NOT start Phase 1b.

Do NOT continue to any additional feature work.

Wait for **Ayush manual verification and approval** before any next phase.

## FINAL EXECUTION PRINCIPLE

**Inspect first → implement Chat35 exactly → verify → document → commit/push → stop for Ayush verification.**

If the actual repository conflicts with the locked Chat35/Chat34 architecture, **STOP and report the conflict instead of inventing a solution.**
