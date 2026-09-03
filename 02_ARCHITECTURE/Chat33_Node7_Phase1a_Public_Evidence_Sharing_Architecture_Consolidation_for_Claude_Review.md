# Chat33 — Node 7 Phase 1a Public Evidence Sharing Architecture Consolidation for Claude Review

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1a — Baseline AI + Shareable Evidence  
**Date:** 2026-09-03 — Day 14 / Chat33  
**Reasoning brain:** ChatGPT  
**Architecture status:** CONSOLIDATION DRAFT — PENDING CLAUDE REVIEW  
**Implementation status:** NOT AUTHORIZED  
**Reviewer:** Claude  
**Purpose:** Provide Claude with one complete architecture package to review against the existing Freight system, the approved Chat30 decision set, and the completed Chat31/Chat32 source-reconciliation findings.

---

## 1. Review Context and Source of Truth

This document is a technical architecture consolidation, not an implementation instruction.

Claude must review this architecture against the following authoritative project records:

1. `00_PROJECT_CONTROL/CHECKPOINTS/Chat30_Node7_Phase1a_Public_Evidence_Sharing_Decision_Checkpoint.md`
   - Contains the complete set of 65 already-approved public evidence-sharing decisions.
   - These decisions must be treated as locked product/security requirements.
   - Do not reopen or re-ask them.

2. `05_DEBUGGING/investigations/Chat31_Node7_Phase1a_Public_Evidence_Sharing_Reconciliation_Investigation.md`
   - Initial source-reconciliation investigation.

3. `05_DEBUGGING/investigations/Chat32_Node7_Phase1a_Remaining_Source_Reconciliation_Investigation_Report.md`
   - Final source reconciliation.
   - Concludes `READY FOR ARCHITECTURE`.

4. Existing Freight source code and Node 6 security baseline referenced by Chat31/Chat32.

### Governance

The governing workflow remains:

```text
OBSERVATION
    ↓
INVESTIGATION
    ↓
EVIDENCE
    ↓
ROOT CAUSE
    ↓
DECISION
    ↓
FIX
    ↓
BUILD / TEST
    ↓
AYUSH MANUAL VERIFICATION
```

This document is still before implementation. Claude's job is architecture review, not implementation.

---

## 2. What Claude's Role Is

Claude is the **independent architecture reviewer** for this checkpoint.

Claude is NOT the implementation agent and must NOT directly instruct Antigravity to modify the source code.

### Claude's review task

Claude must:

1. Read this architecture consolidation.
2. Read the Chat30 approved decision checkpoint.
3. Read the Chat31 and Chat32 investigation records.
4. Compare the proposed architecture against the existing Freight system as evidenced by those records and, where available, the current source.
5. Check that every architectural choice is compatible with the already-approved requirements.
6. Identify contradictions, missing dependencies, unsafe assumptions, unnecessary scope, race conditions, authorization gaps, privacy leaks, data-model problems, API boundary problems, deployment/cache issues, or inconsistencies with existing architecture.
7. Distinguish evidence-backed findings from inference.
8. Recommend corrections only where a genuine architectural issue exists.
9. Confirm which parts are sound and can remain locked.
10. Produce a review result that ChatGPT can use to finalize the architecture before any implementation prompt is created.

### Claude must NOT

- Re-ask the 65 Chat30 product/security decisions.
- Invent new requirements merely for completeness.
- Start implementation.
- Produce Antigravity coding instructions.
- Claim source behavior that is not supported by the Records/source evidence.
- Reopen completed Nodes 1–6 without concrete regression evidence.
- Expand Phase 1a into the Phase 1b UI/UX redesign or Phase 3 stretch work.

### Claude review output should classify findings as

```text
CONFIRMED ISSUE
RISK / CONCERN
MISSING ARCHITECTURAL DETAIL
CONSISTENT / NO CHANGE NEEDED
SOURCE EVIDENCE REQUIRED
```

For each issue, Claude should state:

```text
Finding
Evidence / basis
Impact
Recommended architectural correction
Whether ChatGPT must resolve it before implementation
```

---

# 3. Consolidated Phase 1a Architecture

## Item 1 — Overall Public Share Data Flow

The public-sharing architecture is a separate security boundary from existing authenticated Freight APIs.

```text
COMPANY PORTAL
    │
    │ POST /api/trips/[tripId]/public-share
    ▼
AUTHENTICATED SHARE MANAGEMENT API
    │
    ├─ Verify authenticated COMPANY role
    ├─ Verify company → trip authorization
    ├─ Verify trip completion
    ├─ Verify required evidence
    └─ Perform server-controlled share lifecycle operation
    │
    ▼
ATOMIC DATABASE OPERATION
    │
    ├─ Revoke existing ACTIVE share if present
    ├─ Generate fresh 256-bit random token
    ├─ Base64URL-encode token
    ├─ SHA-256 hash token
    └─ Create new ACTIVE share
    │
    ▼
Return complete publicUrl
    │
    ▼
PUBLIC VERIFICATION PAGE
    │
    │ GET /api/public/verify/[token]
    ▼
DEDICATED PUBLIC VERIFICATION API
    │
    ├─ Hash incoming token
    ├─ Resolve matching ACTIVE share
    ├─ Apply public endpoint rate limiting
    ├─ Resolve current permitted trip/evidence state
    ├─ Build explicit public projection
    ├─ Produce AI summary or generic AI-unavailable state
    └─ Return only approved public fields
    │
    ▼
PUBLIC READ-ONLY VERIFICATION VIEW
```

Core principle:

> The public share is a trip-level publication/access object, not an immutable copy of the trip.

The public representation is resolved from current permitted data on each request.

---

## Item 2 — `trip_public_shares` Database Model

Use a dedicated table because public sharing is a trip-level lifecycle object and no existing public-share persistence model was found during investigation.

Conceptual model:

```text
trip_public_shares
├── id
├── trip_id
├── token_hash
├── status
├── created_by
├── created_at
└── revoked_at
```

Locked architectural constraints:

```text
trip_id
  → foreign key to trip
  → ON DELETE CASCADE

token_hash
  → NOT NULL
  → UNIQUE

status
  → PostgreSQL ENUM
  → ACTIVE | REVOKED

created_by
  → existing authenticated-user identity relationship

created_at
  → database-generated

revoked_at
  → NULL while ACTIVE
  → database-generated when REVOKED
```

Database must enforce maximum one active share per trip using a partial uniqueness constraint equivalent to:

```text
UNIQUE (trip_id) WHERE status = 'ACTIVE'
```

Historical revoked records may remain.

No share fields are added directly to `trips`, and events are not used as the share lifecycle store.

---

## Item 3 — Share Creation, Replacement, and Revocation Lifecycle

### Creation

`POST /api/trips/[tripId]/public-share` is an authenticated company-management operation.

Server checks, in order conceptually:

```text
Authenticated request
    ↓
COMPANY role
    ↓
Company authorized for trip
    ↓
Trip completed
    ↓
Required evidence present
    ↓
Atomic share lifecycle transaction
```

### Replacement

If an ACTIVE share already exists, the same creation operation automatically revokes the previous share and creates a fresh share.

The operation must be atomic. A partial state such as “old share revoked but new share not created” must not be committed.

### Token generation

The server generates:

```text
32 cryptographically random bytes
        ↓
256-bit entropy
        ↓
Base64URL encoding
        ↓
opaque bearer token
        ↓
SHA-256
        ↓
token_hash stored only
```

The raw token is never persisted.

The response contains the raw token only inside the complete `publicUrl`; there is no separate `token` response field.

### Revocation

`DELETE /api/trips/[tripId]/public-share` is company-controlled and requires normal confirmation in the UI before the destructive action.

Revoked tokens are permanently invalid and cannot be reactivated.

### URL construction

The server constructs the complete public URL using deployment-aware server-side public-host configuration. The client does not construct the security credential.

---

## Item 4 — Public Verification API

### Route

```text
GET /api/public/verify/[token]
```

This route is anonymous/public and must remain separate from existing authenticated APIs.

### Verification sequence

Conceptually:

```text
Incoming opaque token
        ↓
Hash token with SHA-256
        ↓
Find token_hash
        ↓
Require ACTIVE status
        ↓
Resolve associated trip
        ↓
Build current permitted public projection
        ↓
Return response
```

### Invalid / revoked / unusable token behavior

All malformed, unknown, revoked, nonexistent, or otherwise unusable tokens have the same public behavior:

```text
HTTP 404
Generic unavailable response
```

No token-state distinction may be exposed.

### Successful response

```text
HTTP 200
Structured public projection
```

The response contains only:

```text
Company
Trip / Delivery
Evidence
AI Summary
Timeline
```

Raw database objects are never returned directly.

### Unexpected server failure

A genuine internal failure returns a generic public-safe `500` response. No database, stack trace, provider, SQL, or implementation details are exposed.

### Rate limiting

The public verification endpoint must have application-level rate limiting. The exact mechanism and threshold must align with the established Node 6 rate-limiting architecture rather than inventing an unrelated policy.

### No authentication requirement

A valid opaque share token is the public access credential. Freight login is not required.

---

## Item 5 — Public Data Projection

The public API uses an explicit server-side allowlist/projection.

### Public contract

```text
Company identity
→ Public-safe trip/reference
→ Delivery date
→ Completed status
→ Pickup city/area
→ Destination city/area
→ Evidence completeness
→ Required-evidence checklist
→ AI evidence-grounded summary
→ Timeline
→ Read-only indicator
```

### Company identity

Reuse the existing company name. If a safe existing logo is available from the existing company profile, it may be exposed; otherwise use company name only.

No new company/logo model is introduced solely for Phase 1a.

### Vehicle

Vehicle information is omitted by default because investigation did not establish a granular public-safe vehicle model. No new vehicle model is created solely for public sharing.

### Trip reference

Reuse an existing public-safe trip/reference field if available. Never expose internal database IDs as the public reference. If no safe existing reference exists, omit it rather than inventing a new identifier system solely for Phase 1a.

### Delivery date

Derive the public delivery date from the existing completed/departure event timestamp rather than creating a duplicate delivery-date source of truth.

### Location

Only an existing safe city/area representation may cross the public boundary.

```text
Safe city/area exists → expose
No safe city/area → omit
```

Do not expose exact GPS, GPS accuracy, street/full addresses, or a public route map. Do not introduce reverse-geocoding solely for Phase 1a.

### Photos

Photos are completely excluded from the public projection. The public route must not request, return, render, or otherwise surface photo URLs.

### Private data

Never expose:

```text
Driver identity
Raw photos
Exact GPS
GPS accuracy
Street/full addresses
Internal IDs
Internal company/customer details
Internal vehicle identifiers/registration details
AI provider/model details
Security/audit implementation details
```

---

## Item 6 — Evidence, Timeline, and AI Behavior

### Evidence completeness

Use existing required-evidence/completion rules as the source of truth. Do not create a second validation system.

Public state is categorical:

```text
COMPLETE
INCOMPLETE
```

Provide a human-readable checklist of required evidence categories. Do not expose internal validation algorithms and do not create a numerical score.

### Timeline

The public timeline is a dedicated projection containing only approved verification milestones:

```text
Arrival
Check-in
Departure
```

Each public event contains only:

```text
event_type
timestamp
city/area (when safely available)
```

Events are ordered chronologically ascending.

No exact GPS, GPS accuracy, photo evidence, street address, or private metadata crosses the public boundary.

### Live representation

The public page is a live read-only representation of current permitted data, not an immutable snapshot.

If evidence later becomes incomplete:

```text
Share remains ACTIVE
        ↓
Public response remains HTTP 200
        ↓
Evidence status becomes INCOMPLETE
        ↓
Current checklist is shown
```

The share is not automatically revoked merely because evidence later becomes incomplete.

### AI summary

AI and deterministic evidence/timeline remain separate sections.

Recorded evidence is primary; AI is an enhancement.

The public AI summary must correspond to the current permitted public evidence state.

If AI is unavailable:

```text
HTTP 200
Evidence → available
Timeline → available
AI Summary → generic UNAVAILABLE state
```

Do not expose provider, model, API, stack, or internal error details.

Include a concise public disclaimer that recorded evidence is primary and the AI summary is generated from available evidence.

---

## Item 7 — Public UI and Company Management Behavior

### Public verification page

The public page is a dedicated read-only view consuming the dedicated public verification API.

It must:

- require no Freight login;
- clearly identify itself as public/read-only verification;
- display only the approved public contract;
- show generic unavailable behavior for invalid/revoked/unusable tokens;
- show current evidence completeness;
- keep the timeline visible when AI is unavailable;
- not expose photos, exact GPS, street addresses, internal IDs, or security details;
- provide no public download/export/PDF function;
- provide no public share-management control;
- use `noindex`/crawler controls and appropriate cache controls.

The page must not access the database directly. It consumes the dedicated public API.

### Company portal

Company users can:

```text
Create share
Copy public URL
Revoke active share
```

The UI may hide Share before completion, but the backend remains the authoritative enforcement point.

Creating a new share replaces the old active share automatically.

### Driver / Reviewer

The approved role model remains:

```text
Company   → create / copy / revoke
Reviewer  → view status/link only
Driver    → view status only
Public    → read-only verification only
```

No existing authenticated API is converted into a mixed public/authenticated endpoint.

---

## Item 8 — Security, Audit, and Deployment Controls

### Token security

- 256-bit cryptographically random token.
- Base64URL representation.
- SHA-256 hash storage only.
- No raw token persistence.
- No raw token in audit records.
- One active token per trip.
- Replacement permanently invalidates the old token.

### Authorization

Share creation/revocation uses the established authenticated Company → trip authorization path. Public verification does not expose or bypass authenticated trip APIs.

### Anti-enumeration

Invalid, revoked, malformed, nonexistent, and otherwise unusable tokens must not be distinguishable through public response behavior.

### Rate limiting

Public verification receives application-level rate limiting aligned with Node 6 architecture.

### Audit

Minimal private audit events are required for:

```text
share created
share accessed
share revoked
share replaced
```

Raw bearer tokens are never written to audit records.

Exact audit fields must align with the existing Node 6 audit/security architecture and must not be invented independently if the source establishes a convention.

### Caching / dynamic behavior

The public verification route must resolve current state on each request and must not serve stale public state after revocation or evidence changes.

The investigation established that the deployed Next.js/Vercel environment requires dynamic route handling (`force-dynamic`) for the public route and appropriate no-cache controls.

### Search indexing

The public verification page must use `noindex`/crawler controls. This is a discovery-control measure, not an authorization mechanism.

### Photo storage boundary

The existing photo storage behavior was found to expose public URLs through the upload-photo flow. Therefore, the new public projection must never request, return, or render those photo URLs.

### Deployment

The architecture must fit the existing Next.js App Router and Vercel deployment conventions without changing unrelated infrastructure.

---

# 4. Explicit Phase 1a Scope Protection

The following are outside this architecture baseline unless separately approved later:

- Public raw-photo viewing
- Public exact GPS
- Public street addresses
- Public route map
- Public PDF/export/download
- Automatic link expiration
- Configurable public-sharing policies
- Public AI provider/model details
- Numerical evidence-completeness scoring
- Immutable public snapshots
- New vehicle-data model solely for public sharing
- Expanded public visitor analytics
- Phase 1b full three-portal UI/UX redesign
- Phase 3 conditional add-on features

---

# 5. Existing-System Alignment Already Established by Chat32

The final investigation reported the following status:

```text
Company profile/logo      → GAP CONFIRMED; use company name, safe existing logo only
Vehicle                   → VERIFIED; omit by default
Required evidence        → VERIFIED
Company → trip auth       → VERIFIED
Rate limiting             → GAP CONFIRMED; Phase 1a must introduce public protection aligned to Node 6
Audit/logging             → GAP CONFIRMED; lightweight lifecycle audit needed
App Router                → VERIFIED
Deployment/cache          → VERIFIED
Photo storage exposure    → VERIFIED; public route must not expose photos
Public share endpoint     → GAP / NEW WORK
Public share persistence  → GAP / NEW WORK
Token lifecycle           → GAP / NEW WORK
Public API boundary       → GAP / NEW WORK
Public page               → GAP / NEW WORK
```

The investigation concluded that all previously unknown/partial areas were reconciled sufficiently to enter architecture.

---

# 6. Claude Review Questions

Claude should review the architecture as a whole, not treat every paragraph as a new decision question.

Specifically verify:

### A. Consistency

- Does the architecture faithfully implement the 65 Chat30 decisions?
- Are any previously locked requirements accidentally weakened or contradicted?

### B. Existing-system compatibility

- Does the architecture fit the existing Freight authentication, company/trip authorization, evidence/completion, timeline, AI summary, Next.js App Router, Supabase/database, storage, and Vercel patterns evidenced in Chat31/Chat32?
- Are any assumptions unsupported by the investigation?

### C. Security

- Is the anonymous public boundary sufficiently isolated?
- Is token entropy/hash handling correct?
- Does the replacement lifecycle prevent races and partial states?
- Does the database enforce one ACTIVE share per trip?
- Are anti-enumeration, rate limiting, cache behavior, and no-index controls adequate?
- Can any private trip/event/photo information leak through the public projection?

### D. Data model

- Is `trip_public_shares` sufficient for Phase 1a?
- Are its constraints and lifecycle fields coherent?
- Are any fields unnecessary or missing?

### E. API contract

- Are the authenticated management routes and anonymous verification route cleanly separated?
- Are HTTP status behaviors coherent?
- Is the public response projection sufficiently explicit?

### F. Evidence / AI consistency

- Does the architecture correctly keep deterministic evidence primary?
- Can evidence become incomplete without creating an inconsistent public state?
- Does AI failure degrade safely?
- Does the public AI summary remain tied to the current public evidence state?

### G. Operational concerns

- Are audit events sufficient but minimal?
- Are deployment and caching controls sufficient for current-state revocation/evidence changes?
- Is the architecture appropriately scoped for a hackathon Phase 1a baseline?

### H. Implementation readiness

Identify anything that must be resolved **before** an Antigravity implementation prompt is written.

Do not provide implementation instructions; only identify architecture-level blockers or corrections.

---

# 7. Expected Claude Review Deliverable

Claude should return a concise but technically rigorous review with this structure:

```text
1. Overall verdict
   - READY
   - READY WITH CORRECTIONS
   - NOT READY

2. Architecture strengths

3. Confirmed issues
   For each:
   - Finding
   - Evidence/basis
   - Impact
   - Required correction

4. Risks / concerns

5. Missing architectural details

6. Chat30 compatibility check
   - Any contradiction? YES/NO

7. Chat32 compatibility check
   - Any unsupported assumption? YES/NO

8. Implementation readiness
   - Ready for implementation prompt? YES/NO

9. Final recommendation
```

If Claude finds no genuine issue, it should explicitly say so rather than inventing changes.

---

# 8. Current Decision State

```text
Chat30 65 approved decisions              → LOCKED
Chat31 investigation                       → COMPLETE
Chat32 source reconciliation               → COMPLETE / READY FOR ARCHITECTURE
Architecture consolidation                 → COMPLETE DRAFT
Claude independent architecture review     → NEXT
Implementation                             → NOT STARTED
Antigravity prompt                         → NOT CREATED
Ayush final architecture approval          → NOT YET
```

**Important:** This document does NOT authorize implementation. It exists specifically to give Claude one efficient, complete architecture artifact to review before the implementation stage.
