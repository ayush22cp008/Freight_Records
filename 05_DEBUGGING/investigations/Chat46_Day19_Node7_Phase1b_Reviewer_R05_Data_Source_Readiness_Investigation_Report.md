# Chat46 — Day 19 — Node 7 — Phase 1b
# Existing Reviewer Portal / System Investigation + R-05 Readiness Report

**Status:** COMPLETE — EXISTING-SYSTEM DISCOVERY
**Investigation:** Chat46 / Day19 / Node 7 / Phase 1b
**Date:** 2026-09-09
**Investigator:** Antigravity
**Purpose:** Discover and verify the whole existing Reviewer Portal/system before Reviewer redesign implementation.

---

## 1. Investigation Objective

The purpose of this investigation was to discover the **whole existing Reviewer Portal/system** and compare its actual implementation capabilities with the locked Reviewer Portal Blueprint.

R-05 is treated as the resulting readiness gate, with particular attention to whether the existing system can support the locked Verification History / completed-record experience without crossing the protected Phase 1b implementation boundary.

This is discovery/readiness work only. It does not authorize implementation or backend changes.

---

## 2. Scope and Governing Records Inspected

### Governing Records

- `00_PROJECT_CONTROL/ROADMAP.md`
- `00_PROJECT_CONTROL/CURRENT_STATUS.md`
- `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
- `00_PROJECT_CONTROL/DECISIONS/Chat42_Day16_Node7_Phase1b_Implementation_Boundary_Decision.md`
- `03_IMPLEMENTATION/plans/Chat42_Day16_Node7_Phase1b_Implementation_Preparation_Master_Scope.md`
- `03_IMPLEMENTATION/prompts/Chat42_Day16_Node7_Phase1b_Master_Implementation_Prompt.md`
- `03_IMPLEMENTATION/prompts/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Data_Source_Readiness_Investigation_Handoff.md`

### Existing application areas inspected

- `src/app/(authenticated)/reviewer/queue/page.tsx`
- `ReviewAction.tsx`
- `src/app/api/admin/review/route.ts`
- `004_create_freight_identities.sql`
- `005_v2_onboarding_evidence.sql`

---

## 3. Existing Reviewer Portal — Whole-System Discovery

### 3.1 Existing route and surface structure

The existing Reviewer implementation contains a Reviewer Queue route:

- `/reviewer/queue`
- `src/app/(authenticated)/reviewer/queue/page.tsx`

No existing dedicated Reviewer History route or completed-record detail route was identified.

### 3.2 Existing Reviewer entry / queue

The existing Reviewer experience is centered on the pending verification queue.

The queue page retrieves pending verification records and provides the current entry point into the Reviewer workflow.

The investigation found that the queue fetches pending records from a server component using `supabaseServer`.

### 3.3 Existing review interaction

`ReviewAction.tsx` provides the existing review action surface and evidence-access behavior used by the current Reviewer flow.

The existing implementation therefore contains a functioning foundation for:

- entering review from the queue;
- accessing submitted onboarding evidence;
- performing the existing review action;
- handling existing verification decisions.

### 3.4 Existing verification decision mechanism

`src/app/api/admin/review/route.ts` contains the existing review decision handling.

The existing decision mechanism updates verification state to `VERIFIED` or `REJECTED` and updates the onboarding evidence status accordingly.

### 3.5 Existing evidence handling

The existing system contains storage-path and signed-URL generation logic in `ReviewAction.tsx`.

This demonstrates that submitted onboarding evidence is already represented and can be accessed through existing mechanisms. It is a potential foundation for the eventual Reviewer evidence viewer, subject to the locked blueprint and authorization boundaries.

### 3.6 Existing completed/history capability

No existing Reviewer History UI, completed-record route, or dedicated completed-record query mechanism was identified in the inspected Reviewer application structure.

Therefore the existing system is substantially centered on the **pending verification lifecycle** rather than the complete locked Queue → Verification → Decision → History workflow.

---

## 4. Existing Persistence / Data Model Discovery

The investigation identified two relevant existing persistence areas.

### `freight_identities`

Existing fields relevant to Reviewer verification include:

- `email`
- `requested_role`
- `verification_status`

`verification_status` represents the final verification outcome as `VERIFIED` or `REJECTED`.

### `onboarding_evidence`

Existing fields relevant to Reviewer evidence/decision records include:

- `rejection_reason`
- `document_type`
- `storage_path`
- evidence `status`

The existing data model therefore contains several of the data elements required by the locked Reviewer workflow.

---

## 5. Existing Reviewer Workflow — Capability Discovery

| Reviewer capability | Existing evidence | Status |
|---|---|---|
| Reviewer Queue | `/reviewer/queue` | VERIFIED |
| Pending verification retrieval | Reviewer queue server component | VERIFIED |
| Review action | `ReviewAction.tsx` | VERIFIED |
| Submitted evidence access | Existing storage/signed-URL logic | VERIFIED |
| Verification decision endpoint | `api/admin/review/route.ts` | VERIFIED |
| Verified outcome persistence | `freight_identities.verification_status` | VERIFIED |
| Rejected outcome persistence | `freight_identities.verification_status` / evidence status | VERIFIED |
| Rejection reason storage | `onboarding_evidence.rejection_reason` | VERIFIED |
| Reviewer History UI | No existing route/component found | VERIFIED GAP |
| Completed-record detail UI | No existing route/component found | VERIFIED GAP |
| Completed-record Reviewer read mechanism | No existing mechanism found | VERIFIED GAP |
| Persisted decision timestamp | No `reviewed_at` / decision-date field identified | VERIFIED GAP |
| Reviewer RLS access to `freight_identities` | No Reviewer policy identified | VERIFIED GAP |

---

## 6. Existing System vs Locked Reviewer Blueprint

The locked Reviewer Blueprint requires:

```text
Reviewer
├── Verification Queue
├── Applicant Verification
│   └── Evidence Examination
├── Decision Result
└── Verification History
    └── Read-only Verification Record
        └── Submitted Evidence Viewer
```

The existing system provides a meaningful operational foundation for the first portion of this workflow, particularly:

```text
Verification Queue
      ↓
Review Action
      ↓
Evidence Access
      ↓
Verification Decision
```

However, the investigation did not find existing implementation for the completed-history portion:

```text
Decision
   ↓
Verification History
   ↓
Read-only Completed Record
   ↓
Submitted Evidence Viewer
```

### Existing foundation

- Verification Queue exists.
- Applicant review action exists.
- Evidence access exists.
- Existing verification decision mechanism exists.
- Verified/Rejected outcomes are persisted.
- Rejection reason is persisted.

### Missing / readiness gaps

- Verification History route/UI.
- Completed verification record read/query mechanism.
- Dedicated read-only completed-record surface.
- Persisted decision timestamp needed for decision chronology.
- Reviewer-readable access to `freight_identities` records required to combine applicant identity/role with completed decisions.

---

## 7. R-05 Data-Source Readiness Mapping

| Locked requirement | Existing source/mechanism | Evidence | Readiness |
|---|---|---|---|
| Applicant email | `freight_identities.email` | `004_create_freight_identities.sql` | AVAILABLE, access constrained |
| Claimed role | `freight_identities.requested_role` | `004_create_freight_identities.sql` | AVAILABLE, access constrained |
| Final decision | `freight_identities.verification_status` | `api/admin/review/route.ts` | AVAILABLE |
| Rejection reason | `onboarding_evidence.rejection_reason` | `005_v2_onboarding_evidence.sql` | AVAILABLE |
| Decision date/time | No persisted decision timestamp identified | Review handler / migration inspection | GAP |
| Submitted evidence reference | `onboarding_evidence.storage_path` | `005_v2_onboarding_evidence.sql` | AVAILABLE |
| Completed-record read mechanism | No existing Reviewer mechanism identified | Reviewer application structure | GAP |
| Reviewer authorization for identity data | Reviewer RLS policy not identified on `freight_identities` | `004_create_freight_identities.sql` | GAP |
| Chronological History ordering | No persisted decision timestamp | Existing persistence inspection | GAP |
| Pagination for larger History sets | No existing History implementation identified | Reviewer application structure | GAP |

---

## 8. Authorization / Access Discovery

The existing system includes an explicit Reviewer RLS policy for `onboarding_evidence` that allows Reviewers to view evidence.

The investigation did not identify an equivalent Reviewer RLS policy on `freight_identities`.

The existing queue's server-side access pattern uses `supabaseServer`, so current queue behavior should not be interpreted as proof that a normal Reviewer-readable client/data contract already exists for completed identity records.

This is relevant to R-05 implementation readiness.

---

## 9. Decision Timestamp Discovery

The existing persistence model does not expose a dedicated `reviewed_at` or equivalent decision timestamp identified by this investigation.

The review handler does not establish a persisted decision timestamp as part of the final approval/rejection operation.

Therefore, the locked requirement that History be ordered by newest decision first cannot currently be demonstrated from an existing decision-time data source.

This is a data-source readiness gap, not a frontend styling issue.

---

## 10. Evidence Viewer Discovery

The existing system already contains evidence storage references and signed URL generation logic in `ReviewAction.tsx`.

This establishes an existing foundation for evidence viewing.

However, completed History records do not currently have an existing read-only record route through which that evidence could be reached.

Therefore:

- evidence storage/reference capability: **VERIFIED**;
- completed-history evidence access flow: **NOT READY**.

---

## 11. Boundary / Dependency Findings

The whole-system investigation identified these dependencies:

1. **Completed-record data access** is not currently exposed through an existing Reviewer History read mechanism.
2. **Reviewer access to `freight_identities`** is not established through the existing Reviewer RLS policy set.
3. **Decision timestamp persistence** is not established.
4. **Reviewer History and completed-record UI/routes** do not currently exist.

Potentially addressing items 1–3 could require protected backend/data changes, including an API/data-access mechanism, RLS changes, and schema/persistence changes.

Under the locked Phase 1b boundary, these must not be implemented silently as part of the Reviewer frontend redesign.

---

## 12. Evidence Classification

### VERIFIED

- Existing Reviewer Queue route exists.
- Existing pending-review retrieval exists.
- Existing review action exists.
- Existing evidence access/signed URL foundation exists.
- Existing review decision endpoint exists.
- Existing Verified/Rejected persistence exists.
- Existing rejection reason storage exists.
- No existing Reviewer History route/component was identified.
- No existing completed-record Reviewer read mechanism was identified.
- No persisted decision timestamp was identified.
- No Reviewer RLS policy on `freight_identities` was identified.

### INFERRED

- Existing evidence signed-URL logic may provide a foundation for the future read-only History evidence viewer, subject to the approved implementation boundary.

### UNKNOWN

- None identified within the inspected scope.

---

## 13. Root Cause of R-05 Readiness Gap

The existing Reviewer implementation was built primarily around the **pending verification workflow**. Its current persistence and access model supports pending review and final decisions, but the completed-decision/history workflow required by the locked Reviewer Blueprint was not established in the existing implementation.

As a result, the existing system has a usable verification foundation but does not currently provide the complete data/read path required for the locked Verification History experience.

---

## 14. Final R-05 Readiness Decision

**R-05: NOT READY**

The **whole existing Reviewer Portal/system discovery is COMPLETE**. Its current capabilities have been mapped against the locked Reviewer Blueprint, and the completed-history dependency has been identified as the R-05 readiness blocker.

The system is **not ready for Reviewer Portal implementation under the current Phase 1b boundary** because the completed-history data/read dependencies are not presently available without potentially crossing protected backend/data boundaries.

---

## 15. Governance Next Step

Do **not** begin Reviewer implementation from this report alone.

The discovered backend/data dependencies must first receive a project-control decision.

Possible governance outcomes are:

- preserve the current frontend-only boundary and explicitly constrain Reviewer implementation to capabilities supported by the existing system; or
- separately investigate and authorize any required backend/data changes before implementation.

No backend/schema/RLS/API change is authorized by this investigation report.

---

## 16. Investigation Closure

**Whole Existing Reviewer Portal/System Discovery:** COMPLETE

**R-05 Readiness:** NOT READY

**Reviewer Implementation:** BLOCKED pending governance decision

**Implementation authorization:** NOT granted by this report
