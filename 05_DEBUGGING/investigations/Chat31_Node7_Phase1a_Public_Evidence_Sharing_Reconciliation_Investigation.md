# Chat31 — Node 7 Phase 1a — Public Evidence Sharing Reconciliation Investigation

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1a — Baseline AI + Shareable Evidence  
**Date:** 2026-09-03  
**Investigation status:** IN PROGRESS / SOURCE RECONCILIATION  
**Purpose:** Reconcile the 65 approved public shareable read-only evidence-link decisions from `Chat30_Node7_Phase1a_Public_Evidence_Sharing_Decision_Checkpoint.md` against the current Freight source repository and the accepted Node 6 security/evidence baseline before any implementation instruction is created.

## 1. Governing workflow

Investigation and implementation remain separate.

Required pipeline:

OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE / GAP → DECISION → FIX → BUILD/TEST → AYUSH MANUAL VERIFICATION

This file is investigation/reconciliation only. It does not authorize source-code implementation and does not create an Antigravity implementation prompt.

## 2. Authoritative baselines inspected

### Records repository

- `00_PROJECT_CONTROL/PROJECT_STATE.md`
- `00_PROJECT_CONTROL/Chat29_Node7_Roadmap_Reassessment_Phasing.md`
- `00_PROJECT_CONTROL/CHECKPOINTS/Chat30_Node7_Phase1a_Public_Evidence_Sharing_Decision_Checkpoint.md`
- `05_DEBUGGING/investigations/Chat27_Node6_Security_Evidence_Investigation.md`
- Node 6 completion/security evidence records surfaced by the Records repository

### Current application source

- Repository: `ayush22cp008/freight_hackathon`
- Branch: `main`
- Current source tree/API structure inspected directly from GitHub.

## 3. Phase 1a decision baseline

The approved Chat30 checkpoint defines the intended public representation as:

Company identity → public trip reference → delivery date → Completed → pickup/destination city/area → evidence completeness → required evidence checklist → AI evidence-grounded summary → event timeline with event type + timestamp + city/area → read-only indicator.

It also explicitly excludes raw photos, precise GPS, street/full addresses, route map, public downloads/exports, driver identity, internal IDs, internal company/customer details, internal vehicle identifiers, AI provider/model details, numerical confidence/completeness scoring, immutable public snapshots, and expanded visitor analytics.

## 4. Source-level observations

### O1 — Public share endpoint/route

**Status: GAP / NOT FOUND IN CURRENT SOURCE SEARCH**

The current `src/app/api` tree contains authenticated application API areas including `admin`, `auth`, `companies`, `completion`, `events`, `onboarding`, `server-time`, `summary`, `trips`, and `upload-photo`. No public share/public verification route was found in the inspected API tree, and targeted source search for public share/share-token/evidence verification terms returned no matching implementation.

**Implication:** Public evidence sharing is a new Phase 1a capability, not an existing route to expose.

### O2 — Existing AI summary

**Status: VERIFIED**

`src/app/api/summary/route.ts` exists and requires an authenticated user. It resolves the authenticated driver server-side, scopes the selected trip to that driver, reads trip events, requires Arrival + Check-in + Departure evidence, builds an evidence payload from event data, and asks the AI to summarize only supplied structured evidence. The route is therefore an existing authenticated AI-summary capability that must not simply be exposed publicly without a separate public-safe boundary.

### O3 — Existing timeline/evidence data

**Status: VERIFIED / EXISTING BASELINE**

The current source contains the events API area and the existing summary route reads chronological events by `trip_id` ordered by `server_timestamp`. Node 5/Node 6 Records also establish the completed evidence lifecycle and immutable historical evidence as accepted baselines.

**Implication:** Phase 1a should compose existing evidence/timeline data rather than redesign the event lifecycle.

### O4 — Existing authenticated security boundary

**Status: VERIFIED**

The accepted Node 6 baseline states that application API routes use server-side privileged Supabase access paths and therefore require explicit application/API authorization rather than relying on RLS alone. Node 6 also established the existing authorization/evidence boundaries and should not be reopened absent contradictory evidence.

**Implication:** A public verification surface must be a deliberately separate boundary with its own token validation, eligibility, field allowlist, rate limiting, anti-enumeration behavior, and cache policy.

### O5 — Public share persistence model

**Status: GAP / NOT FOUND**

No existing public-share token model or migration was identified in the current source/Records search performed for this investigation.

**Implication:** Persistence design remains a Phase 1a implementation gap. It must use the approved opaque high-entropy token model and secure hash storage from Chat30 rather than exposing internal IDs or using the public reference as the credential.

### O6 — Company logo/profile fields

**Status: UNKNOWN**

The current source search did not establish a verified company logo field or a verified public-safe company-logo serving mechanism.

**Required follow-up:** Inspect the company/profile schema and UI/API implementation before deciding whether an existing logo can safely be reused. Do not create a new profile/logo model solely for public sharing unless a material existing-model gap is proven.

### O7 — Vehicle fields

**Status: UNKNOWN**

The current targeted source search did not establish which existing vehicle fields are available and safe for the public representation.

**Required follow-up:** Inspect the trip/driver/vehicle schema and current UI/API usage. Phase 1a must not create a new vehicle-data model solely for public sharing.

### O8 — Required-evidence rules

**Status: PARTIALLY VERIFIED / FINAL PUBLIC CHECKLIST UNKNOWN**

The existing AI summary route explicitly requires Arrival, Check-in, and Departure evidence before generating a summary. This proves an existing required-sequence rule used by the AI flow. However, the complete authoritative required-evidence rule/categories for the final completed-trip public representation have not yet been fully reconciled against the current source and Node 5/Node 6 records.

**Required follow-up:** Identify the authoritative completion/evidence requirements and map them to the public human-readable checklist without exposing internal algorithms.

### O9 — Company-to-trip authorization for share creation

**Status: UNKNOWN**

The Phase 1a decision requires that only the company can create/revoke a share link and that only completed trips are eligible. The current source-level investigation has not yet verified the exact company identity → trip ownership/relationship authorization path that should guard creation/revocation.

**Required follow-up:** Inspect company trip APIs and authenticated company portal/server authorization before implementation.

### O10 — Rate limiting

**Status: ARCHITECTURE VERIFIED; PUBLIC THRESHOLD UNKNOWN**

Node 6 Records establish that rate-limiting architecture is an accepted security concern and that later reconciliation distinguished Supabase-native Auth rate limiting from custom application-level protection. The exact current source implementation and the numeric threshold to apply to the new public verification endpoint have not yet been source-verified in this investigation.

**Required follow-up:** Reconcile the actual current rate-limit helper/middleware and choose the public verification threshold without silently importing an older proposal.

### O11 — Audit/logging

**Status: UNKNOWN FOR PHASE 1a SHARE LIFECYCLE**

The Chat30 decision requires internal audit events for share create/access/revoke/replacement, but the exact existing audit/logging mechanism and minimal fields have not yet been reconciled against current source.

**Required follow-up:** Inspect existing audit/security logging patterns before designing any new audit surface.

### O12 — Routing conventions

**Status: PARTIALLY VERIFIED**

The application uses Next.js App Router and `src/app/api/...` route handlers. The authenticated API structure is verified. The exact public page route convention for Phase 1a has not yet been selected from the current application routing structure.

### O13 — Cache/deployment behavior

**Status: UNKNOWN**

The Chat30 decision requires appropriate no-cache/private caching controls and deployment-safe behavior for public verification. Current source/deployment configuration has not yet been fully reconciled for this new route.

**Required follow-up:** Inspect `next.config.ts`, existing response/cache behavior, Vercel/deployment configuration records, and any existing security-header conventions before implementation.

### O14 — Storage/photo exposure

**Status: UNKNOWN FOR PUBLIC REPRESENTATION**

The current API tree includes `upload-photo`, confirming photo storage/upload functionality exists. The Phase 1a decision explicitly prohibits public raw photos. The current investigation has not yet verified whether existing photo URLs are public, signed, private, or otherwise safely inaccessible to an unauthenticated public visitor.

**Required follow-up:** Inspect photo upload/storage implementation and Supabase Storage policy/URL generation. Public sharing must not accidentally expose raw evidence photos.

## 5. Preliminary 65-decision reconciliation

The approved decisions are classified using the required confidence discipline:

- **VERIFIED:** existing AI summary capability; existing chronological event/evidence data; accepted authenticated API security baseline; accepted evidence immutability baseline; Phase 1a exclusions/requirements as recorded in Chat30.
- **GAP:** public share endpoint/route; public-share persistence/token model.
- **UNKNOWN:** company logo/profile serving; vehicle-safe fields; exact complete public evidence checklist; company-to-trip authorization path; exact public rate-limit implementation/threshold; audit/logging implementation; public page routing convention; cache/deployment behavior; raw-photo/storage URL exposure.
- **PARTIALLY VERIFIED:** required evidence because the AI summary route verifies Arrival + Check-in + Departure but the complete authoritative completion/public checklist still requires reconciliation.

No UNKNOWN should be converted to VERIFIED by assumption.

## 6. Security boundary decision carried forward from Chat30 + Node 6

The intended architecture remains:

- Existing authenticated Freight APIs stay protected.
- Public verification uses a dedicated public route/API boundary.
- Access is authorized by an opaque high-entropy bearer token, not by internal trip IDs.
- Only one completed trip is in scope for each token.
- Raw token is not stored; only a secure cryptographic hash is persisted.
- Revocation is permanent.
- Replacement creates a fresh independent token and revokes the prior active token.
- Invalid, revoked, nonexistent, and unusable tokens return the same generic unavailable response.
- Public responses use an explicit allowlist of approved public fields.
- Public endpoint is rate-limited and anti-enumeration protected.
- Public response/cache behavior prevents unintended sensitive caching.
- Public access must not expose raw photos or precise GPS.

## 7. Scope boundaries

This investigation does not authorize:

- source-code implementation;
- database migration creation in the application repository;
- Antigravity prompt creation;
- reopening Nodes 1–6;
- redesigning authentication or RLS;
- adding raw-photo public access;
- adding a route map;
- adding PDF/export/download;
- adding numerical confidence/completeness scoring;
- adding an immutable public snapshot model;
- creating a new vehicle model solely for sharing;
- adding visitor analytics beyond the approved minimum internal audit events.

## 8. Next investigation actions before implementation decision

1. Inspect company profile/company lookup and trip-ownership authorization source paths.
2. Inspect trip schema and existing company/driver/vehicle fields.
3. Locate the authoritative completion/evidence requirement implementation and map it to the public checklist.
4. Locate current rate-limit helper/middleware and verify the protected surfaces.
5. Locate current audit/security logging implementation and identify the minimum lifecycle fields for share create/access/revoke/replacement.
6. Inspect current App Router page conventions for authenticated and public pages.
7. Inspect deployment/cache/security-header behavior relevant to public GET/read-only responses.
8. Inspect photo storage policy and URL generation to ensure raw evidence cannot become publicly reachable through the share page.
9. Complete the 65-decision matrix as VERIFIED / GAP / UNKNOWN with source evidence.
10. Produce the formal investigation result and only then create an implementation decision/prompt if the evidence supports one.

## 9. Current decision state

**DECISION: INVESTIGATION CONTINUES.**

The investigation confirms that Phase 1a public sharing is a new capability built around existing verified AI/timeline/evidence primitives, with a dedicated public security boundary required. Several implementation-critical dependencies remain UNKNOWN and must be source-reconciled before an implementation prompt is created.

No implementation is authorized by this file.
