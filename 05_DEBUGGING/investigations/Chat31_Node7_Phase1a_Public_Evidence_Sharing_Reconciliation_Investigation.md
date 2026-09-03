# Chat31 — Node 7 Phase 1a — Public Evidence Sharing Reconciliation Investigation

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1a — Baseline AI + Shareable Evidence  
**Date:** 2026-09-03  
**Investigation status:** COMPLETED / SOURCE RECONCILED  
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

**Status: GAP CONFIRMED**

The `companies` schema contains basic profile data (name, email) but no verified logo field exists. 
**Implication:** Do not create a new profile/logo model solely for public sharing. Public representation must fallback to text-based company names based on existing schema.

### O7 — Vehicle fields

**Status: VERIFIED**

Current driver/trip schemas do not expose granular, public-safe vehicle identifiers. 
**Implication:** Phase 1a must not create a new vehicle-data model solely for public sharing; public representation will omit vehicle identifiers per the baseline exclusions.

### O8 — Required-evidence rules

**Status: VERIFIED**

The completion logic in `src/app/api/completion/receiver/route.ts` requires the `DELIVERY_DEPARTED` event. The AI summary route (`src/app/api/summary/route.ts`) requires Arrival + Check-in + Departure. 
**Implication:** The public checklist can safely mirror these specific established chronological milestones without exposing internal algorithms.

### O9 — Company-to-trip authorization for share creation

**Status: VERIFIED**

Company identity is verified via `trusted_role === 'COMPANY'` and checking the trip's `receiving_company_id` (or `company_id`) against the authenticated `company.id` (as seen in `trips/create` and `events/receiver-checkin`).
**Implication:** The exact same server-side authorization path will be used to guard the creation and revocation of share links.

### O10 — Rate limiting

**Status: GAP CONFIRMED**

A codebase-wide search revealed no explicit custom application-level rate limiting middleware (such as `@upstash/ratelimit`).
**Implication:** Phase 1a must introduce a basic rate-limiting strategy (e.g., in-memory or standard Next.js tools if applicable, or explicit integration) specifically for the public verification endpoint to prevent enumeration.

### O11 — Audit/logging

**Status: GAP CONFIRMED**

No dedicated internal audit logging mechanism was found in `src/lib/` or the API routes for generic lifecycle events.
**Implication:** A lightweight audit logging pattern specifically for share link creation, revocation, and replacement must be designed and implemented as part of Phase 1a.

### O12 — Routing conventions

**Status: VERIFIED**

The application uses standard Next.js App Router conventions. Authenticated APIs live in `src/app/api/`. 
**Implication:** A dedicated public API route (e.g., `src/app/api/public/share/...`) or page component must be used.

### O13 — Cache/deployment behavior

**Status: VERIFIED**

The app is deployed on Vercel (`next.config.ts` is standard).
**Implication:** Dynamic route segment config (`export const dynamic = 'force-dynamic'`) is mandatory for the public share route to prevent Vercel/Next.js from statically caching sensitive token responses.

### O14 — Storage/photo exposure

**Status: VERIFIED**

The `upload-photo` API generates secure, deterministic paths in the `event-photos` bucket and exposes them via `getPublicUrl`.
**Implication:** Since raw photo URLs are publicly resolvable if guessed/leaked, the Phase 1a share page UI must explicitly NOT request or render these URLs, adhering to the prohibition on raw photo sharing.

## 5. Final 65-decision reconciliation

The approved decisions are classified using the required confidence discipline:

- **VERIFIED:** existing AI summary capability; existing chronological event/evidence data; accepted authenticated API security baseline; accepted evidence immutability baseline; Phase 1a exclusions/requirements; vehicle fields (none to expose); company-to-trip authorization; required evidence checklist; routing conventions; cache behavior; photo storage boundaries.
- **GAP:** public share endpoint/route; public-share persistence/token model; company logo; rate-limiting implementation; audit logging.

All UNKNOWNs from the preliminary investigation have been source-reconciled.

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

## 8. Completed investigation actions

1. Inspected company profile/company lookup and trip-ownership authorization source paths.
2. Inspected trip schema and existing company/driver/vehicle fields.
3. Located the authoritative completion/evidence requirement implementation and mapped it to the public checklist.
4. Investigated current rate-limit helper/middleware and verified a GAP.
5. Investigated current audit/security logging implementation and verified a GAP.
6. Inspected current App Router page conventions for authenticated and public pages.
7. Inspected deployment/cache/security-header behavior relevant to public GET/read-only responses.
8. Inspected photo storage policy and URL generation to ensure raw evidence cannot become publicly reachable through the share page.
9. Completed the 65-decision matrix as VERIFIED / GAP / UNKNOWN with source evidence.

## 9. Current decision state

**DECISION: READY FOR IMPLEMENTATION.**

The investigation confirms that Phase 1a public sharing is a new capability built around existing verified AI/timeline/evidence primitives, with a dedicated public security boundary required. All implementation-critical dependencies have been source-reconciled.

No implementation is authorized by this file. An implementation instruction prompt can now be safely created.
