# Chat34 — Node 7 Phase 1a Public Evidence Sharing — Antigravity Implementation Prompt

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1a — Baseline AI + Shareable Evidence  
**Date:** 2026-09-03 — Day 14 / Chat34  
**Implementation authorization:** APPROVED BY AYUSH IN CHAT34  
**Architecture source:** `02_ARCHITECTURE/Chat34_Node7_Phase1a_Public_Evidence_Sharing_Architecture_Finalization.md`  
**Supporting decisions:** `00_PROJECT_CONTROL/CHECKPOINTS/Chat30_Node7_Phase1a_Public_Evidence_Sharing_Decision_Checkpoint.md`  
**Source reconciliation:** `05_DEBUGGING/investigations/Chat31_Node7_Phase1a_Public_Evidence_Sharing_Reconciliation_Investigation.md`, `05_DEBUGGING/investigations/Chat32_Node7_Phase1a_Remaining_Source_Reconciliation_Investigation_Report.md`  

---

## 1. Execution Rule

Implement exactly the finalized Phase 1a architecture and the 65 locked Chat30 decisions. Do not reopen architecture decisions, completed Nodes 1–6, or Phase 1b/Phase 3 scope.

Before modifying code:

1. Inspect the current repository/source and existing conventions.
2. Confirm the relevant schema, auth, evidence/completion, AI, rate-limiting, audit, and deployment conventions from the current source.
3. If implementation reality contradicts the finalized architecture, STOP and report the discrepancy before making an architectural change.
4. Do not silently invent replacements for missing source conventions.

This prompt is an execution handoff. It is not permission to expand scope.

---

## 2. Required Phase 1a Outcome

Implement public evidence sharing for completed trips with:

- company-controlled share creation, replacement, and revocation;
- dedicated `trip_public_shares` persistence;
- opaque 256-bit random bearer tokens;
- SHA-256 token-hash persistence only;
- one ACTIVE share maximum per trip;
- atomic concurrent-safe replacement;
- dedicated anonymous public verification API;
- dedicated public read-only page consuming that API;
- explicit public-safe data projection;
- evidence completeness and verification timeline;
- evidence-grounded AI summary with evidence-state freshness protection;
- graceful AI failure;
- anonymous rate limiting aligned with Node 6;
- minimal private audit events;
- dynamic/no-cache/noindex public behavior;
- no exposure of private or prohibited evidence fields.

---

## 3. Database / Persistence Requirements

Create the dedicated `trip_public_shares` lifecycle store.

Conceptual fields:

```text
id
trip_id
 token_hash
status
created_by
created_at
revoked_at
```

Required constraints/behavior:

- `trip_id` references the existing trip with `ON DELETE CASCADE`.
- `token_hash` is NOT NULL and UNIQUE.
- `status` is PostgreSQL ENUM with exactly `ACTIVE` and `REVOKED`.
- `created_by` references the existing authenticated-user identity and uses `ON DELETE SET NULL`.
- `created_at` is database-generated.
- `revoked_at` is NULL while ACTIVE and database-generated when REVOKED.
- Enforce one ACTIVE share per trip with a PostgreSQL partial unique index on `trip_id WHERE status = 'ACTIVE'`.
- Do not use a normal `UNIQUE(trip_id)` constraint because historical REVOKED records must remain possible.
- Do not put share lifecycle fields directly on `trips` or events.

Use the repository's established migration mechanism. The partial unique index must be represented in a form that PostgreSQL actually enforces; do not assume an ORM ordinary unique constraint can express the conditional index.

---

## 4. Share Lifecycle / Concurrency

Authenticated company management routes:

```text
POST   /api/trips/[tripId]/public-share
DELETE /api/trips/[tripId]/public-share
```

Creation/replacement must verify:

```text
authenticated user
→ COMPANY role
→ existing company → trip authorization
→ trip completed
→ required evidence present
→ transaction
```

Replacement requirements:

- Use one database transaction.
- Lock the relevant trip row to serialize concurrent share operations for the same trip.
- Re-check eligibility/current share state after acquiring the lock.
- Revoke any existing ACTIVE share.
- Generate a fresh token and create the replacement ACTIVE share in the same transaction.
- Never commit an old-revoked/new-missing partial state.
- Keep the partial unique index as the final database invariant.
- Treat unique-violation handling as a defensive safety path.

Token generation:

```text
32 cryptographically secure random bytes
→ 256-bit entropy
→ Base64URL opaque token
→ SHA-256 hash
→ persist hash only
```

The raw token must never be persisted or written to logs/audit records. Return it only as part of the complete server-constructed `publicUrl`; do not return a separate token field.

New share creation automatically invalidates the old share permanently.

Revocation permanently invalidates the token and cannot reactivate it.

The server constructs the public URL from the deployment's configured public host; the client must not construct the security credential.

---

## 5. Public Verification API

Implement the dedicated anonymous route:

```text
GET /api/public/verify/[token]
```

Keep it separate from all existing authenticated APIs.

Verification behavior:

```text
incoming opaque token
→ apply anonymous rate limiting
→ SHA-256 hash
→ lookup token_hash
→ require ACTIVE
→ resolve current permitted trip/evidence state
→ build explicit public projection
→ produce current-valid AI summary or generic unavailable state
→ return public response
```

Invalid, revoked, malformed, nonexistent, or otherwise unusable tokens must all produce the same generic public behavior:

```text
HTTP 404
generic unavailable response
```

Do not reveal why a token failed.

Valid request:

```text
HTTP 200
structured public projection
```

Unexpected internal failure:

```text
HTTP 500
generic public-safe response
```

Never expose SQL, stack traces, provider errors, model names, internal IDs, or security/audit implementation details.

---

## 6. Public Data Projection

Construct a dedicated server-side allowlist. Never return raw trip/event/database objects.

Public response sections:

```text
Company
Trip / Delivery
Evidence
AI Summary
Timeline
```

Allowed content:

- company name;
- safe existing company logo only if already available from an existing profile source, otherwise company name only;
- existing public-safe trip/reference if available, otherwise omit;
- delivery date derived from the completed/departure event timestamp;
- Completed status;
- pickup city/area when an existing safe value is available;
- destination city/area when an existing safe value is available;
- categorical evidence status COMPLETE/INCOMPLETE;
- human-readable required-evidence checklist based on existing completion/evidence rules;
- AI evidence-grounded summary;
- Arrival, Check-in, and Departure timeline events in chronological ascending order;
- event timestamp and safe city/area only when available;
- clear read-only/public verification indicator.

Do not create a second evidence/completion source of truth.

Vehicle information is omitted by default. Do not create a new vehicle model solely for Phase 1a.

Do not expose:

```text
Driver identity
Raw photos
Photo URLs
Exact GPS
GPS accuracy
Street/full addresses
Internal IDs
Internal company/customer details
Internal vehicle identifiers/registration details
AI provider/model details
Security/audit implementation details
```

Do not reverse-geocode or introduce a route map solely for this feature.

The public API/page must never request, render, or return existing photo URLs.

---

## 7. Evidence / Live State / AI

Use existing required evidence/completion rules.

Public evidence state:

```text
COMPLETE
INCOMPLETE
```

If evidence later becomes incomplete, an ACTIVE share remains active and the public API returns the current INCOMPLETE state rather than automatically revoking the share.

The public representation is live/current permitted data, not an immutable snapshot.

AI rules:

- Evidence/timeline are authoritative.
- AI is an enhancement.
- AI output must correspond to the current permitted public evidence state.
- If implementing caching, bind the cached AI result to the current evidence state/fingerprint/version so it cannot be reused for a different evidence state.
- When evidence state changes, stale AI output must not be served as current; regenerate as required by the existing AI architecture.
- If AI is unavailable, return HTTP 200 with deterministic evidence/timeline intact and a generic AI-unavailable state.
- Do not expose provider/model/API/stack error details.
- Include the approved concise public disclaimer that recorded evidence is primary and AI is generated from available evidence.

Do not create an immutable public snapshot or numerical evidence score.

---

## 8. Anonymous Rate Limiting

Apply application-level rate limiting to the public verification endpoint.

Use the finalized architecture's composite identity:

```text
requester IP + supplied opaque share token
```

Also provide IP-level protection for malformed/invalid-token requests.

Apply rate limiting before expensive verification/AI work.

Use thresholds and implementation conventions established by Node 6. Do not invent an unrelated threshold policy.

Raw bearer tokens must not be persisted in rate-limit storage, logs, or audit records.

---

## 9. Audit / Security

Add minimal private audit events for:

```text
share created
share accessed
share revoked
share replaced
```

Follow the existing Node 6 audit/security conventions discovered during implementation. Do not invent a conflicting audit schema if an established convention exists.

Never store raw bearer tokens in audit records.

Existing authenticated APIs must remain protected. Do not convert existing authenticated trip APIs into mixed public/authenticated endpoints.

---

## 10. Public UI

Implement a dedicated public verification page consuming only the dedicated public verification API.

Requirements:

- no Freight login required;
- obvious public/read-only verification state;
- display only the approved public contract;
- generic unavailable page/state for invalid/revoked/unusable tokens;
- current evidence completeness and checklist;
- timeline remains visible when AI is unavailable;
- no photos;
- no exact GPS or street addresses;
- no public downloads/export/PDF;
- no public share-management controls;
- no direct database access from the page;
- appropriate noindex/crawler controls;
- appropriate no-cache/dynamic behavior.

Company portal behavior:

```text
Company → create share
Company → copy URL
Company → revoke active share
```

The UI may hide Share before completion, but backend eligibility is authoritative.

Driver/reviewer role behavior remains unchanged except for viewing the share status/link according to the locked Chat30 decisions.

---

## 11. Dynamic / Cache / Deployment Controls

Fit the existing Next.js App Router and Vercel deployment conventions.

The public verification route must be dynamically resolved and must not serve stale state after revocation or evidence changes. Use the repository's established dynamic route mechanism; the architecture specifically requires `force-dynamic` for the public route where applicable.

Apply explicit no-cache/private-cache controls appropriate to the public verification response.

Use `noindex`/crawler controls on the public page. Do not treat noindex as access control.

Use the existing deployment-aware public host configuration for URL construction. Do not hard-code a production host if the repository already has a deployment configuration convention.

---

## 12. Testing / Evidence Required Before Handoff

Implement tests appropriate to the existing project test conventions, including at minimum:

### Database/lifecycle

- schema/migration applies successfully;
- partial unique index prevents two ACTIVE shares for one trip;
- multiple REVOKED records remain possible;
- `created_by` becomes NULL rather than deleting share history when the user is deleted;
- trip deletion follows the approved cascade behavior.

### Authorization

- non-COMPANY cannot create/revoke;
- unauthorized company cannot manage another company's trip;
- incomplete/unqualified trip cannot create a share;
- existing authenticated APIs remain protected.

### Token/security

- token has 256-bit random generation source;
- only SHA-256 hash is persisted;
- raw token is not logged/audited;
- replacement permanently invalidates the old token;
- revoked/invalid/malformed tokens are indistinguishable through the public response.

### Concurrency

- concurrent replacement requests for the same trip cannot leave two ACTIVE shares;
- transaction rollback cannot leave the old share revoked without a valid replacement where the operation is expected to succeed;
- post-lock eligibility/state re-check is effective.

### Public privacy boundary

Verify that the public response contains no:

```text
photos/photo URLs
exact GPS
GPS accuracy
street addresses
internal IDs
private company/customer details
driver identity
internal vehicle identifiers
AI provider/model details
```

### AI/evidence

- current evidence state is reflected;
- stale AI cache cannot cross evidence-state boundaries;
- AI failure leaves evidence/timeline available;
- no provider/model/internal error leaks.

### Rate limiting

- valid public verification is rate limited according to Node 6-aligned policy;
- malformed/invalid requests receive IP-level protection;
- rate-limit behavior does not expose token validity unnecessarily.

### Cache/indexing

- revoked share is not served from stale cache;
- evidence changes are visible through the live public representation;
- public page is noindex.

### Manual verification

After automated/build checks pass, provide a clear manual verification checklist for Ayush covering at least:

1. Company creates a share for an eligible completed trip.
2. Returned public URL opens without Freight login.
3. Public page shows only approved data.
4. Old URL becomes unavailable after replacement.
5. Revoke makes the current URL unavailable.
6. Invalid/malformed/revoked URLs all show the same unavailable state.
7. Evidence timeline remains visible if AI is unavailable.
8. No photos/exact GPS/private identifiers are visible.
9. Public state reflects current evidence.

Do not claim VERIFIED until actual test/manual evidence exists.

---

## 13. Scope Guardrails

Do NOT implement:

- Phase 1b full three-portal UI/UX redesign;
- Phase 3 stretch features;
- public raw photos;
- public exact GPS;
- public street addresses;
- public route map;
- public PDF/export/download;
- automatic link expiration;
- configurable public-sharing policies;
- public AI provider/model details;
- numeric evidence scoring;
- immutable public snapshots;
- new vehicle model solely for public sharing;
- expanded public visitor analytics;
- unrelated refactors.

If a requested behavior conflicts with the locked Chat30 decisions or finalized Chat34 architecture, STOP and report the conflict rather than silently changing scope.

---

## 14. Required Implementation Report

When implementation is complete, create the implementation report in:

`03_IMPLEMENTATION/implementation_reports/`

The report must include:

- exact files changed;
- database migration/index details;
- API routes added/changed;
- UI routes/components added/changed;
- authorization behavior;
- token lifecycle/security behavior;
- concurrency strategy;
- AI freshness/cache strategy;
- rate-limiting implementation and Node 6 alignment;
- audit behavior;
- privacy/public projection evidence;
- automated test/build results;
- manual verification status;
- known limitations or UNKNOWN items;
- commit SHA(s), if available through the established workflow.

Use VERIFIED / INFERRED / UNKNOWN accurately. Do not claim successful manual verification without Ayush's actual verification evidence.

**Execution target:** Implement Phase 1a only, validate it, report evidence, and stop for Ayush manual verification. Do not proceed into Phase 1b until separately authorized.
