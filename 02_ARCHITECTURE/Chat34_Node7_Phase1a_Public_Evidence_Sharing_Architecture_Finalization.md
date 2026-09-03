# Chat34 — Node 7 Phase 1a Public Evidence Sharing Architecture Finalization

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1a — Baseline AI + Shareable Evidence  
**Date:** 2026-09-03 — Day 14 / Chat34  
**Reasoning brain:** ChatGPT  
**Architecture status:** READY FOR IMPLEMENTATION  
**Implementation status:** NOT YET AUTHORIZED  
**Source architecture:** `02_ARCHITECTURE/Chat33_Node7_Phase1a_Public_Evidence_Sharing_Architecture_Consolidation_for_Claude_Review.md`  
**Independent review:** Claude — READY WITH CORRECTIONS

---

## 1. Finalization Purpose

This record finalizes the Phase 1a public evidence-sharing architecture after resolving every implementation-blocking correction identified by Claude's architecture review.

The 65 approved Chat30 decisions remain locked. Chat31/Chat32 source-reconciliation findings remain authoritative. No completed Node 1–6 work is reopened.

This record is an architecture decision/finalization record, not an implementation prompt.

---

## 2. Claude Review Corrections — Resolved and Locked

### Correction 1 — One ACTIVE share per trip

**Decision:** Use a PostgreSQL partial unique index on `trip_public_shares.trip_id` for rows whose status is `ACTIVE`.

Requirements:

- Multiple historical `REVOKED` records are permitted.
- At most one `ACTIVE` record may exist for a trip.
- The constraint must be implemented as a PostgreSQL partial unique index rather than an ordinary table-level `UNIQUE(trip_id)` constraint.
- The database remains the final invariant-enforcement layer.

**Status:** RESOLVED / LOCKED

### Correction 2 — Concurrent share replacement

**Decision:** Share replacement uses a single database transaction with trip-row locking.

Requirements:

- Serialize concurrent share lifecycle operations for the same trip by locking the trip row.
- Re-check eligibility/current state after acquiring the lock.
- Revoke the existing ACTIVE share and create the replacement within the same transaction.
- Never commit a state where the old share is revoked but the replacement was not created.
- Retain the partial unique index as the final database invariant.
- Treat unique-violation handling as a defensive safety path.

**Status:** RESOLVED / LOCKED

### Correction 3 — AI freshness

**Decision:** AI summaries may be cached only when tied to the current public evidence state.

Requirements:

- Evidence and deterministic timeline remain the source of truth.
- A cached AI summary must not be reused for a different evidence state.
- Evidence changes invalidate the applicability of the previous cached AI summary and require regeneration.
- The public representation remains live/current rather than an immutable snapshot.
- AI failure degrades gracefully to the deterministic evidence/timeline response.

**Status:** RESOLVED / LOCKED

### Correction 4 — Anonymous rate-limit key

**Decision:** Public verification uses a composite anonymous rate-limit strategy based on requester IP plus the supplied opaque share token, with IP-level protection also covering malformed/invalid token requests.

Requirements:

- Apply application-level rate limiting before expensive verification/AI work.
- Align thresholds with the established Node 6 rate-limiting architecture rather than inventing a conflicting policy.
- Do not persist raw bearer tokens.

**Status:** RESOLVED / LOCKED

### Correction 5 — `created_by` deletion behavior

**Decision:** The `created_by` relationship uses `ON DELETE SET NULL`.

Requirements:

- Deleting the creating user must not delete the public-share lifecycle record.
- `created_by` becomes `NULL` when its referenced user is deleted.
- Share history (`created_at`, `status`, `revoked_at`) remains intact.

**Status:** RESOLVED / LOCKED

---

## 3. Final Architecture Consistency Check

The finalized architecture remains consistent with the previously approved design:

- Dedicated public verification security boundary.
- Dedicated `trip_public_shares` lifecycle table.
- Opaque 256-bit bearer token; SHA-256 hash persisted only.
- Exactly one ACTIVE share per completed trip.
- Automatic replacement revokes the previous token permanently.
- Company-only share creation/revocation with existing company→trip authorization.
- Dedicated anonymous public verification API.
- Explicit server-side public projection allowlist.
- Current permitted data, not immutable public snapshots.
- Public evidence/timeline primary; AI enhancement secondary.
- Public photos, exact GPS, GPS accuracy, street addresses, internal IDs, and private metadata excluded.
- Evidence completeness remains categorical COMPLETE/INCOMPLETE using existing evidence/completion rules.
- Public timeline limited to Arrival, Check-in, and Departure with timestamp and safe city/area when available.
- AI-unavailable behavior preserves evidence/timeline.
- Generic invalid/revoked/unusable token behavior with HTTP 404 and no state distinction.
- Public verification rate limiting aligned with Node 6 architecture.
- Minimal private audit events for create/access/revoke/replacement without raw bearer tokens.
- Dynamic/no-cache behavior prevents stale public state after revocation or evidence changes.
- `noindex`/crawler controls are used as discovery controls, not authorization.
- Existing authenticated APIs remain protected and are not converted into mixed public/authenticated endpoints.
- Phase 1b and Phase 3 scope remain outside this implementation baseline.

**Consistency result:** PASS.

---

## 4. Final Status

Claude's review verdict was **READY WITH CORRECTIONS**. All five corrections have now been resolved as explicit architecture decisions.

Therefore:

> **Node 7 Phase 1a public evidence-sharing architecture is READY FOR IMPLEMENTATION.**

Implementation is still **not authorized by this record alone**. The next governance step is to create the approved Antigravity implementation prompt in:

`03_IMPLEMENTATION/prompts/`

That implementation prompt must derive only from the finalized architecture and locked Chat30 requirements. It must remain separate from this architecture record.

---

## 5. Scope Protection

No implementation work in this record authorizes:

- Phase 1b full three-portal UI/UX redesign.
- Phase 3 stretch features.
- Public raw photos.
- Public exact GPS or route maps.
- Public street addresses.
- Public PDF/export/download.
- Automatic share expiration.
- Configurable sharing policies.
- Numeric evidence scores.
- Immutable public snapshots.
- New vehicle model solely for public sharing.
- Expanded public analytics.

---

## 6. Governance

```text
Chat30 — 65 product/security decisions LOCKED
        ↓
Chat31 — initial source reconciliation
        ↓
Chat32 — final source reconciliation / READY FOR ARCHITECTURE
        ↓
Chat33 — architecture consolidation for Claude review
        ↓
Claude — READY WITH CORRECTIONS
        ↓
Chat34 — five corrections resolved and architecture finalized
        ↓
READY FOR IMPLEMENTATION
        ↓
Next: implementation prompt in 03_IMPLEMENTATION/prompts/
```

**No direct implementation instruction is contained in this record.**
